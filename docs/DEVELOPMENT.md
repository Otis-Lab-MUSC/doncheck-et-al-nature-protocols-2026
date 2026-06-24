# REACHER System - Development Guide

This guide provides comprehensive instructions for developers who want to modify the REACHER system, including how to coordinate changes across multiple repositories, testing procedures, and release workflows.

---

## Table of Contents

1. [Development Environment Setup](#development-environment-setup)
2. [Repository Structure](#repository-structure)
3. [Development Workflow](#development-workflow)
4. [Modifying the Core REACHER Package](#modifying-the-core-reacher-package)
5. [Modifying Labrynth Frontend](#modifying-labrynth-frontend)
6. [Modifying Arduino Firmware](#modifying-arduino-firmware)
7. [Coordinating Changes Across Repositories](#coordinating-changes-across-repositories)
8. [Testing](#testing)
9. [Building and Packaging](#building-and-packaging)
10. [Version Management](#version-management)
11. [Release Process](#release-process)
12. [Contribution Guidelines](#contribution-guidelines)

---

## Development Environment Setup

### Prerequisites

Ensure you have completed the setup steps from `SETUP_AND_USAGE.md`:
- Git installed and configured
- All repositories cloned
- Python virtual environments created
- Arduino IDE installed

### Additional Development Tools

Install additional tools for development:

#### Code Editor/IDE

**Recommended:**
- **Visual Studio Code** (cross-platform)
- **PyCharm** (Python development)
- **Arduino IDE 2.x** (firmware development)

**VS Code Extensions:**
```bash
# Install VS Code extensions via command line (if using VS Code)
code --install-extension ms-python.python
code --install-extension ms-python.vscode-pylance
code --install-extension ms-toolsai.jupyter
code --install-extension platformio.platformio-ide
code --install-extension vsciot-vscode.vscode-arduino
```

#### Development Dependencies

**Windows (PowerShell):**
```powershell
# Navigate to each repository and activate venv, then:

# For reacher
cd "$HOME\Projects\REACHER\reacher"
.\venv\Scripts\Activate.ps1
pip install ruff pytest pytest-asyncio pytest-mock pytest-cov
pip install -e .  # editable install of the reacher package
deactivate

# For labrynth
cd "$HOME\Projects\REACHER\labrynth"
.\venv\Scripts\Activate.ps1
pip install ruff pytest
deactivate
```

**macOS/Linux (Bash):**
```bash
# For reacher
cd ~/Projects/REACHER/reacher
source venv/bin/activate
pip install ruff pytest pytest-asyncio pytest-mock pytest-cov
pip install -e .  # editable install of the reacher package
deactivate

# For labrynth
cd ~/Projects/REACHER/labrynth
source venv/bin/activate
pip install ruff pytest
deactivate
```

---

## Repository Structure

### REACHER (Core Python Package)

```
reacher/
├── src/
│   └── reacher/
│       ├── __init__.py           # Package initialization (__version__)
│       ├── kernel/               # Core serial + command logic
│       │   ├── __init__.py
│       │   ├── reacher.py        # Main REACHER class, serial handling
│       │   └── commands.py       # CommandCode enum + COMMAND_REGISTRY
│       ├── api/                  # FastAPI backend (REST + WebSocket)
│       │   ├── app.py            # App factory + entry point (port 6229)
│       │   └── routers/          # session, serial, firmware, hardware,
│       │                         # program, data, file, websocket,
│       │                         # discovery, pairing, proxy, lifecycle,
│       │                         # update, validate
│       ├── session_manager.py    # Session lifecycle/state
│       ├── uploader/             # avrdude-based firmware upload (boards.py)
│       ├── hex/                  # Committed firmware hex (package data)
│       │   └── <board>/<paradigm>.hex
│       ├── monitor.py            # Terminal monitor (reacher-monitor)
│       └── assets/               # Icons, images
├── firmware/                     # Arduino firmware source (folded in)
│   ├── fr/  pr/  vi/  omission/  pavlovian/   # paradigm sketches
│   ├── libraries/REACHERDevices/ # shared library (Commands.h, Pins.h)
│   └── compile.sh                # writes src/reacher/hex/<board>/*.hex
├── tests/                        # Unit tests (incl. test_command_parity.py)
├── pyproject.toml                # Build config, version, deps
└── README.md                     # Documentation
```

**Key Files to Modify:**
- `kernel/reacher.py` - Core serial communication, event handling
- `kernel/commands.py` - Command codes and registry
- `api/routers/*.py` - REST/WebSocket endpoints
- `pyproject.toml` - Version, dependencies, package metadata

### Labrynth (Frontend Application)

```
labrynth/
├── web/                          # React 19 + Vite 6 + TypeScript frontend
│   ├── src/                      # components/, store/, api/, hooks/, types/
│   ├── vite.config.ts            # dev server :5173, proxies /api and /ws → :6229
│   └── package.json              # frontend version
├── cli/                          # Terminal CLI/TUI (python -m cli / reacher-cli)
├── launcher.py                   # PyInstaller entry point (starts the backend)
├── build.py                      # Build orchestrator (PyInstaller)
├── labrynth.spec                 # PyInstaller spec (GUI)
├── labrynth-cli.spec             # PyInstaller spec (CLI/TUI)
├── docs/                         # Documentation
├── pyproject.toml                # Python deps (incl. reacher2p>=3.0.0)
└── README.md                     # Documentation
```

**Key Files to Modify:**
- `web/src/` - React components, store, API client
- `cli/` - Terminal CLI/TUI logic
- `labrynth.spec` / `labrynth-cli.spec` - Build configuration for PyInstaller
- `pyproject.toml` - Dependencies (ensure `reacher2p` version matches)

### REACHER Firmware (Arduino)

Firmware source now lives **inside the reacher repo** at `reacher/firmware/` (the
standalone `reacher-firmware` repo is archived). Each paradigm is a sketch folder;
device classes and the shared command map live in a common library.

```
reacher/firmware/
├── fr/                           # Fixed Ratio paradigm
│   ├── fr.ino                   # Main Arduino sketch
│   └── Config.h                 # Paradigm configuration
├── pr/                           # Progressive Ratio paradigm
├── vi/                           # Variable Interval paradigm
├── omission/                     # Omission paradigm
├── pavlovian/                    # Pavlovian paradigm
├── libraries/
│   └── REACHERDevices/           # Shared device library
│       └── src/
│           ├── Commands.h        # CommandCode ranges (numeric protocol)
│           └── Pins.h            # Pin map
└── compile.sh                    # Compiles sketches → src/reacher/hex/<board>/*.hex
```

**Key Files to Modify:**
- `<paradigm>/<paradigm>.ino` - Main sketch logic
- `libraries/REACHERDevices/src/` - Device classes, `Commands.h`, `Pins.h`
- Pin configurations (`Pins.h`) are standardized across all paradigms

---

## Development Workflow

### Branch Strategy

Follow a feature-branch workflow:

```bash
# Create a new feature branch
git checkout -b feature/your-feature-name

# Make changes and commit
git add .
git commit -m "Add feature: description"

# Push to remote
git push origin feature/your-feature-name

# Create pull request on GitHub
```

### Commit Message Conventions

Use clear, descriptive commit messages:

```
# Format: <type>: <subject>

# Types:
feat: New feature
fix: Bug fix
docs: Documentation changes
style: Code style changes (formatting, no logic change)
refactor: Code refactoring
test: Adding or updating tests
chore: Maintenance tasks (dependencies, build, etc.)

# Examples:
feat: Add variable interval paradigm support
fix: Resolve serial port connection timeout issue
docs: Update installation instructions for Windows
refactor: Simplify dashboard tab initialization
test: Add unit tests for lever debouncing logic
```

### Code Style

#### Python

Lint and format with [ruff](https://docs.astral.sh/ruff/):

```bash
# Check style
ruff check src/

# Auto-format
ruff format src/
```

**Configuration** lives in `pyproject.toml`:

```toml
[tool.ruff]
line-length = 120
target-version = "py310"
```

#### Arduino/C++

Follow Arduino style guidelines:
- Indent with 2 spaces
- Class names in PascalCase
- Function names in camelCase
- Constants in UPPER_CASE
- Comment complex logic
- Use Doxygen-style documentation comments

---

## Modifying the Core REACHER Package

The REACHER package is the heart of the system, handling serial communication, event logging, and the FastAPI backend that the labrynth frontend talks to over REST + WebSocket.

### Understanding the Architecture

**Data Flow:**
1. `kernel/reacher.py` (class `REACHER`) establishes the serial connection to Arduino
2. Two threads handle serial communication:
   - **Reader thread:** `read_serial` reads data from Arduino and queues it
   - **Processor thread:** `handle_queue` drains the queue and dispatches to `handle_data`
3. The FastAPI layer (`api/app.py` + `api/routers/`) exposes the REACHER instance over REST + WebSocket; the labrynth frontend and CLI are peer clients
4. Events are written to live logs under `~/REACHER/LOG/<YYYY-MM-DD_HH-MM-SS>/` in real time

### Common Modification Scenarios

#### Scenario 1: Adding a New Hardware Device

The old Panel/Qt dashboard is gone. In v3, a new device touches three layers:
firmware, the reacher backend, and the labrynth React frontend.

**Step 1: Modify Arduino firmware first** (see firmware section). Add the device's
`CommandCode` values to `firmware/libraries/REACHERDevices/src/Commands.h` and handle
them in each sketch.

**Step 2: Register the commands in the backend**

Add the matching entries to the `CommandCode` enum and `COMMAND_REGISTRY` in
`src/reacher/kernel/commands.py` (with the appropriate paradigm filter). The
`tests/test_command_parity.py` test enforces that `Commands.h` and `CommandCode`
stay in sync.

**Step 3: Expose control via the API**

Hardware control is issued through the hardware router
(`src/reacher/api/routers/hardware.py`). The router resolves the request to a numeric
`CommandCode` and sends it over serial; no per-widget callback wiring is needed.

**Step 4: Add the frontend control**

Add the UI control under `labrynth/web/src/` (a component plus its store/API wiring)
that calls the backend endpoint to arm/disarm and configure the new device.

#### Scenario 2: Modifying Event Logging

**File:** `src/reacher/kernel/reacher.py`

Incoming serial lines (newline-delimited JSON) are read by `read_serial`, queued, and
drained by `handle_queue`, which dispatches each message to `handle_data`. From there,
events are routed by level code to the appropriate updater
(`update_behavioral_events`, `update_frame_events`, `update_slm_events`,
`update_firmware_information`) and persisted via `_write_event_log`. The firmware
tags each message with a level code: `000` config, `001` arm/disarm, `006` error,
`007` behavioral, `008` microscope frame, `009` SLM.

```python
def handle_data(self, line: str) -> None:
    """Parse a newline-delimited JSON line from the Arduino and route it by level code."""
    event = json.loads(line)
    # ... existing routing on event["level"] ...
    # Add handling for a new behavioral ("007") event type, e.g.:
    self.update_behavioral_events(event)   # appends to the in-memory log
    self._write_event_log(event)           # persists to ~/REACHER/LOG/<session>/
```

#### Scenario 3: Adding a New API Endpoint

The Panel dashboard has been replaced by a FastAPI backend and a React frontend.
To add new functionality, add a backend endpoint and a frontend control.

1. **Add (or extend) a router** under `src/reacher/api/routers/`. Routers are grouped
   by concern (`session`, `serial`, `hardware`, `program`, `data`, `websocket`, etc.).
   A handler resolves the request to a `CommandCode` and calls into the `REACHER`
   instance / `session_manager`.

2. **Wire the frontend** under `labrynth/web/src/`: add an API call in the `api/`
   layer, a component in `components/`, and any `store/` state it needs. The Vite dev
   server proxies `/api` and `/ws` to the backend on port 6229.

### Testing REACHER Changes

**Windows (PowerShell):**
```powershell
cd "$HOME\Projects\REACHER\reacher"
.\venv\Scripts\Activate.ps1

# Run unit tests
pytest tests/ -v

# Run with coverage
pytest tests/ --cov=src/reacher --cov-report=html

# View coverage report
start htmlcov/index.html

deactivate
```

**macOS/Linux (Bash):**
```bash
cd ~/Projects/REACHER/reacher
source venv/bin/activate

# Run unit tests
pytest tests/ -v

# Run with coverage
pytest tests/ --cov=src/reacher --cov-report=html

# View coverage report
open htmlcov/index.html  # macOS
xdg-open htmlcov/index.html  # Linux

deactivate
```

### Manual Integration Testing

1. **Install modified REACHER in Labrynth:**

```bash
# Navigate to labrynth
cd labrynth
source venv/bin/activate  # or .\venv\Scripts\Activate.ps1 on Windows

# Install modified reacher in editable mode
pip install -e ../reacher

# Run the backend (serves REST + WS on :6229)
python -m reacher

# In a second terminal, run the frontend dev server (proxies to :6229)
cd web && npm run dev
```

2. **Test with hardware:**
   - Connect Arduino with test firmware
   - Create session in Labrynth
   - Test new functionality end-to-end

---

## Modifying Labrynth Frontend

Labrynth is the application shell around REACHER: a React + Vite frontend (`web/`),
a terminal CLI/TUI (`cli/`), and the PyInstaller build pipeline. It depends on the
`reacher2p` package for the backend and does not vendor it.

### Common Modifications

#### Scenario 1: Changing Application Branding

1. **Replace icon/asset files** under `web/src/` (and the platform icons referenced by
   `labrynth.spec`):
   - `labrynth-icon.png` (256x256 recommended)
   - `labrynth-icon.ico` (Windows)
   - `labrynth-icon.icns` (macOS)
   - `labrynth-banner-wider.png` (600px wide recommended)

2. **Update titles, footer, and copy** in the React components under `web/src/`.

#### Scenario 2: Changing Default Port

The backend port is read from the `REACHER_PORT` environment variable (default
`6229`); the host is `REACHER_HOST` (default `0.0.0.0`). To change it, set the env var
before launching the backend rather than editing a Python literal:

```bash
export REACHER_PORT=8080
python -m reacher
```

If you change the port, also update the proxy targets in `web/vite.config.ts` (which
proxy `/api` and `/ws` to `:6229` by default).

#### Scenario 3: Adding Startup Configuration

Frontend configuration and state live in the React `store/` under `web/src/`.
Backend-side defaults are environment-driven (`REACHER_PORT`, `REACHER_HOST`,
`REACHER_STATIC_DIR`, optional `REACHER_HEX_DIR`). Live logs are written under
`~/REACHER/LOG/<YYYY-MM-DD_HH-MM-SS>/`.

### Testing Labrynth Changes

**Test locally without building:**

```bash
cd labrynth
source venv/bin/activate  # or .\venv\Scripts\Activate.ps1

# Backend (serves REST + WS on :6229)
export REACHER_STATIC_DIR=$(pwd)/web/dist
python -m reacher

# Frontend dev server (second terminal)
cd web && npm run dev

# Test all features manually
```

---

## Modifying Arduino Firmware

The firmware controls hardware and communicates with REACHER via serial.

### Understanding Firmware Architecture

**Key Components:**

1. **Device Classes** (Cue, Laser, Lever, LickCircuit, Pump):
   - Encapsulate pin control and state management
   - Provide consistent interface

2. **Main Loop** (`loop()` driving the `Scheduler`):
   - `loop()` calls `ParseCommands()` to handle incoming serial commands and advances
     the `Scheduler`, which monitors hardware state, processes lever presses, and
     triggers the reward chain based on the paradigm's `Config.h`
   - Device callbacks (registered via `SetCallback`) log behavioral events to serial

3. **Serial Commands**:
   - Two-way communication with REACHER
   - Newline-delimited JSON, 115200 baud
   - Commands are numeric `CommandCode` integers; firmware → backend events carry a
     numeric level code (`000` config, `001` arm/disarm, `006` error, `007`
     behavioral, `008` microscope frame, `009` SLM)

### Common Modification Scenarios

#### Scenario 1: Adding a New Paradigm

1. **Copy existing paradigm as template:**

**Windows (PowerShell):**
```powershell
cd "$HOME\Projects\REACHER\reacher\firmware"
Copy-Item -Recurse fr new
Rename-Item new\fr.ino new.ino
```

**macOS/Linux (Bash):**
```bash
cd ~/Projects/REACHER/reacher/firmware
cp -r fr new
mv new/fr.ino new/new.ino
```

2. **Adjust the paradigm logic.** Use `fr/` (`fr.ino` + `Config.h`) as the reference.
   A sketch instantiates the device objects (`Cue`, `Pump`, `Laser`, `SwitchLever`,
   etc.) on the pins from `Pins.h`, registers behavioral-event callbacks with
   `SetCallback(...)`, and defines the reward chain in `Config.h` (each step is an
   `ActionType`, e.g. `ActionType::SET_TIMEOUT`). The `Scheduler` drives the chain from
   `loop()`. Adapt the chain and callbacks for your paradigm rather than writing a
   monolithic program function.

3. **Handle any new commands.** Commands arrive as numeric `CommandCode` integers
   (defined in `libraries/REACHERDevices/src/Commands.h`) and are dispatched in the
   sketch's `ParseCommands()` switch:

```cpp
// inside ParseCommands(), dispatching on the numeric CommandCode `code`:
switch (code) {
    case Cmd::SET_NEW_PARAM:   // new code added to Commands.h
        // process parameter from the parsed JSON
        break;
    // ... existing command codes
}
```

   Add the new code to `Commands.h` and a matching `case` here.
   `reacher/tests/test_command_parity.py` enforces that `Commands.h` and the Python
   `CommandCode` enum stay in sync.

4. **Update identification** (optional). Each sketch reports its identity via
   `SendIdentification()` — the level-`000` config event carrying the sketch name,
   version, baud rate, and `schedule`. Update it if your paradigm reports a new
   `schedule` value.

#### Scenario 2: Modifying Pin Configuration

**WARNING:** Pin changes must be coordinated across all paradigms and documented for users.

Pins are defined in `libraries/REACHERDevices/src/Pins.h` (e.g. `PIN_LEVER_RH = 10`). At
runtime, every pin **except the fixed microscope-timestamp input (pin 2, INT0)** can be
remapped via the corresponding `*_SET_PIN` command (e.g. `LEVER_RH_SET_PIN`) without
recompiling. To change a compile-time default, edit `Pins.h`, then recompile and commit
the refreshed hex (`bash reacher/firmware/compile.sh`).

**Important:** Update documentation in:
- README.md
- Pin configuration tables in setup guides
- Hardware schematics if applicable

#### Scenario 3: Adding a New Device Type

1. **Create device class files** (e.g., `Solenoid.h` and `Solenoid.cpp`):

**Solenoid.h:**
```cpp
#ifndef SOLENOID_H
#define SOLENOID_H

#include "Device.h"

class Solenoid : public Device {
private:
    bool state;
    unsigned long onTime;
    unsigned long duration;

public:
    Solenoid(int pin, unsigned long duration);
    void turnOn();
    void turnOff();
    void update();  // Call in loop to auto-turn-off after duration
    bool isOn();
};

#endif
```

**Solenoid.cpp:**
```cpp
#include "Solenoid.h"
#include <Arduino.h>

Solenoid::Solenoid(int pin, unsigned long duration) 
    : Device(pin), state(false), onTime(0), duration(duration) {
    pinMode(pin, OUTPUT);
    digitalWrite(pin, LOW);
}

void Solenoid::turnOn() {
    digitalWrite(pin, HIGH);
    state = true;
    onTime = millis();
}

void Solenoid::turnOff() {
    digitalWrite(pin, LOW);
    state = false;
}

void Solenoid::update() {
    if (state && (millis() - onTime >= duration)) {
        turnOff();
    }
}

bool Solenoid::isOn() {
    return state;
}
```

2. **Integrate into sketch:**

```cpp
#include "Solenoid.h"

// Declare on a free pin (see Pins.h for the standard pin map)
Solenoid solenoidObject(/* pin */ 12, 1000);  // 1 second duration

// In loop(): advance the device each tick (alongside ParseCommands() and the Scheduler)
solenoidObject.update();

// Dispatch new commands inside ParseCommands(), keyed on the numeric CommandCode.
// Add ARM_SOLENOID / TRIGGER_SOLENOID to Commands.h (and the Python CommandCode enum) first.
switch (code) {
    case Cmd::ARM_SOLENOID:
        solenoidArmed = true;
        break;
    case Cmd::TRIGGER_SOLENOID:
        if (solenoidArmed) solenoidObject.turnOn();
        break;
}
```

### Testing Firmware Changes

1. **Compile and upload:**
   - Open `.ino` file in Arduino IDE
   - Verify (checkmark button) - check for compilation errors
   - Upload (arrow button) to Arduino
   - Monitor Serial Monitor (115200 baud) for output

2. **Serial command testing:**

   In the Serial Monitor (115200 baud), commands are sent as newline-delimited JSON
   carrying the numeric `CommandCode` (the codes are defined in `Commands.h`, e.g.
   `PUMP_ARM=401`, `LASER_ARM=601`). Send the JSON for the device's arm/trigger codes
   and observe the device behavior and the JSON the firmware emits back.

3. **Integration testing with REACHER:**
   - Update Python code to send new commands
   - Test end-to-end functionality
   - Verify data logging

### Firmware Documentation

Update Doxygen comments:

```cpp
/**
 * @brief Triggers the solenoid valve.
 * 
 * Activates the solenoid for the configured duration. Automatically
 * turns off after duration expires.
 * 
 * @note Requires solenoid to be armed before triggering.
 */
void triggerSolenoid() {
    if (solenoidArmed) {
        solenoidObject.turnOn();
    }
}
```

Generate documentation:

```bash
cd reacher/firmware
doxygen Doxyfile
# Output in docs/html/index.html
```

---

## Coordinating Changes Across Repositories

When changes span multiple repositories, careful coordination is essential.

### Dependency Management

**Labrynth depends on REACHER** (firmware now ships inside reacher, so there are two
release tracks, not three):
```
labrynth → reacher2p (pinned: reacher2p>=3.0.0) → firmware hex (package data)
```

**Version compatibility matrix:**
| Labrynth        | REACHER (reacher2p) | Firmware       |
|-----------------|---------------------|----------------|
| 3.0.0           | 3.0.0               | 3.0.0          |

### Change Scenarios

#### Scenario 1: Adding Feature That Requires Changes in Multiple Repos

**Example:** Adding a new hardware device

**Step 1: Plan the changes**
1. reacher firmware (`reacher/firmware/`): add device class + `CommandCode`s
2. reacher backend: register codes in `kernel/commands.py`, add/extend an API router
3. Labrynth: add the frontend control under `web/src/`

**Step 2: Create feature branches** in the two repos (`reacher`, `labrynth`):

```bash
# In each repository:
git checkout -b feature/add-solenoid-support
```

**Step 3: Implement in order (bottom-up)**

1. **reacher firmware + backend (one repo):**
   ```bash
   cd reacher
   git checkout -b feature/add-solenoid-support
   # Add Solenoid class + CommandCode in firmware/libraries/REACHERDevices/
   bash firmware/compile.sh          # refresh src/reacher/hex/<board>/*.hex
   # Register codes in src/reacher/kernel/commands.py, add router handling
   git add .
   git commit -m "feat: Add solenoid device support"
   git push origin feature/add-solenoid-support
   ```

2. **Labrynth (frontend):**
   ```bash
   cd labrynth
   # Add control under web/src/ (component + api/ + store/)
   # Bump the reacher2p>=X.Y.Z pin in pyproject.toml if reacher was released
   git add .
   git commit -m "feat: Add solenoid control to UI"
   git push origin feature/add-solenoid-support
   ```

**Step 4: Test integration**

```bash
# Install modified REACHER in Labrynth's venv
cd labrynth
source venv/bin/activate
pip install -e ../reacher

# Upload modified firmware to Arduino, then:
python -m reacher        # backend on :6229
cd web && npm run dev    # frontend dev server
```

**Step 5: Create pull requests**

Create PRs in this order:
1. reacher PR (firmware + backend; merged first)
2. Labrynth PR (reference the reacher PR)

Link PRs in descriptions:
```markdown
## Related PRs
- REACHER: Otis-Lab-MUSC/reacher#456

## Testing
- [x] Tested with reacher PR #456
- [x] Tested end-to-end functionality
- [x] Updated documentation
```

#### Scenario 2: Adding or Changing a Serial Command

In v3 the serial protocol is already newline-delimited JSON carrying **numeric
`CommandCode` integers** (e.g. `SESSION_END=100`, `PUMP_ARM=401`, `LASER_ARM=601`,
`LEVER_RH_ARM=1001`). Adding or changing a command is a coordinated edit across the
firmware and backend (both in the reacher repo) plus the frontend.

**Step 1: Scope the change**
- A new code or a changed range is a backend/firmware contract change — bump reacher
  accordingly and re-pin labrynth's `reacher2p>=X.Y.Z`.

**Step 2: Implement the change**

1. **Firmware** — add/adjust the code in
   `firmware/libraries/REACHERDevices/src/Commands.h` and handle it in each sketch's
   command dispatch:
   ```cpp
   switch (code) {
       case PUMP_ARM:      // 401
           pumpActive = true;
           break;
   }
   ```

2. **Backend** — mirror the value in the `CommandCode` enum and `COMMAND_REGISTRY`
   in `src/reacher/kernel/commands.py` (with the right paradigm filter). The
   `tests/test_command_parity.py` test fails if `Commands.h` and `CommandCode` drift.

3. **Frontend** — wire the new command into the relevant component/API under
   `labrynth/web/src/`.

4. **Recompile + version** — run `bash firmware/compile.sh`, commit the refreshed
   `src/reacher/hex/`, and bump versions via `scripts/bump-version.py`.

**Step 3: Migration guide** (for breaking changes)

Create MIGRATION.md documenting the changed codes/ranges, the minimum compatible
firmware hex (shipped with the reacher release), and the upgrade steps: update the
`reacher2p` package, reflash firmware, and update labrynth's pin.

#### Scenario 3: Bug Fix That Affects Multiple Repos

**Step 1: Identify affected repositories**

Example bug: Incorrect timestamp calculation

- Firmware: Timestamps off by start time
- REACHER: Parsing timestamps incorrectly
- Labrynth: Not affected

**Step 2: Fix in the reacher repo**

Firmware and the serial kernel now live in the same repo, so both fixes go on one
branch:

```bash
cd reacher
git checkout -b fix/timestamp-calculation

# Fix the sketch under firmware/<paradigm>/ and recompile hex
bash firmware/compile.sh

# Fix parsing in src/reacher/kernel/reacher.py (handle_data / update_* methods)

git add .
git commit -m "fix: Correct timestamp offset in firmware and parsing"
git push origin fix/timestamp-calculation
```

**Step 3: Coordinate release**

- Release reacher (firmware hex ships with it), then bump labrynth's `reacher2p` pin
- Use a prerelease/patch bump as appropriate (e.g. 3.0.0 → 3.0.1)
- Note in the changelog that the firmware hex must be reflashed

### Testing Multi-Repository Changes

**Integration test checklist:**
- [ ] Firmware compiles and uploads successfully
- [ ] REACHER can establish serial connection
- [ ] Labrynth launches and creates sessions
- [ ] End-to-end feature functionality works
- [ ] Data is logged correctly
- [ ] No regressions in existing features

**Test script example:**

```bash
#!/bin/bash
# test_integration.sh

set -e

echo "=== Integration Test Suite ==="

echo "1. Testing REACHER package..."
cd reacher
source venv/bin/activate
pytest tests/ -v
deactivate

echo "2. Installing REACHER in Labrynth..."
cd ../labrynth
source venv/bin/activate
pip install -e ../reacher

echo "3. Testing reacher imports..."
python -c "from reacher.kernel.reacher import REACHER; print('OK')"

echo "4. Manual test: Launch Labrynth and verify hardware connection"
read -p "Press enter after verifying hardware test..."

echo "=== All tests passed ==="
```

---

## Testing

### Unit Testing

#### REACHER Tests

Location: `reacher/tests/`

Tests use `pytest` with `pytest-asyncio` (`asyncio_mode = "auto"`) and `pytest-mock`.
`testpaths` is `["tests"]`. A key test, `tests/test_command_parity.py`, fails if the
firmware `Commands.h` and the `CommandCode` enum in `kernel/commands.py` drift apart.

**Example test:**

```python
# tests/test_reacher.py
from reacher.kernel.reacher import REACHER

def test_reacher_construction():
    """REACHER constructs without a live serial port."""
    reacher = REACHER()
    assert reacher is not None

def test_handle_data_routes_behavioral(mocker):
    """A behavioral message (level 007) is logged."""
    reacher = REACHER()
    write_log = mocker.patch.object(reacher, "_write_event_log")
    # handle_data takes a raw newline-delimited JSON line (str) and parses it internally
    reacher.handle_data('{"level": "007", "device": "LEVER_RH", "event": "PRESS"}')
    write_log.assert_called()
```

**Run tests:**
```bash
cd reacher
source venv/bin/activate
pytest tests/ -v --cov=src/reacher
```

#### Firmware Tests

Firmware testing is primarily manual:

1. **Compilation test:** Verify sketch compiles without errors
2. **Upload test:** Upload to Arduino and verify in Serial Monitor
3. **Command test:** Send serial commands and verify responses
4. **Hardware test:** Trigger devices and verify behavior

**Automated firmware testing** (advanced):
- Use Arduino CLI with test frameworks
- PlatformIO with unit testing support
- Hardware-in-the-loop testing with scripted serial communication

### Integration Testing

Create test scenarios that span repositories:

**Test Scenario Example:**

```markdown
## Test: Session Run with Data Export

### Prerequisites:
- Arduino with the FR paradigm firmware uploaded
- reacher (reacher2p) 3.0.0 installed
- Labrynth running

### Steps:
1. Launch Labrynth
2. Create new session "test_session_01"
3. Connect to Arduino on COM3
4. Configure: FR=1, timeout=20s
5. Arm right lever and pump
6. Start session
7. Manually press right lever 5 times
8. Verify: 5 lever presses logged, 5 infusions delivered
9. Stop session
10. Export data
11. Verify the exported ZIP archive contains the expected events
    (5 presses + 5 infusions); live logs are also under
    `~/REACHER/LOG/<session>/`

### Expected Results:
- [ ] All devices respond correctly
- [ ] Data appears in real-time graph
- [ ] Data archive (ZIP) created with correct data
- [ ] No errors in console
```

### Regression Testing

Maintain a checklist of critical functionality:

```markdown
## Regression Test Checklist

### Serial Communication
- [ ] Connect to Arduino
- [ ] Send commands
- [ ] Receive data
- [ ] Handle disconnect gracefully

### Hardware Control
- [ ] Arm/disarm each device
- [ ] Trigger rewards
- [ ] Adjust parameters
- [ ] Timeout behavior

### Data Management
- [ ] Session start/stop
- [ ] Real-time logging
- [ ] Data export (ZIP archive)
- [ ] Multiple sessions

### UI Functionality
- [ ] Create sessions
- [ ] Switch between tabs
- [ ] Update graphs
- [ ] Error messages
```

---

## Building and Packaging

### Building REACHER Package

#### Create Python Wheel

```bash
cd reacher
source venv/bin/activate

# Bump the version first (never hand-edit):
python scripts/bump-version.py ...   # stamps pyproject.toml, __init__.py, firmware

# Build the wheel + sdist with the PEP 517 build frontend
pip install build
python -m build

# Output in dist/ (PyPI distribution name is reacher2p; import stays reacher)
# reacher2p-3.0.0-py3-none-any.whl
# reacher2p-3.0.0.tar.gz
```

There is no Debian/`stdeb` packaging path for reacher; it is published to PyPI as
`reacher2p` (the firmware hex ships as package data inside the wheel).

### Building Labrynth Application

Labrynth uses PyInstaller (driven by `build.py`) to create standalone executables.
`build.py` orchestrates the GUI build via `labrynth.spec` and the CLI/TUI build via
`labrynth-cli.spec`. The PyInstaller entry point is `launcher.py`, which starts the
reacher backend (it is not a GUI window). The frontend is built first with Vite
(`npm run build` in `web/`).

#### Prerequisites

```bash
cd labrynth
source venv/bin/activate
pip install pyinstaller
(cd web && npm install && npm run build)   # produces web/dist for the backend to serve
```

#### Build (all platforms)

```bash
cd labrynth
source venv/bin/activate  # or .\venv\Scripts\Activate.ps1 on Windows

# One command orchestrates PyInstaller (labrynth.spec + labrynth-cli.spec)
python build.py
```

Outputs by platform:

- **macOS:** `dist/Labrynth.app` (plus `dist/LabrynthCLI*`)
- **Windows:** `dist/Labrynth/Labrynth.exe` (plus `dist/LabrynthCLI/...`)
- **Linux:** `dist/Labrynth/Labrynth` (plus `dist/LabrynthCLI/...`)

### Automated Building with GitHub Actions

Example workflow (`.github/workflows/build-installers.yml`):

```yaml
name: Build Installers

on:
  push:
    tags:
      - 'v*'

jobs:
  build-windows:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      - name: Install dependencies
        run: |
          pip install .
          pip install pyinstaller
          (cd web && npm ci && npm run build)
      - name: Build executable
        run: python build.py
      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: labrynth-windows
          path: dist/Labrynth/Labrynth.exe

  build-macos:
    runs-on: macos-latest
    steps:
      # Similar to Windows

  build-linux:
    runs-on: ubuntu-latest
    steps:
      # Similar to Windows
```

---

## Version Management

### Semantic Versioning

Follow [Semantic Versioning](https://semver.org/): `MAJOR.MINOR.PATCH`

- **MAJOR:** Breaking changes (e.g., v1.0.0 → v2.0.0)
- **MINOR:** New features, backward compatible (e.g., v1.1.0 → v1.2.0)
- **PATCH:** Bug fixes, backward compatible (e.g., v1.1.1 → v1.1.2)

### Version File Locations

Never hand-edit these — use each repo's `scripts/bump-version.py` (reacher's script
also stamps the firmware version strings).

**REACHER:**
- `pyproject.toml`: `version = "3.0.0"`
- `src/reacher/__init__.py`: `__version__ = "3.0.0"`

**Labrynth:**
- `pyproject.toml` and `web/package.json`: `3.0.0`
- dependency pin in `pyproject.toml`: `reacher2p>=3.0.0`

**Firmware** (inside the reacher repo):
- `firmware/libraries/REACHERDevices/library.properties`: `version=3.0.0`
  (stamped by reacher's `scripts/bump-version.py`; each sketch's `SendIdentification()`
  reports the same string)

### Updating Versions

When preparing a release:

1. **Decide version number** based on changes
2. **Run `scripts/bump-version.py`** to stamp all version files (the sanctioned path)
3. **Update CHANGELOG.md**
4. **Create git tag** (PyPI/CI publishes on tag)

```bash
# Bump via the repo's script (see its --help for the exact invocation), then:
python scripts/bump-version.py
git add .
git commit -m "chore: Bump version to 3.0.0"
git tag -a v3.0.0 -m "Release v3.0.0"
git push origin main
git push origin v3.0.0
```

### Changelog Management

Maintain CHANGELOG.md in each repository:

```markdown
# Changelog

All notable changes to this project will be documented in this file.

## [3.0.0] - 2026-06-24

### Added
- Solenoid device support
- Additional API router endpoint

### Changed
- Improved serial connection stability

### Fixed
- Timestamp offset calculation bug

## [3.0.0-beta.7] - 2026-06-18

### Fixed
- Hardware control UI layout issue
- Data export path handling on Windows
```

---

## Release Process

### Pre-Release Checklist

- [ ] All tests passing
- [ ] Version numbers updated in all files
- [ ] CHANGELOG.md updated
- [ ] Documentation updated
- [ ] Integration testing completed
- [ ] BREAKING.md created (if breaking changes)
- [ ] MIGRATION.md created (if needed)

### Release Steps

There are two release tracks: **reacher** (which now ships the firmware hex as package
data) and **labrynth**. Release reacher first, then bump labrynth's `reacher2p` pin.

#### 1. REACHER Release (includes firmware hex)

```bash
cd reacher
git checkout main
git pull origin main

# If firmware changed: edit sketches/library, recompile, commit the hex
bash firmware/compile.sh
git add src/reacher/hex && git commit -m "chore: recompile firmware hex"

# Bump version (also stamps firmware version strings) and build the wheel
python scripts/bump-version.py
python -m build

# Tag — CI publishes reacher2p to PyPI on tag
git tag -a v3.0.0 -m "Release v3.0.0"
git push origin main
git push origin v3.0.0
```

**GitHub Release:**
- Upload wheel: `reacher2p-3.0.0-py3-none-any.whl`
- Upload sdist: `reacher2p-3.0.0.tar.gz`
- Mark as pre-release for beta/alpha tags

**PyPI:** Publishing is automated via a trusted publisher on tag (distribution name
`reacher2p`). No manual `twine upload` is required.

#### 2. Labrynth Application Release

```bash
cd labrynth
git checkout main
git pull origin main

# Bump the reacher2p pin in pyproject.toml to the just-released reacher version
# reacher2p>=3.0.0

# Build for each platform (or use CI/CD) — see Building and Packaging section
python build.py

# Tag
git tag -a v3.0.0 -m "Release v3.0.0"
git push origin v3.0.0
```

**GitHub Release:**
- Upload: `Labrynth.exe` / `Labrynth/` (Windows)
- Upload: `Labrynth.app` (macOS)
- Upload: `Labrynth/Labrynth` (Linux)
- (plus the corresponding `LabrynthCLI` artifacts)

### Post-Release

1. **Announce release:**
   - Update main README with latest version
   - Notify users/collaborators

2. **Monitor for issues:**
   - Watch GitHub Issues
   - Respond to bug reports promptly

3. **Update documentation:**
   - Ensure setup guides reference new version
   - Update compatibility matrix

---

## Contribution Guidelines

### For External Contributors

1. **Fork the repository** on GitHub

2. **Clone your fork:**
   ```bash
   git clone https://github.com/YOUR-USERNAME/reacher.git
   cd reacher
   git remote add upstream https://github.com/Otis-Lab-MUSC/reacher.git
   ```

3. **Create feature branch:**
   ```bash
   git checkout -b feature/your-feature
   ```

4. **Make changes and commit:**
   ```bash
   git add .
   git commit -m "feat: Add feature description"
   ```

5. **Push to your fork:**
   ```bash
   git push origin feature/your-feature
   ```

6. **Create Pull Request:**
   - Go to original repository on GitHub
   - Click "New Pull Request"
   - Select your fork and branch
   - Fill out PR template

### Pull Request Template

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix (non-breaking change fixing an issue)
- [ ] New feature (non-breaking change adding functionality)
- [ ] Breaking change (fix or feature causing existing functionality to break)
- [ ] Documentation update

## Testing
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manual testing completed

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Comments added for complex code
- [ ] Documentation updated
- [ ] No new warnings generated
- [ ] Tests added for new functionality

## Related Issues
Closes #123
```

### Code Review Process

1. **Automated checks run:**
   - Linting
   - Unit tests
   - Build verification

2. **Maintainer reviews:**
   - Code quality
   - Functionality
   - Documentation
   - Tests

3. **Revisions:**
   - Address feedback
   - Push updates to same branch

4. **Approval and merge:**
   - Maintainer merges PR
   - Branch deleted

---

## Best Practices Summary

### General

- ✅ Always work in feature branches
- ✅ Write clear commit messages
- ✅ Keep changes focused and atomic
- ✅ Update documentation with code
- ✅ Test thoroughly before pushing

### Python

- ✅ Follow PEP 8 style guide
- ✅ Use type hints
- ✅ Write docstrings
- ✅ Add unit tests for new functions
- ✅ Use virtual environments

### Arduino

- ✅ Follow consistent naming conventions
- ✅ Comment complex logic
- ✅ Keep functions small and focused
- ✅ Test on actual hardware
- ✅ Use Doxygen documentation

### Multi-Repository

- ✅ Coordinate breaking changes
- ✅ Maintain version compatibility
- ✅ Test integration end-to-end
- ✅ Document dependencies
- ✅ Create clear migration guides

---

## Additional Resources

### Documentation

- [REACHER Documentation](https://github.com/Otis-Lab-MUSC/reacher)
- [Labrynth Documentation](https://github.com/Otis-Lab-MUSC/labrynth)
- [Firmware Documentation](https://github.com/Otis-Lab-MUSC/reacher) — firmware source now lives in the reacher package under `firmware/`; the standalone [reacher-firmware](https://github.com/Otis-Lab-MUSC/reacher-firmware) repository is archived

### External Resources

- [Git Documentation](https://git-scm.com/doc)
- [Python Package Guide](https://packaging.python.org/)
- [Arduino Reference](https://www.arduino.cc/reference/en/)
- [PySerial Documentation](https://pyserial.readthedocs.io/)

### Community

- **GitHub Issues:** Report bugs and request features
- **Discussions:** Ask questions and share ideas
- **Email:** thejoshbq@proton.me (maintainer)
- **Lab Website:** http://www.otis-lab.org

---

## Citation

If using these resources, please cite:

> Doncheck, E.M. et al. Drug self-administration in head-fixed mice. *Nat. Protoc.* (2026). https://doi.org/10.1038/s41596-026-01406-1

---

*Last updated: June 2026*
