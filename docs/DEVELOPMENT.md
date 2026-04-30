# REACHER v2 — Development Guide

This guide explains how the REACHER system works and how to modify it. It is written for researchers or developers who may need to add new hardware devices, create new behavioral paradigms, change the user interface, or extend the data pipeline. No prior experience with the codebase is assumed.

---

## Table of Contents

1. [System Architecture Overview](#1-system-architecture-overview)
2. [Development Environment Setup](#2-development-environment-setup)
3. [Repository Structure](#3-repository-structure)
4. [Development Workflow](#4-development-workflow)
5. [Modifying the Backend (reacher)](#5-modifying-the-backend-reacher)
6. [Modifying the Frontend (labrynth)](#6-modifying-the-frontend-labrynth)
7. [Modifying the Firmware (reacher-firmware)](#7-modifying-the-firmware-reacher-firmware)
8. [Coordinating Cross-Repository Changes](#8-coordinating-cross-repository-changes)
9. [Testing](#9-testing)
10. [Building and Packaging](#10-building-and-packaging)
11. [Version Management](#11-version-management)
12. [Release Process](#12-release-process)
13. [Downstream Analysis](#13-downstream-analysis)
14. [Additional Resources](#appendix-additional-resources)

---

## 1. System Architecture Overview

REACHER is a three-layer system for controlling head-fixed rodent operant conditioning experiments:

```
┌──────────────────────┐         JSON / Serial           ┌──────────────────────┐       REST + WebSocket       ┌──────────────────────┐
│                      │       (115200 baud, \n)         │                      │     (http://localhost:6229)  │                      │
│   Arduino + Firmware │ ◄─────────────────────────────► │   reacher backend    │ ◄──────────────────────────► │   labrynth frontend  │
│   (reacher-firmware) │                                 │   (FastAPI / Python) │                              │   (React 19 / Vite)  │
│                      │                                 │                      │                              │                      │
└──────────────────────┘                                 └──────────────────────┘                              └──────────────────────┘
     Hardware layer                                          Application layer                                     Presentation layer
```

### Data Flow

A typical event (e.g., a lever press) flows through the system like this:

1. **Lever press** — the animal presses a lever, closing a switch on an Arduino GPIO pin.
2. **Firmware** — the Scheduler classifies the press (active/inactive/timeout), fires the reward chain if the trigger condition is met, and emits a JSON event over serial.
3. **Backend** — the REACHER kernel's serial-reader thread reads the JSON line, queues it, and the queue-handler thread routes it by `level` code to the appropriate handler. Behavioral events are stored in memory and broadcast via WebSocket.
4. **Frontend** — the React UI receives the WebSocket message, updates Zustand stores, and renders live counters, timelines, and status indicators.
5. **Export** — when the session ends, the backend auto-exports `behavior_events.csv` and `frame_timestamps.csv`. The user can also trigger a ZIP export containing all session data.

### Serial Protocol

All communication between the backend and firmware uses **JSON-encoded messages over serial** at **115200 baud**, newline-delimited (`\n`).

**Firmware → Host (output levels):**

| Level | Purpose | Example |
|-------|---------|---------|
| `"000"` | Configuration / settings dump | `{"level":"000","device":"CONTROLLER","sketch":"fr.ino","version":"v2.0.0","schedule":"FIXED_RATIO"}` |
| `"001"` | Arm / disarm state changes | `{"level":"001","device":"CUE","armed":true}` |
| `"006"` | Error messages | `{"level":"006","desc":"Pump relay timeout"}` |
| `"007"` | Behavioral events | `{"level":"007","device":"SWITCH_LEVER","event":"PRESS","class":"ACTIVE","orientation":"RH","start_timestamp":12340,"end_timestamp":12420}` |
| `"008"` | Microscope frame timestamps | `{"level":"008","device":"MICROSCOPE","timestamp":12500}` |

**Host → Firmware (commands):**

Commands are sent as JSON objects with a `cmd` field and optional payload keys:
```json
{"cmd": 101}                          // SESSION_START (no payload)
{"cmd": 371, "frequency": 8000}      // CUE_SET_FREQUENCY (with payload)
{"cmd": 105, "paused": true}         // SESSION_PAUSE (bool payload)
```

### Command Code Encoding

Command codes use a **prefix + suffix** system:

| Prefix | Device | Suffix | Action |
|--------|--------|--------|--------|
| `1xx` | Controller | `x00` | Disarm |
| `2xx` | Session Setup | `x01` | Arm |
| `3xx` | Cue (primary) | `x03` | Test |
| `4xx` | Pump (primary) | `x71` | Set frequency |
| `5xx` | Lick Circuit | `x72` | Set duration |
| `6xx` | Laser | `x74` | Set timeout |
| `9xx` | Microscope | `x75` | Set ratio |
| `10xx` | RH Lever | `x80` | Set inactive / mode B |
| `13xx` | LH Lever | `x81` | Set active / mode A |
| | | `x82` | Set mode B (secondary) |

For example, `371` = Cue (`3xx`) + Set frequency (`x71`). The full list is defined identically in both `Commands.h` (firmware) and `commands.py` (Python) — these must always stay in sync.

### Paradigm Overview

| Paradigm | Firmware Scheduler | Trigger Type | Description |
|----------|-------------------|--------------|-------------|
| FR (Fixed Ratio) | `Scheduler` | `PRESS_COUNT` | N active lever presses → reward chain |
| PR (Progressive Ratio) | `Scheduler` | `PRESS_COUNT` (with `prStep > 0`) | Increasing ratio per reinforcement |
| VI (Variable Interval) | `Scheduler` | `AVAILABILITY_WINDOW` | Press during random window → reward |
| Omission | `Scheduler` | `ABSENCE_TIMER` | No press for N ms → reward |
| Pavlovian | `PavlovianScheduler` | Trial state machine | CS+/CS- trials with configurable ITI |

FR, PR, VI, and Omission all use the shared `Scheduler` class with a **trigger → chain → action** pattern. Pavlovian uses a separate `PavlovianScheduler` that implements a trial-based state machine.

### Multi-Machine / mDNS Discovery

Starting in the v2 series, the backend supports a dual-role deployment in which one host runs the user-facing Labrynth GUI ("primary") while one or more peripherals (typically Raspberry Pis) run the same `reacher` package headless and own the Arduino hardware. The user pairs each peripheral with the GUI and then drives sessions on it as if it were local.

**Role detection (no flag, no separate package).** The same `reacher` package serves both roles. At startup, `src/reacher/api/app.py` resolves a `STATIC_DIR` (the bundled React frontend). If a frontend is bundled, the instance is a **primary**; if not, it is a **peripheral**. Peripherals rotate a 6-digit pairing code on stdout for one-time pairing; primaries do not.

**Service registration.** `src/reacher/discovery.py` registers the backend over Zeroconf as `_reacher._tcp.local.` with TXT records for `device_id` and `version` (no API key is broadcast). Discovery polling on the primary surfaces unpaired peers via mDNS every ~10 s.

**Pairing flow.** The frontend (`web/src/components/machines/MachinePanel.tsx`, backed by `useMachineStore`) shows discovered peripherals as cards. The user clicks **Add Machine**, enters the URL plus the 6-digit code, and the primary calls `POST /api/discovery/{id}/pair` (auth-free) to exchange the code for a long-lived API key. The key is stored on the primary's backend, never in the browser.

**Proxy mode.** When the active machine is remote, the React app's API client targets `/api/proxy/{deviceId}/...` on the **local** primary; the primary forwards each request to the remote peripheral with the stored API key attached. WebSocket connections fetch a short-lived token via `getWsTokenAsync()` before opening, so even the WS handshake doesn't expose the key to the browser.

**Multicast fallback.** On networks where mDNS multicast is filtered (university LANs, some VLANs), the peripheral can self-register via unicast HTTP by setting the `REACHER_BROKER_URL` env var. The primary then discovers it through `POST /api/discovery/register` rather than mDNS.

---

## 2. Development Environment Setup

### Prerequisites

| Tool | Version | Purpose |
|------|---------|---------|
| Python | >= 3.10 | Backend runtime |
| Node.js | 20+ | Frontend build toolchain |
| npm | (bundled with Node) | Frontend dependency management |
| arduino-cli | latest | Firmware compilation and upload |
| Git | latest | Version control |

### Installing Development Dependencies

**Backend (reacher):**
```bash
cd reacher
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -e ".[dev]"     # Installs: pytest, pytest-asyncio, httpx, ruff
```

**Frontend (labrynth/web):**
```bash
cd labrynth/web
npm ci
```

**CLI (labrynth — optional):**
```bash
cd labrynth
pip install -e ".[cli]"    # Installs: prompt_toolkit, httpx, websockets
```

**Firmware toolchain:**
```bash
arduino-cli core update-index
arduino-cli core install arduino:avr
```

### Running in Development

**Backend only:**
```bash
# From the reacher directory (venv activated)
python -m reacher
# Server starts on http://127.0.0.1:6229
```

**Frontend with hot-reload:**
```bash
# Terminal 1: start backend
cd reacher && python -m reacher

# Terminal 2: start Vite dev server
cd labrynth/web && npm run dev
# Vite proxies API requests to the backend
```

**Firmware compilation:**
```bash
cd reacher-firmware
bash compile.sh
# Outputs: hex/fr.hex, hex/pr.hex, hex/vi.hex, hex/omission.hex, hex/pavlovian.hex
```

---

## 3. Repository Structure

### reacher (Backend — Python)

```
reacher/
├── pyproject.toml                    # Package metadata, dependencies, scripts
├── src/
│   └── reacher/
│       ├── __init__.py               # Public API exports, __version__ = "2.0.1"
│       ├── __main__.py               # python -m reacher entry point
│       ├── session_manager.py        # Multi-session coordinator (port locking, state)
│       ├── kernel/
│       │   ├── reacher.py            # REACHER class — serial I/O, 3-thread model, data
│       │   ├── commands.py           # CommandCode enum, COMMAND_REGISTRY, build_command_payload
│       │   └── simulator.py          # SimulatedSerial + FirmwareSimulator for testing
│       ├── api/
│       │   ├── app.py                # FastAPI app factory, router registration, static files
│       │   ├── middleware/
│       │   │   └── auth.py           # API key authentication middleware
│       │   └── routers/
│       │       ├── session.py        # POST/GET/DELETE /api/sessions
│       │       ├── serial.py         # POST /api/serial/{id}/connect|disconnect
│       │       ├── firmware.py       # POST /api/firmware/upload/{id}, GET boards/paradigms
│       │       ├── hardware.py       # POST /api/hardware/{id}/command (generic)
│       │       ├── program.py        # POST /api/program/{id}/start|stop|pause|limit
│       │       ├── data.py           # GET /api/data/{id}/behavior|frames
│       │       ├── file.py           # POST /api/file/{id}/export/zip, config
│       │       ├── lifecycle.py      # POST /api/lifecycle/shutdown
│       │       └── websocket.py      # WS /ws/{session_id} — real-time event streaming
│       └── uploader/
│           ├── uploader.py           # FirmwareUploader — avrdude subprocess management
│           └── boards.py             # Board profiles (uno, mega) — avrdude args
├── tests/                            # pytest test suite
└── .github/workflows/main.yml       # CI workflow
```

### labrynth (Frontend + Packaging)

```
labrynth/
├── pyproject.toml                    # Package metadata (depends on reacher>=2.0.0)
├── launcher.py                       # PyInstaller entry point — sets REACHER_STATIC_DIR, calls main()
├── build.py                          # 6-stage build orchestrator (firmware → frontend → PyInstaller)
├── reacher.spec                      # PyInstaller spec (bundles frontend, hex, avrdude)
├── cli/
│   ├── __main__.py                   # CLI entry point (reacher-cli command)
│   ├── app.py                        # prompt_toolkit menus, auto-starts backend
│   └── client.py                     # HTTP client for CLI → backend communication
├── web/
│   ├── package.json                  # React 19, Zustand 5, Vite 6, Tailwind 3
│   ├── src/
│   │   ├── main.tsx                  # React app entry point
│   │   ├── App.tsx                   # Root component
│   │   ├── api/
│   │   │   ├── client.ts             # REST API client (fetch wrapper, auth, all endpoints)
│   │   │   ├── websocket.ts          # ReacherWebSocket class (auto-reconnect, heartbeat)
│   │   │   └── mock.ts              # Mock API for demo/tutorial mode
│   │   ├── store/
│   │   │   ├── useSessionStore.ts    # Session state (Zustand)
│   │   │   ├── useLogStore.ts        # Event log state
│   │   │   ├── useNavigationStore.ts # UI navigation state
│   │   │   ├── useThemeStore.ts      # Theme selection
│   │   │   └── useTutorialStore.ts   # Tutorial/demo mode state
│   │   ├── components/
│   │   │   ├── session/              # Port selection, firmware upload
│   │   │   ├── hardware/             # Device control panels (lever, cue, pump, laser, etc.)
│   │   │   ├── program/              # Paradigm settings, limits, presets
│   │   │   ├── monitor/              # Live stats, event timeline, paradigm flow diagram
│   │   │   ├── data/                 # Arduino config display, data export, session notes
│   │   │   ├── terminal/             # Terminal output panel
│   │   │   ├── layout/               # Header, sidebar, theme selector, backgrounds
│   │   │   └── tutorial/             # Tutorial overlay, help panel, welcome screen
│   │   ├── hooks/                    # Custom React hooks
│   │   ├── themes/                   # Theme definitions (reacher, ember, neural, etc.)
│   │   └── types/
│   │       └── index.ts              # TypeScript type definitions
│   └── ...                           # Vite config, Tailwind config, tsconfig, etc.
├── installer/                        # Platform installer scripts (Inno Setup .iss)
└── .github/workflows/
    └── build-installers.yml          # CI: builds Windows/macOS/Linux installers on v*.*.* tags
```

### reacher-firmware (Arduino Firmware)

```
reacher-firmware/
├── compile.sh                        # Compiles all paradigms to hex files
├── libraries/
│   └── REACHERDevices/
│       └── src/
│           ├── Device.h / .cpp       # Base class for all hardware peripherals
│           ├── Pins.h                # Central pin assignment constants
│           ├── Commands.h            # Serial command code constants (Cmd:: namespace)
│           ├── Scheduler.h / .cpp    # Central contingency engine (trigger → chain → action)
│           ├── Trigger.h             # Trigger struct (PRESS_COUNT, ABSENCE_TIMER, AVAILABILITY_WINDOW)
│           ├── Action.h              # Action, Chain, PendingAction structs
│           ├── SwitchLever.h / .cpp  # Lever input device (debounce, press timing)
│           ├── Cue.h / .cpp          # Tone speaker output (frequency, duration)
│           ├── Pump.h / .cpp         # Syringe pump relay output
│           ├── Laser.h / .cpp        # Optogenetic laser PWM output
│           ├── LickCircuit.h / .cpp  # Lick detection input (capacitive touch)
│           ├── Microscope.h / .cpp   # Two-photon microscope sync (ISR + trigger)
│           └── ReacherHelpers.h/.cpp # Command routing and device initialization
├── fr/
│   ├── fr.ino                        # Fixed Ratio sketch
│   └── Config.h                      # FR trigger/chain configuration
├── pr/
│   ├── pr.ino                        # Progressive Ratio sketch
│   └── Config.h                      # PR trigger/chain configuration
├── vi/
│   ├── vi.ino                        # Variable Interval sketch
│   └── Config.h                      # VI trigger/chain configuration
├── omission/
│   ├── omission.ino                  # Omission sketch
│   └── Config.h                      # Omission trigger/chain configuration
├── pavlovian/
│   ├── pavlovian.ino                 # Pavlovian sketch
│   ├── PavlovianScheduler.h / .cpp   # Trial-based state machine (CS+/CS- trials, ITI)
├── hex/                              # Compiled output (generated by compile.sh)
└── .github/workflows/
    └── update_assets.yml             # CI workflow
```

### Pin Assignments (from `Pins.h`)

| Constant | Pin | Description |
|----------|-----|-------------|
| `PIN_LEVER_RH` | 10 | Right-hand lever switch (INPUT_PULLUP) |
| `PIN_LEVER_LH` | 12 | Left-hand lever switch (INPUT_PULLUP) |
| `PIN_LICK_CIRCUIT` | 5 | Lick detection circuit (INPUT_PULLUP) |
| `PIN_MICROSCOPE_TS` | 2 | Microscope frame timestamp ISR input (INT0) |
| `PIN_CUE` | 3 | Primary tone output (PWM) |
| `PIN_PUMP` | 4 | Primary syringe pump relay |
| `PIN_LASER` | 6 | Optogenetic laser PWM output |
| `PIN_MICROSCOPE_TRIG` | 9 | Microscope trigger pulse output |
| `PIN_CUE_2` | 7 | Secondary tone output |
| `PIN_PUMP_2` | 8 | Secondary syringe pump relay |

---

## 4. Development Workflow

### Branch Strategy

Use **feature branches** off `main`:

```bash
git checkout -b feature/add-nosepoke-device
# ... make changes ...
git push -u origin feature/add-nosepoke-device
# Create PR for review
```

### Commit Conventions

Use `<type>: <subject>` format:

```
feat: add nosepoke device support
fix: correct lever debounce timing on Mega boards
refactor: extract command routing from main loop
docs: update firmware pin assignment table
test: add simulator tests for Pavlovian paradigm
chore: bump React to 19.1
```

### Code Style

| Language | Tool | Configuration |
|----------|------|---------------|
| Python | `ruff` | `line-length = 120`, `target-version = "py310"` (in `pyproject.toml`) |
| TypeScript | ESLint 9 | Flat config in `web/` (run via `npm run lint`) |
| C++ (Arduino) | Manual | 2-space indent, `PascalCase` for classes/methods, `camelCase` for locals |

Run linters:
```bash
# Python
cd reacher && ruff check src/

# TypeScript
cd labrynth/web && npm run lint
```

### Running the System Locally

For active development, run the backend and frontend separately:

```bash
# Terminal 1 — Backend (auto-reloads on save if using uvicorn --reload)
cd reacher
source .venv/bin/activate
python -m reacher

# Terminal 2 — Frontend (Vite hot-reload)
cd labrynth/web
npm run dev
```

The backend serves the API on `http://127.0.0.1:6229`. In production, it also serves the built React frontend as static files at the same URL. During development, Vite's dev server handles the frontend and proxies API calls.

---

## 5. Modifying the Backend (reacher)

### Architecture

The backend is a **FastAPI** application (`src/reacher/api/app.py`) that manages one or more concurrent REACHER sessions. Each session owns:

- A **REACHER instance** (`src/reacher/kernel/reacher.py`) with three threads:
  1. **Serial reader thread** — reads JSON lines from the Arduino serial port and queues them.
  2. **Queue handler thread** — dequeues messages and routes them by `level` code to the appropriate handler (`handle_data()`).
  3. **Time monitor thread** — checks program limits (time, infusions, or both) every 100ms.

- A **SessionManager** (`src/reacher/session_manager.py`) that:
  - Creates/destroys sessions, each bound to a unique serial port.
  - Tracks session state: `idle → uploading → connected → running → paused → stopped`.
  - Broadcasts state changes over WebSocket.
  - Prevents two sessions from binding the same port.

**API structure:**

```
GET  /health                          — Health check (no auth)
GET  /api/auth/token                  — Get auth token (localhost only)

POST /api/sessions                    — Create session (port, paradigm)
GET  /api/sessions                    — List sessions
GET  /api/sessions/{id}               — Get session details
DELETE /api/sessions/{id}             — Destroy session

POST /api/serial/{id}/connect         — Open serial, send IDENTIFY
POST /api/serial/{id}/disconnect      — Close serial

POST /api/firmware/upload/{id}        — Upload hex via avrdude
GET  /api/firmware/boards             — List supported boards
GET  /api/firmware/paradigms          — List available paradigms

POST /api/hardware/{id}/command       — Send any command (code + optional value)
GET  /api/hardware/{id}/commands      — List commands for session's paradigm
GET  /api/hardware/{id}/config        — Get firmware info + hardware settings

POST /api/program/{id}/start          — Start program
POST /api/program/{id}/stop           — Stop program
POST /api/program/{id}/pause          — Pause/resume program
POST /api/program/{id}/limit          — Set time/infusion limits

POST /api/sessions/{id}/split         — Begin a new behavior CSV segment without ending the session
POST /api/sessions/{id}/restart       — Reset session counters and start a new segment

GET  /api/data/{id}/behavior          — Get behavioral event data
GET  /api/data/{id}/frames            — Get microscope frame timestamps

POST /api/file/{id}/config            — Set filename and destination
POST /api/file/{id}/export/zip        — Export session data as ZIP

GET  /api/discovery                   — List mDNS peers + paired remote machines
POST /api/discovery/{id}/pair         — Pair a peripheral with the 6-digit code (auth-free)
POST /api/discovery/register          — Unicast self-registration fallback (broker mode)

POST /api/lifecycle/shutdown          — Graceful server shutdown (auth-free for browser sendBeacon)

WS   /ws/{session_id}                 — Real-time event stream (WebSocket)
```

All `/api/*` routes require a Bearer token (auto-fetched from `/api/auth/token` by the frontend) **except** for `/api/discovery/{id}/pair` and `/api/lifecycle/shutdown`, which are auth-free by design (the former bootstraps a new pairing, the latter must work from browser `navigator.sendBeacon`).

**Session segmentation.** When the frontend calls `/sessions/{id}/split` or `/sessions/{id}/restart` mid-session, the backend rotates the on-disk behavioral CSV: the next event lands in `behavior_events_002.csv`, and so on (`behavior_events_001.csv` is the first segment). The exported ZIP collects every segment.

### Environment Variables

The backend reads the following environment variables at startup (defaults shown):

| Variable | Default | Purpose |
|----------|---------|---------|
| `REACHER_PORT` | `6229` | HTTP/WebSocket port for the FastAPI server |
| `REACHER_HOST` | `0.0.0.0` | Bind address. `0.0.0.0` exposes the API on the LAN; set to `127.0.0.1` to restrict to localhost |
| `REACHER_WS_PING_INTERVAL` | `20` (seconds) | WebSocket keepalive ping interval |
| `REACHER_WS_PING_TIMEOUT` | `60` (seconds) | WebSocket disconnect timeout if no pong is received |
| `REACHER_BROKER_URL` | unset | If set on a peripheral, the peripheral self-registers via unicast HTTP to this broker URL (used when mDNS multicast is blocked) |

### Scenario 1: Adding a New Hardware Device

If you add a new hardware peripheral (e.g., a nosepoke sensor), you need to register its command codes in the Python backend so the API can send commands and the simulator can respond.

**Step 1: Add to `CommandCode` enum in `commands.py`:**

```python
# --- Nosepoke (7xx) ---
NOSEPOKE_DISARM = 700
NOSEPOKE_ARM = 701
NOSEPOKE_TEST = 703
NOSEPOKE_SET_SENSITIVITY = 772
```

**Step 2: Add to `COMMAND_REGISTRY` in `commands.py`:**

```python
700: CommandSpec(
    CommandCode.NOSEPOKE_DISARM, "NOSEPOKE_DISARM",
    "Disarm nosepoke sensor",
),
701: CommandSpec(
    CommandCode.NOSEPOKE_ARM, "NOSEPOKE_ARM",
    "Arm nosepoke sensor",
),
703: CommandSpec(
    CommandCode.NOSEPOKE_TEST, "NOSEPOKE_TEST",
    "Test nosepoke sensor",
),
772: CommandSpec(
    CommandCode.NOSEPOKE_SET_SENSITIVITY, "NOSEPOKE_SET_SENSITIVITY",
    "Set nosepoke detection sensitivity",
    payload_key="sensitivity", payload_type="int",
),
```

**Step 3: Update the simulator in `simulator.py`:**

Add arm/disarm handling and test event generation in `FirmwareSimulator.handle_command()`, and include the nosepoke in identification output if armed.

No new API router is needed — the generic `/api/hardware/{id}/command` endpoint can send any registered command. The frontend sends commands via `sendCommand(sessionId, code, value)`.

### Scenario 2: Adding a New API Endpoint

To add a new REST endpoint:

**Step 1: Create a new router file (e.g., `src/reacher/api/routers/analysis.py`):**

```python
from fastapi import APIRouter, Request

router = APIRouter()

@router.get("/{session_id}/summary")
async def get_session_summary(session_id: str, request: Request):
    sm = request.app.state.session_manager
    instance = sm.get_instance(session_id)
    behavior = instance.get_behavior_data()
    return {
        "total_events": len(behavior),
        "infusion_count": instance._infusion_count,
    }
```

**Step 2: Register the router in `app.py`:**

```python
from .routers import data, file, firmware, hardware, lifecycle, program, serial, session, websocket, analysis

# In create_app():
app.include_router(analysis.router, prefix="/api/analysis", tags=["analysis"], dependencies=api_deps)
```

### Scenario 3: Modifying Event Handling

The `REACHER.handle_data()` method routes incoming serial JSON by `level` code:

```python
self.code_dict: Dict = {
    "000": self.update_firmware_information,  # Config/settings
    "001": self.logger.info,                  # Arm/disarm state changes
    "006": self.handle_firmware_error,         # Errors
    "007": self.update_behavioral_events,      # Behavioral events
    "008": self.update_frame_events            # Microscope frames
}
```

To handle a new level code, add a handler method to the `REACHER` class and register it in `code_dict`. For example, to add a level `"009"` for temperature sensor data:

```python
# In __init__:
self.code_dict["009"] = self.update_temperature

# New method:
def update_temperature(self, event: dict) -> None:
    temp = event.get("temperature")
    self.logger.info(f"Temperature: {temp}°C")
    self._emit("temperature", event)
```

---

## 6. Modifying the Frontend (labrynth)

### Architecture

The frontend is a **React 19** single-page application built with:

- **Vite 6** — build tool and dev server
- **TypeScript 5.7** — type safety
- **Zustand 5** — lightweight state management (replaces Redux)
- **Tailwind CSS 3** — utility-first styling
- **Lucide React** — icon library

**Key architectural patterns:**

- **API Client** (`src/api/client.ts`) — all REST calls go through `request<T>()`, which handles auth token injection and error parsing. Individual exports like `sendCommand()`, `startProgram()`, `listPorts()` wrap this helper.
- **WebSocket** (`src/api/websocket.ts`) — `ReacherWebSocket` class manages auto-reconnect with exponential backoff, 20s heartbeat keepalive, and stale connection detection.
- **Stores** (`src/store/`) — Zustand stores hold session state, event logs, navigation, theme, and tutorial state. Components subscribe to slices of store state.
- **Demo Mode** — the API client checks `useTutorialStore.getState().demoMode` and routes to mock responses when enabled, allowing the UI to function without a backend.

### CLI

The `cli/` directory contains a prompt_toolkit-based terminal interface that:
- Auto-starts the backend server if not already running.
- Provides interactive menus for port selection, firmware upload, and session management.
- Communicates with the backend via the same REST API.

### Scenario 1: Adding a New Device Control Panel

To add a UI panel for controlling a new device (e.g., nosepoke):

**Step 1: Add types in `types/index.ts` if needed.**

**Step 2: Create the component in `components/hardware/NosepokeControl.tsx`:**

```tsx
import { sendCommand } from "../../api/client";

interface Props {
  sessionId: string;
}

export default function NosepokeControl({ sessionId }: Props) {
  const handleArm = () => sendCommand(sessionId, 701);    // NOSEPOKE_ARM
  const handleDisarm = () => sendCommand(sessionId, 700);  // NOSEPOKE_DISARM
  const handleTest = () => sendCommand(sessionId, 703);    // NOSEPOKE_TEST

  return (
    <div className="p-4 border rounded">
      <h3 className="font-bold mb-2">Nosepoke</h3>
      <button onClick={handleArm}>Arm</button>
      <button onClick={handleDisarm}>Disarm</button>
      <button onClick={handleTest}>Test</button>
    </div>
  );
}
```

**Step 3: Add to `HardwarePanel.tsx`** to render it alongside other device controls.

The `sendCommand(sessionId, code, value?)` function in `client.ts` sends a POST to `/api/hardware/{id}/command` with the command code and optional value. This is the standard pattern for all device interactions — no per-device API endpoints are needed.

### Scenario 2: Changing Application Behavior

- **State logic** — modify Zustand stores in `src/store/`. For example, `useSessionStore.ts` manages session state, connection status, and firmware info.
- **Visual components** — modify components in `src/components/`. The layout is organized by panel: session, hardware, program, monitor, data, terminal, tutorial.
- **API calls** — modify `src/api/client.ts` to add or change REST endpoints.
- **WebSocket handling** — modify `src/hooks/useSessionWebSockets.ts` to handle new event types from the backend.

---

## 7. Modifying the Firmware (reacher-firmware)

### Architecture

The firmware is organized as:

- **Shared library** (`libraries/REACHERDevices/src/`) — device classes, scheduler, commands, and pin assignments shared by all paradigms.
- **Paradigm sketches** (`fr/`, `pr/`, `vi/`, `omission/`, `pavlovian/`) — each contains a `.ino` file and a `Config.h` that configures the Scheduler's triggers and chains for that behavioral paradigm.

**Device base class** (`Device.h`):

```cpp
class Device {
public:
  Device(int8_t pin, uint8_t mode, const char* device);
  void ArmToggle(bool arm);      // Arm/disarm + log state change (level "001")
  void SetOffset(uint32_t offset); // Set session timestamp offset
  byte Pin() const;
  bool Armed() const;
  uint32_t Offset() const;
protected:
  int8_t pin;
  uint8_t mode;
  bool armed;
  const char* device;  // Name for JSON output (e.g., "CUE", "PUMP")
};
```

All device classes (`SwitchLever`, `Cue`, `Pump`, `Laser`, `LickCircuit`, `Microscope`) extend `Device`.

**Scheduler trigger-chain-action pattern:**

The `Scheduler` evaluates **Triggers** on each input event or tick. When a trigger fires, it activates a **Chain** — an ordered sequence of **Actions** that execute immediately or are deferred:

```
Trigger (condition)  ──►  Chain (sequence)  ──►  Action (device activation)
   │                         │                      │
   ├── PRESS_COUNT           ├── Step 0: CUE        ├── ACTIVATE_DEVICE
   ├── ABSENCE_TIMER         ├── Step 1: PUMP       ├── SET_TIMEOUT
   └── AVAILABILITY_WINDOW   ├── Step 2: LASER      └── RESET_TRIGGER
                             └── Step 3: TIMEOUT
```

**Key structs:**

```cpp
// Trigger.h — what condition must be met
struct Trigger {
  TriggerType type;       // PRESS_COUNT, ABSENCE_TIMER, AVAILABILITY_WINDOW, MANUAL
  uint8_t chainIndex;     // Which chain to fire
  bool enabled;
  uint8_t threshold;      // Press count required (FR/PR)
  uint8_t initialThreshold; // Saved threshold for PR reset
  uint8_t pressCount;     // Current accumulated active presses
  uint8_t prStep;         // PR increment (0 = fixed ratio)
  uint32_t absenceMs;     // Omission: required silence duration
  uint32_t absenceStart;  // Timestamp of last press (timer restarts)
  uint32_t windowStart;   // VI: start of availability window
  uint32_t windowEnd;     // VI: end of availability window
  uint32_t intervalMin;   // VI: total interval length (ms)
  bool firedInWindow;     // VI: already fired in current window
  DeviceType sourceFilter; // NONE = any lever, or specific lever
  uint8_t probability;     // 0-100, chance to fire when met
};

// Action.h — what to do when triggered
struct Action {
  ActionType type;      // ACTIVATE_DEVICE, SET_TIMEOUT, RESET_TRIGGER, NONE
  DeviceType target;    // Which device (CUE, PUMP, LASER, etc.)
  uint32_t offsetMs;    // Delay from trigger fire time (0 = immediate)
  uint32_t param;       // Duration for ACTIVATE, timeout for SET_TIMEOUT
};

// Action.h — ordered sequence of actions
struct Chain {
  Action steps[MAX_CHAIN_STEPS];  // Up to 6 steps per chain
  uint8_t numSteps;
};
```

### Scenario 1: Adding a New Device Type

To add a new input or output device to the firmware:

**Step 1: Create the device class.** Add `Nosepoke.h` and `Nosepoke.cpp` in `libraries/REACHERDevices/src/`:

```cpp
// Nosepoke.h
#ifndef NOSEPOKE_H
#define NOSEPOKE_H
#include "Device.h"

class Nosepoke : public Device {
public:
  Nosepoke(int8_t pin);
  void Poll(uint32_t now);  // Check sensor state
};
#endif
```

**Step 2: Add the pin to `Pins.h`:**

```cpp
/// Nosepoke sensor (INPUT_PULLUP)
constexpr int8_t PIN_NOSEPOKE = 11;
```

**Step 3: Add command codes to `Commands.h`:**

```cpp
// --- Nosepoke (7xx) ---
constexpr int NOSEPOKE_DISARM         = 700;
constexpr int NOSEPOKE_ARM            = 701;
constexpr int NOSEPOKE_TEST           = 703;
constexpr int NOSEPOKE_SET_SENSITIVITY = 772;
```

**Step 4: Add to `DeviceType` enum in `Device.h`:**

```cpp
enum class DeviceType : uint8_t {
  LEVER_RH, LEVER_LH, LICK, CUE, CUE_2,
  PUMP, PUMP_2, LASER, MICROSCOPE,
  NOSEPOKE,  // <-- new
  NONE
};
```

**Step 5: Register with the Scheduler** by adding `RegisterNosepoke()` in `Scheduler.h/.cpp`, and update `ReacherHelpers` to route nosepoke commands.

### Scenario 2: Adding a New Paradigm

To create a new behavioral paradigm (e.g., a Differential Reinforcement of Low-rate behavior, DRL):

**Step 1: Create the paradigm directory:**
```
reacher-firmware/drl/
├── drl.ino       # Main sketch
└── Config.h      # Trigger/chain configuration
```

**Step 2: Configure triggers and chains in `Config.h`.** For DRL, you might use an `ABSENCE_TIMER` trigger that requires the animal to wait N seconds between presses:

```cpp
inline void configureDRL(Scheduler& sched, Cue& cue, Pump& pump,
                         uint32_t drlInterval, DeviceType timeoutTarget) {
  Trigger* t = sched.GetTrigger(0);
  if (t) {
    t->type = TriggerType::ABSENCE_TIMER;
    t->chainIndex = 0;
    t->enabled = true;
    t->absenceMs = drlInterval;
    t->sourceFilter = DeviceType::NONE;
    t->probability = 100;
  }

  Chain* c = sched.GetChain(0);
  if (c) {
    c->numSteps = 3;
    c->steps[0] = {ActionType::ACTIVATE_DEVICE, DeviceType::CUE, 0, cue.Duration()};
    c->steps[1] = {ActionType::ACTIVATE_DEVICE, DeviceType::PUMP, cue.Duration(), pump.Duration()};
    c->steps[2] = {ActionType::SET_TIMEOUT, timeoutTarget, 0, sched.TimeoutInterval()};
  }
}
```

**Step 3: Add to `compile.sh`:**

```bash
for sketch in fr pr vi omission pavlovian drl; do
```

**Step 4: Update the Python backend.** In `commands.py`:

```python
PARADIGMS = ("fr", "pr", "vi", "omission", "pavlovian", "drl")

SCHEDULE_TO_PARADIGM: Dict[str, str] = {
    ...
    "DRL": "drl",
}
```

In `uploader.py`, `PARADIGMS` must also include `"drl"`.

### Scenario 3: Modifying Pin Configuration

All pin assignments are centralized in `Pins.h`. To reassign a pin:

1. Edit `Pins.h` (e.g., change `PIN_PUMP` from 4 to 11).
2. Recompile **all** paradigms: `bash compile.sh`.

No other files need to change — all device classes read their pin from the constant passed at construction time.

### Scenario 4: Modifying the Reward Chain

Each paradigm's `Config.h` defines the reward chain — the sequence of device activations triggered by the schedule condition. Here is the FR chain annotated:

```cpp
// fr/Config.h — Fixed Ratio reward chain
Chain* c = sched.GetChain(0);
c->numSteps = 4;

// Step 0: Play cue tone immediately when ratio is met
c->steps[0].type = ActionType::ACTIVATE_DEVICE;
c->steps[0].target = DeviceType::CUE;
c->steps[0].offsetMs = 0;                      // Immediate
c->steps[0].param = cue.Duration();             // Duration of tone (default 1600ms)

// Step 1: Activate pump after cue + trace interval
c->steps[1].type = ActionType::ACTIVATE_DEVICE;
c->steps[1].target = DeviceType::PUMP;
c->steps[1].offsetMs = cue.Duration() + traceInterval;  // Deferred
c->steps[1].param = pump.Duration();             // Infusion duration (default 2000ms)

// Step 2: Activate laser at same time as pump
c->steps[2].type = ActionType::ACTIVATE_DEVICE;
c->steps[2].target = DeviceType::LASER;
c->steps[2].offsetMs = cue.Duration() + traceInterval;
c->steps[2].param = laser.Duration();            // Laser pulse (default 5000ms)

// Step 3: Set lever timeout (prevents presses during reward delivery)
c->steps[3].type = ActionType::SET_TIMEOUT;
c->steps[3].target = timeoutTarget;              // Which lever gets the timeout
c->steps[3].offsetMs = 0;                        // Immediate
c->steps[3].param = sched.TimeoutInterval();     // Timeout duration (default 20000ms)
```

To modify the chain (e.g., add a secondary pump activation), add a new step and increment `numSteps`. The maximum is `MAX_CHAIN_STEPS = 6` (defined in `Action.h`).

---

## 8. Coordinating Cross-Repository Changes

### Dependency Chain

Changes flow in this order:

```
reacher-firmware  →  reacher (backend)  →  labrynth (frontend)
```

The firmware defines the hardware interface. The backend mirrors it in Python. The frontend calls the backend API.

### Twin Sources of Truth

**`Commands.h`** (firmware) and **`commands.py`** (Python) must define the same command codes. If you add a command to one, you must add it to the other with the same integer code, or commands will be misrouted.

### Complete Walkthrough: Adding a New Device Across All 3 Repos

This walks through adding a "Nosepoke" sensor end-to-end.

**1. Firmware (`reacher-firmware`):**
- Add `PIN_NOSEPOKE` to `Pins.h` (see [Section 7, Scenario 1](#scenario-1-adding-a-new-device-type))
- Add `NOSEPOKE` to `DeviceType` enum in `Device.h`
- Add command codes (700–772) to `Commands.h`
- Create `Nosepoke.h` / `Nosepoke.cpp` extending `Device`
- Register with Scheduler and update `ReacherHelpers` for command routing
- Recompile: `bash compile.sh`

**2. Backend (`reacher`):**
- Add command codes to `CommandCode` enum in `commands.py` (see [Section 5, Scenario 1](#scenario-1-adding-a-new-hardware-device))
- Add `CommandSpec` entries to `COMMAND_REGISTRY`
- Update `FirmwareSimulator` in `simulator.py` to handle nosepoke commands
- No new router needed — the generic hardware endpoint handles all commands

**3. Frontend (`labrynth`):**
- Create `NosepokeControl.tsx` in `web/src/components/hardware/` (see [Section 6, Scenario 1](#scenario-1-adding-a-new-device-control-panel))
- Add to `HardwarePanel.tsx`
- Use `sendCommand(sessionId, code, value)` for all interactions

### Complete Walkthrough: Adding a New Paradigm Across All 3 Repos

**1. Firmware:**
- Create paradigm directory with `.ino` + `Config.h` (see [Section 7, Scenario 2](#scenario-2-adding-a-new-paradigm))
- Configure triggers and chains in `Config.h`
- Add to `compile.sh`

**2. Backend:**
- Add to `PARADIGMS` tuple in `commands.py`
- Add to `SCHEDULE_TO_PARADIGM` dict in `commands.py`
- Add to `PARADIGMS` tuple in `uploader.py`
- Add to `PARADIGM_TO_SCHEDULE` and `SCHEDULE_TO_SKETCH` in `simulator.py`
- Add simulator runner method (e.g., `_run_drl()`)

**3. Frontend:**
- The paradigm will automatically appear in the firmware upload UI (it reads available paradigms from the API).
- If the paradigm needs custom settings UI (e.g., DRL interval), add a settings component in `web/src/components/program/`.

### Version Compatibility

| Component | Develop branch | Notes |
|-----------|----------------|-------|
| reacher-firmware | v2.0.0 | Pin assignments and serial protocol unchanged within the v2 series |
| reacher (Python) | 2.0.1 | Adds session SPLIT/RESTART, configurable WS ping, mDNS discovery, broker fallback |
| labrynth (React + packaging) | 2.1.x-dev | Adds Machines panel, multi-machine pairing, proxy mode |

The three must be at compatible versions across the v2 line. The firmware embeds its version in the `SendIdentification()` response, and the backend logs it on connection.

---

## 9. Testing

### Testing Without Hardware

The backend includes a **SimulatedSerial** class (`src/reacher/kernel/simulator.py`) that acts as a drop-in replacement for `serial.Serial`. It wraps a `FirmwareSimulator` that generates paradigm-aware firmware output (lever presses, reward chains, Pavlovian trials, etc.) in real time.

To use the simulator:
1. Start the backend: `python -m reacher`
2. In the frontend, select **"SIMULATOR"** from the port list.
3. Upload firmware and run sessions as normal — the simulator generates realistic event streams.

The simulator responds to all commands (arm/disarm, parameter changes, start/stop) and generates events matching the selected paradigm's behavior.

### Backend Unit Tests

```bash
cd reacher
source .venv/bin/activate
pytest tests/ -v
```

The dev extras include `pytest`, `pytest-asyncio`, and `httpx` for testing FastAPI endpoints. `asyncio_mode = "auto"` is configured in `pyproject.toml`.

### Frontend Testing

```bash
cd labrynth/web
npm run dev     # Start dev server
npm run lint    # Run ESLint
```

The frontend also supports **demo mode** via mock API responses, which can be used for UI testing without a backend.

### Firmware Testing

**Compilation testing:**
```bash
cd reacher-firmware
bash compile.sh
# Verifies all paradigms compile without errors
```

**Command testing:** Connect an Arduino, open the Serial Monitor at 115200 baud, and send JSON commands manually:
```json
{"cmd": 102}
```
The firmware will respond with identification JSON.

### Integration Testing Checklist

When testing a change that spans multiple repos:

- [ ] Backend starts without errors: `python -m reacher`
- [ ] Frontend loads in browser: `http://localhost:6229`
- [ ] Port list includes available serial ports and SIMULATOR
- [ ] Firmware upload completes (or simulator connects)
- [ ] IDENTIFY response is received and parsed
- [ ] Hardware arm/disarm commands update firmware state
- [ ] Device test commands trigger events in the timeline
- [ ] Session start/stop works correctly
- [ ] Live event counters update during session
- [ ] Data export produces valid CSV/ZIP files
- [ ] WebSocket reconnects after brief disconnection

---

## 10. Building and Packaging

### Building the reacher Package

```bash
cd reacher
python -m build
# Produces dist/reacher-{VERSION}.tar.gz and dist/reacher-{VERSION}-py3-none-any.whl
# (e.g., 2.0.1 on the current develop branch)
```

### Building the Labrynth Standalone Application

The `build.py` script orchestrates a 6-stage build pipeline:

```bash
cd labrynth
python build.py
```

| Stage | Action | Flag to skip |
|-------|--------|--------------|
| 0 | Validate environment (submodule, reacher installed) | — |
| 1 | Compile firmware hex files via `compile.sh` | `--skip-firmware` |
| 2 | Build React frontend (`npm ci && npm run build`) | `--skip-frontend` |
| 3 | Validate required assets (frontend dist, hex files, avrdude) | — |
| 4 | Run PyInstaller with `reacher.spec` | — |
| 5 | Report output location | — |

Common invocations:

```bash
python build.py                              # Full build
python build.py --skip-firmware              # Skip hex compilation (use existing)
python build.py --skip-frontend              # Skip npm build (use existing dist/)
python build.py --avrdude /usr/bin/avrdude   # Bundle specific avrdude binary
```

The PyInstaller spec (`reacher.spec`) bundles:
- Python backend (FastAPI + kernel + all dependencies)
- Built React frontend → `_MEIPASS/static/`
- Pre-compiled firmware hex files → `_MEIPASS/hex/`
- Platform avrdude binary → `_MEIPASS/avrdude/`

Output:
- **macOS:** `dist/REACHER.app`
- **Windows:** `dist/REACHER/REACHER.exe`
- **Linux:** `dist/REACHER/REACHER`

### Platform Installers

The CI workflow (`build-installers.yml`) creates platform-specific installers:

| Platform | Format | Tool |
|----------|--------|------|
| Windows | `.exe` installer | Inno Setup (`iscc installer/reacher.iss`) |
| macOS | `.dmg` disk image | `hdiutil create` |
| Linux | `.deb` package | `dpkg-deb --build` |
| Linux | `.tar.gz` portable archive | `tar -czf` |
| Linux | `.AppImage` self-contained binary | `appimagetool` |

All artifacts are emitted with the `labrynth-{VERSION}-{platform}.{ext}` naming convention (the `.deb` uses `_` separators per Debian policy: `labrynth_{VERSION}_amd64.deb`).

### CI/CD

The `build-installers.yml` workflow triggers on:
- **Tag push** matching `v*.*.*` — builds all 3 platforms and creates a GitHub Release.
- **Manual dispatch** (`workflow_dispatch`) — allows specifying a custom reacher branch/tag.

The workflow uses Python 3.11, Node.js 20, and `arduino-cli` with `arduino:avr` core.

---

## 11. Version Management

### Semantic Versioning

REACHER follows [Semantic Versioning](https://semver.org/):

- **MAJOR** — breaking changes to serial protocol, API, or data format
- **MINOR** — new features (devices, paradigms, endpoints) that are backwards-compatible
- **PATCH** — bug fixes with no API changes

### Version Locations

| File | Location | Develop value |
|------|----------|---------------|
| `reacher/pyproject.toml` | `version = "X.Y.Z"` | `2.0.1` |
| `reacher/src/reacher/__init__.py` | `__version__ = "X.Y.Z"` | `2.0.1` |
| `labrynth/pyproject.toml` | `version = "X.Y.Z"` | `2.1.18-dev` |
| `labrynth/web/package.json` | `"version": "X.Y.Z"` | `2.1.18-dev` |
| Firmware | `library.properties` and `SendIdentification()` | `v2.0.0` |

### Version Sync

When releasing a new version, update all five locations above. The CI workflow extracts the version from the git tag (e.g., `v2.1.0` → `2.1.0`) and uses it for installer naming.

---

## 12. Release Process

### Pre-Release Checklist

- [ ] All tests pass: `pytest tests/ -v` (backend), `npm run lint` (frontend)
- [ ] `bash compile.sh` compiles all paradigms without errors
- [ ] Version numbers updated in all locations ([Section 11](#version-locations))
- [ ] `Commands.h` and `commands.py` are in sync
- [ ] Changelog updated

### Release Steps

1. **Create and push a version tag:**
   ```bash
   git tag v2.1.0
   git push origin v2.1.0
   ```

2. **CI auto-builds** all 3 platform installers via `build-installers.yml`.

3. **GitHub Release** is created automatically with generated release notes. Installers are attached:
   - `labrynth-2.1.0-windows-x64.exe`
   - `labrynth-2.1.0-macos-arm64.dmg`
   - `labrynth_2.1.0_amd64.deb`
   - `labrynth-2.1.0-linux-x64.tar.gz`
   - `labrynth-2.1.0-linux-x64.AppImage`

### Post-Release

- Verify installers download and run on each platform.
- Update any external documentation referencing the version number.

---

## 13. Downstream Analysis

REACHER's session output (the per-session `~/REACHER/LOG/{timestamp}/` folder and the user-initiated ZIP archive) is the input format expected by two analysis packages from the same lab. Both are outside the scope of REACHER itself — the protocol guide stops at session export — but they are the standard downstream path for the data this system produces, so any developer extending REACHER's data schema should keep them in mind.

| Tool | Purpose | Repository |
|------|---------|-----------|
| **pynapse** | Neural data engine for aligning calcium-imaging signals (`.npy` fluorescence traces) with REACHER behavioral events and extracting peri-event tensors. Provides the core `Sample`, `Population`, and `Project` abstractions. | https://github.com/Otis-Lab-MUSC/pynapse |
| **axplorer** | Peri-event analysis dashboard (FastAPI + React) built on Pynapse, with PETH computation, response classification, behavioral summary, and figure export to PNG/SVG/PDF/CSV/HDF5. | In development at the Otis Lab; not yet publicly released. |

**Schema-compatibility note:** The columns in `behavior_events.csv` and the format of `frame_timestamps.csv` are part of REACHER's de facto interface with these downstream tools. Renaming columns or changing units (timestamps are in milliseconds from session start) is a breaking change that must be coordinated with `pynapse`'s ingestion code.

---

## Appendix: Additional Resources

### Documentation

- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [React Documentation](https://react.dev/)
- [Zustand Documentation](https://zustand.docs.pmnd.rs/)
- [Arduino CLI Documentation](https://arduino.github.io/arduino-cli/)
- [PySerial Documentation](https://pyserial.readthedocs.io/)
- [Vite Documentation](https://vite.dev/)
- [PyInstaller Documentation](https://pyinstaller.org/)
- [Tailwind CSS Documentation](https://tailwindcss.com/)

### Community

- **Issue Tracker:** Use the GitHub Issues on the relevant repository.
- **Contact:** thejoshbq@proton.me

---

*Document version: REACHER-Suite v2 (develop). Last updated: April 2026.*
