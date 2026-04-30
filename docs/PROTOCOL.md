# REACHER System Protocol Guide

> **Note:**
> **REACHER** — Rodent Experiment Application Controls and Handling Ecosystem for Research
>
> This is the abridged, top-to-bottom walkthrough for running a self-administration session
> with the REACHER system. The example uses the Fixed-Ratio (FR) paradigm to match the
> published protocol; all five paradigms (FR, PR, VI, Omission, Pavlovian) ship in v2.0.0
> and follow the same setup pattern.
>
> For long-form installation guidance, per-device testing, and platform-specific issues,
> see [SETUP_AND_USAGE.md](SETUP_AND_USAGE.md) and [TROUBLESHOOTING.md](TROUBLESHOOTING.md).

<a id="table-of-contents"></a>

### Table of Contents

| # | Section | Description |
|:-:|---------|-------------|
| 1 | [Prerequisites](#prerequisites) | System, hardware, and software requirements |
| 2 | [Install Labrynth](#install-labrynth) | Platform installer or pip install from source |
| 3 | [Flash Firmware](#flash-firmware) | Upload via Labrynth dashboard or Arduino IDE |
| 4 | [Wire the Rig](#wire-the-rig) | Pin-out reference and grounding |
| 5 | [Run a Fixed-Ratio Session](#run-a-fixed-ratio-session) | Connect, configure, start, stop |
| 6 | [Export Data](#export-data) | Backend logs and the user ZIP archive |
| 7 | [Testing and Troubleshooting](#testing-and-troubleshooting) | Pointer to TROUBLESHOOTING.md |
| 8 | [Downstream Analysis](#downstream-analysis) | Pynapse and Axplorer pointers |

> **Tip:**
> Click any section heading to expand it.

<!-- ============================================================ -->

<a id="prerequisites"></a>
<details open>
<summary><h2>1. Prerequisites</h2></summary>

> **Important:**
> Self-administration sessions involve continuous serial communication, real-time
> plotting, and on-disk logging. An under-powered machine can drop events or freeze
> mid-session.

**System requirements:**

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| **CPU** | Quad-core (e.g., Intel i3) | 6–8 core (e.g., Intel i5/i7, AMD Ryzen 5) |
| **RAM** | 8 GB | 16 GB |
| **Storage** | 256 GB SSD | 512 GB SSD |
| **OS** | Windows 10 (64-bit), macOS 11+, or Ubuntu 22.04 / Debian 12 | Ubuntu 22.04 LTS, Windows 11, macOS 13+ |

**Additional requirements:**

- An **Arduino UNO** (or compatible) connected via **USB-A to USB-B cable**
- A modern **web browser** (Chrome, Firefox, or Edge)
- The computer must remain **powered on and connected** to the Arduino for the entire session — do not let it sleep

**Software dependencies** (Python 3.10+, Node.js 18+, avrdude) are bundled inside the platform installer in [Section 2](#install-labrynth). They are only needed separately if you build from source — see [SETUP_AND_USAGE.md](SETUP_AND_USAGE.md).

> **Note:**
> If you plan to run multiple sessions simultaneously (one Arduino per session), each
> additional session increases CPU and memory usage. The recommended specs scale
> accordingly.

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="install-labrynth"></a>
<details>
<summary><h2>2. Install Labrynth</h2></summary>

Labrynth is the desktop application that hosts the REACHER backend, the React dashboard, and the firmware uploader.

### 2.1 Platform Installer (recommended)

Download the latest installer for your platform from the
[Labrynth releases page](https://github.com/Otis-Lab-MUSC/labrynth/releases).

| Platform | File | Notes |
|----------|------|-------|
| **Windows** | `REACHER-2.0.0-windows-x64.exe` | Requires administrator privileges; installs to `C:\Program Files\REACHER\` |
| **macOS** | `REACHER-2.0.0-macos-x64.dmg` | Right-click → **Open** on first launch (Gatekeeper bypass) |
| **Linux** | `REACHER-2.0.0-linux-amd64.deb` | Install with `sudo apt install ./REACHER-2.0.0-linux-amd64.deb` |

The installer bundles the REACHER backend (FastAPI/Uvicorn), the dashboard, the firmware uploader (`avrdude`), and pre-compiled firmware hex files for all five paradigms. **No separate Python or Node.js installation is needed when using the installer.**

### 2.2 Install from Source (for development or modification)

Requires Python 3.10+ and Node.js 18+. With a virtual environment active:

```bash
# Install the published package from PyPI
pip install labrynth
labrynth
```

For the full source-based workflow (cloning the repository, initializing the firmware submodule, building the React frontend, running the build pipeline), see [SETUP_AND_USAGE.md](SETUP_AND_USAGE.md).

### 2.3 First Launch

When you launch Labrynth:

1. The **REACHER backend starts headlessly** (FastAPI/Uvicorn) on port `6229`. There is no separate launcher window.
2. Your default web browser opens automatically to `http://localhost:6229`.
3. The dashboard loads in your browser.

> **Important:**
> The REACHER backend runs as a background process. Closing the browser tab does **not**
> shut down the backend — it continues running so you can reconnect. To stop the backend,
> use the application's shutdown menu or close the process.

> **Note:**
> If the browser does not open automatically, manually navigate to `http://localhost:6229`.

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="flash-firmware"></a>
<details>
<summary><h2>3. Flash Firmware</h2></summary>

The firmware tells the Arduino how to run a paradigm. Two upload methods are supported.

### 3.1 Upload from the Labrynth Dashboard (recommended)

This is the easiest method — `avrdude` and the firmware hex files are already bundled in the installer.

1. Connect the Arduino via USB.
2. Launch Labrynth. The dashboard opens at `http://localhost:6229`.
3. In the **Session** panel, click **Search Microcontrollers** and select the Arduino's port from the dropdown:
   - Windows: `COM3`, `COM4`, etc.
   - macOS: `/dev/cu.usbmodem*` or `/dev/cu.usbserial*`
   - Linux: `/dev/ttyACM0` or `/dev/ttyUSB0`
4. Select the **paradigm** (`fr`, `pr`, `vi`, `omission`, or `pavlovian`). For the Fixed-Ratio walkthrough that follows, choose `fr`.
5. Select the **board** (Arduino UNO).
6. Click **Upload Firmware**. Wait for the success message.
7. After upload, click **Connect** in the Session panel. On success, a three-tone connection jingle (500 Hz → 1000 Hz → 1500 Hz) plays on the cue speaker.

The same upload flow is also available from the terminal CLI — see [TROUBLESHOOTING.md Section 7](TROUBLESHOOTING.md#cli-usage).

### 3.2 Upload from the Arduino IDE (if modifying firmware source)

1. Install the [Arduino IDE](https://www.arduino.cc/en/software).
2. Install the **ArduinoJson** library via **Sketch → Include Library → Manage Libraries** (search "ArduinoJson by Benoit Blanchon" → Install).
3. Clone the [reacher-firmware](https://github.com/Otis-Lab-MUSC/reacher-firmware) repository, or use the `firmware/` submodule already inside your `labrynth` checkout.
4. Open the paradigm sketch:

| Paradigm | Sketch path |
|----------|-------------|
| Fixed-Ratio (FR) | `reacher-firmware/fr/fr.ino` |
| Progressive-Ratio (PR) | `reacher-firmware/pr/pr.ino` |
| Variable-Interval (VI) | `reacher-firmware/vi/vi.ino` |
| Omission | `reacher-firmware/omission/omission.ino` |
| Pavlovian | `reacher-firmware/pavlovian/pavlovian.ino` |

5. Set **Tools → Board → Arduino AVR Boards → Arduino UNO** and **Tools → Port → your Arduino's port**.
6. Click the **Upload** button (right-arrow icon). Wait for "Done uploading."
7. **Close the Arduino IDE (or at least its Serial Monitor) before launching Labrynth** — only one program can hold the serial port at a time.

> **Note:**
> All five paradigms share the `REACHERDevices` library at
> `reacher-firmware/libraries/REACHERDevices/` and are stable in v2.0.0. Pre-compiled
> hex files for all paradigms are committed in `reacher-firmware/hex/` and are also
> bundled inside the Labrynth installer.

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="wire-the-rig"></a>
<details>
<summary><h2>4. Wire the Rig</h2></summary>

The firmware expects each component on a specific Arduino UNO pin (defined in `Pins.h`). Pin assignments are identical across all five paradigms.

| Pin | Device | Direction | Description |
|-----|--------|-----------|-------------|
| 2 | Microscope timestamp | INPUT (INT0) | Receives frame sync signals from the imaging system |
| 3 | Cue speaker (primary) | OUTPUT | Plays tones |
| 4 | Pump (primary) | OUTPUT | Activates the primary infusion pump |
| 5 | Lick circuit | INPUT_PULLUP | Detects lick contact on the spout |
| 6 | Laser | OUTPUT | Optogenetic stimulation |
| 7 | Cue speaker (secondary) | OUTPUT | Secondary tone (Pavlovian CS-/advanced paradigms) |
| 8 | Pump (secondary) | OUTPUT | Secondary infusion pump |
| 9 | Microscope trigger | OUTPUT | Sends 50 ms trigger pulses to the imaging system |
| 10 | Right-hand (RH) lever | INPUT_PULLUP | Detects right lever presses |
| 12 | Left-hand (LH) lever | INPUT_PULLUP | Detects left lever presses |

> **Important:**
> Each component also needs a **ground (GND)** connection to one of the Arduino's GND
> pins. High-current components (lasers, some pumps) additionally require an external
> power supply.

For the Fixed-Ratio walkthrough that follows, you only need the levers, primary cue speaker, and primary pump — pins 10, 12, 3, 4, plus GND.

For wiring diagrams, optocoupler isolation, and 3D-printable enclosure parts (lever assemblies, syringe pumps, head-fixation station), see:

- [SETUP_AND_USAGE.md "Hardware Setup"](SETUP_AND_USAGE.md#hardware-setup) — full pin reference, wiring guidelines, manual hardware testing
- [TROUBLESHOOTING.md "System Overview"](TROUBLESHOOTING.md#system-overview) — wiring diagram, per-device test procedures
- [reacher-hardware-models](https://github.com/Otis-Lab-MUSC/reacher-hardware-models) — STL files for printable rig components

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="run-a-fixed-ratio-session"></a>
<details>
<summary><h2>5. Run a Fixed-Ratio Session</h2></summary>

With Labrynth running, the dashboard open at `http://localhost:6229`, and the Arduino connected via USB:

### 5.1 Create a Session

1. In the **Session** panel, type a unique name for the box (e.g., `Box1`, `Mouse_001_Session_3`).
2. Click **New wired session**. A tab labeled with your name appears in the dashboard.

### 5.2 Connect to the Arduino

1. Click **Search Microcontrollers** and select the Arduino's port from the dropdown.
2. Click **Connect**. On success:
   - The serial connection opens at **115200 baud**.
   - The **Firmware Information** panel populates with sketch name (e.g., `fr.ino`), version (`v2.0.0`), and schedule (`FIXED_RATIO`).
   - The connection jingle plays on the cue speaker (three ascending tones).

### 5.3 Configure Hardware (Hardware Tab)

In the **Hardware Tab**, arm the components used in your experiment using the toggle buttons. For the Fixed-Ratio walkthrough, arm at minimum:

| Toggle | Pin | Purpose |
|--------|-----|---------|
| **Arm RH Lever** | 10 | Detects right-hand lever presses |
| **Arm LH Lever** | 12 | Detects left-hand lever presses |
| **Arm Cue** | 3 | Plays reward-paired tone |
| **Arm Pump** | 4 | Delivers infusion |

Use the **Active Lever** menu to designate which lever delivers reward (LH or RH). Configure cue parameters (default 8000 Hz / 1600 ms) if needed; click the upload icon next to the cue settings to send the configuration to the Arduino.

For lick circuit, laser, secondary cue/pump, and microscope, arm and configure as appropriate.

### 5.4 Configure the Schedule (Schedule Tab)

In the **Schedule Tab**, set the within-trial dynamics:

| Parameter | Default | Range |
|-----------|---------|-------|
| **Fixed Ratio** | 1 (FR1 = every active press is rewarded) | 1–50 |
| **Timeout Duration** | 20 s | 0–600 s |
| **Trace Duration** | 0 s | 0–60 s |

Click the upload icon next to each slider to send the value to the Arduino.

### 5.5 Configure the Program (Program Tab)

In the **Program Tab**:

1. Optionally apply a preset:

| Preset | Limit Type | Infusions | Time | Stop Delay |
|--------|-----------|-----------|------|------------|
| SA High | Both | 10 | 1 hour | 10 s |
| SA Mid | Both | 20 | 1 hour | 10 s |
| SA Low | Both | 40 | 1 hour | 10 s |
| SA Extinction | Time | — | 1 hour | — |
| Custom | — | manual | manual | manual |

2. Or set the **Limit Type** (Time, Infusion, Both) and corresponding values manually. Click **Set Program Limit** to apply.
3. Set the export filename and folder. The placeholder/recommended folder is `~/REACHER/DATA/`. If left blank, exports default to `~/Downloads/`. Click **Set File Configuration** to apply.

### 5.6 Start the Session (Monitor Tab)

1. Switch to the **Monitor Tab**.
2. Click **Start** (play icon). The **Settings Overview** modal opens for confirmation.
3. Review settings and click **Ready to run?** to begin.
4. The session header updates and the live timeline begins displaying lever presses, infusions, licks, and other events as they occur.

### 5.7 Pause / Stop

- **Pause** (yellow): suspends dashboard processing; the Arduino keeps running. Click again to resume. Paused time is subtracted from the session duration.
- **Stop** (red): ends the session permanently. Sends an END command to the Arduino, closes the serial connection, and triggers data finalization.

> **Caution:**
> Always click **Stop** before unplugging the USB cable. If USB is pulled
> mid-session, behavioral data up to that point is preserved in `~/REACHER/LOG/`
> (see [Section 6](#export-data)), but session metadata may be incomplete and
> automatic export may not finish.

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="export-data"></a>
<details>
<summary><h2>6. Export Data</h2></summary>

When the session stops, the system produces two distinct outputs.

### 6.1 Backend Logs (always written)

Every session writes a timestamped folder of raw logs to:

```
~/REACHER/LOG/{YYYY-MM-DD_HH-MM-SS}/
```

This folder is written **regardless of whether you click Export**. Contents:

| File | What it contains |
|------|------------------|
| `behavior_events.csv` | Timestamped behavioral events (presses, infusions, licks, laser, ...) |
| `frame_timestamps.csv` | Imaging frame timestamps (if microscope was armed) |
| `controller_log.json` | Raw JSON messages received from the Arduino |
| `interface_log.log` | Application debug log |
| `event_log.jsonl` | Structured event log (one JSON object per line) |

Events are fsynced per-event, so this folder is the durable record even if the session ends abnormally.

### 6.2 User Export (Data Tab → Export ZIP)

To produce a portable ZIP archive of the session for sharing or analysis:

1. After stopping the session, switch to the **Data Tab**.
2. Optionally add notes in the notes field.
3. Click **Export ZIP**.
4. The archive lands at the destination configured in the Program Tab:
   - **Recommended:** `~/REACHER/DATA/` (the placeholder shown in the export field)
   - **If the destination field is left blank:** `~/Downloads/`

The ZIP contains:

| File | What it contains |
|------|------------------|
| `behavior_events.csv` | Timestamped behavioral events |
| `frame_timestamps.csv` | Imaging frame timestamps |
| `arduino_config.json` | Firmware identification and hardware/schedule settings |
| `metadata.json` | Session metadata: start/end times, chamber name, event counts |
| `notes.txt` | Optional notes entered in the Data Tab |

> **Note:**
> Timestamps in the CSV files are **milliseconds from session start** (not wall-clock
> time). Divide by 1000 for seconds, or by 60000 for minutes.

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="testing-and-troubleshooting"></a>
<details>
<summary><h2>7. Testing and Troubleshooting</h2></summary>

After completing a session, it is good practice to test each hardware component individually to verify correct operation. For per-device test procedures, diagnostic flowcharts, platform-specific issues, and recovery from unexpected stops, refer to:

**→ [TROUBLESHOOTING.md](TROUBLESHOOTING.md)**

Key sections:

- Per-device hardware testing (lever, cue, pump, laser, lick circuit, microscope)
- Serial connection and COM port issues
- Platform-specific problems (Windows, macOS, Linux)
- Data export validation and crash recovery
- Quick-reference flowcharts

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="downstream-analysis"></a>
<details>
<summary><h2>8. Downstream Analysis</h2></summary>

The CSV files and ZIP archive produced in [Section 6](#export-data) are the input format expected by two analysis packages from the same lab:

- [**pynapse**](https://github.com/Otis-Lab-MUSC/pynapse) — neural data engine for aligning calcium-imaging signals with REACHER behavioral events and extracting peri-event tensors.
- **axplorer** — peri-event analysis dashboard built on Pynapse, with PETH computation, response classification, behavioral summary, and figure export. Currently in development at the Otis Lab; not yet publicly released.

Both are outside the scope of this protocol guide. See `pynapse`'s README for installation and usage.

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

---

*Document version: REACHER-Suite v2.0.0.*
