# REACHER System Troubleshooting Guide

> **Note:**
> **REACHER** — Rodent Experiment Application Controls and Handling Ecosystem for Research
>
> This guide covers the entire pipeline from physical hardware setup through data export
> for all five paradigms (FR, PR, VI, Omission, Pavlovian) using wired connections. It is
> written for users with no prior programming or lab hardware experience.

<a id="table-of-contents"></a>

### Table of Contents

| # | Section | Description |
|:-:|---------|-------------|
| 1 | [System Overview](#system-overview) | Architecture, pin assignments, wiring |
| 2 | [Installation & First Launch](#installation) | Download, install, and first run |
| 3 | [Arduino & Firmware Setup](#arduino-firmware) | Hardware checklist, firmware upload, wiring verification |
| 4 | [Connecting to the Arduino](#connecting) | Session creation, COM ports, drivers |
| 5 | [Configuring an Experiment](#configuring) | Hardware arming, schedule timing, session limits |
| 6 | [Running an Experiment](#running) | Start, monitor, pause, stop, recovery |
| 7 | [CLI Usage](#cli-usage) | Terminal interface for experiment control |
| 8 | [Hardware Testing & Diagnostics](#hardware-testing) | Per-device testing procedures |
| 9 | [Data Export & Validation](#data-export) | Export, verification, raw logs |
| 10 | [Platform-Specific Issues](#platform-issues) | Windows, macOS, Linux |
| 11 | [Troubleshooting Flowcharts](#flowcharts) | Quick-reference decision trees |
| 12 | [Glossary](#glossary) | Terms and definitions |

> **Tip:**
> Click any section heading to expand it.

<!-- ============================================================ -->

<a id="system-overview"></a>
<details open>
<summary><h2>1. System Overview</h2></summary>

The REACHER system has three main components that work together:

```
+-------------------+       USB Cable         +-------------------+      Browser       +-------------------+
|                   |  (USB-A to USB-B)       |                   | (localhost:6229)   |                   |
|  Arduino UNO      | <-------------------->  |  Your Computer    | <----------------> |  Dashboard        |
|  (Firmware)       |    Serial @ 115200      |  (REACHER backend)|  FastAPI/Uvicorn   |  (React 19 App)   |
|                   |       baud              |                   |   REST + WebSocket |                   |
+-------------------+                         +-------------------+                    +-------------------+
  Controls hardware                            Runs the headless                        Where you interact
  (levers, pumps,                              FastAPI backend                          with the experiment
   speakers, laser)                            on port 6229
```

### What Each Piece Does

| Component | What It Is | What It Does |
|-----------|-----------|--------------|
| **Arduino UNO** | A small circuit board | Controls all physical hardware (levers, pumps, speakers, laser, lick circuit, imaging) |
| **Firmware** | Code uploaded to the Arduino | Listens for commands, monitors hardware, sends data back over USB |
| **REACHER backend** | Headless FastAPI/Uvicorn server on your computer | Manages serial communication, session state, data collection, and serves the dashboard |
| **Dashboard** | React 19 web application in your browser | Where you connect, configure, run, and monitor experiments |
| **CLI** | Terminal interface (prompt_toolkit) | Alternative to the browser — full experiment control from a terminal |

<details>
<summary><strong>Pin Assignment Reference</strong></summary>

The Arduino UNO connects to each hardware component through specific pins:

| Pin | Device | Direction | Description |
|-----|--------|-----------|-------------|
| 2 | Microscope Timestamp | INPUT (Interrupt) | Receives frame sync signals from imaging system |
| 3 | Cue Speaker (Primary) | OUTPUT | Plays tones through a connected speaker |
| 4 | Pump (Primary) | OUTPUT | Activates the primary infusion pump |
| 5 | Lick Circuit | INPUT_PULLUP | Detects lick contact on the spout |
| 6 | Laser | OUTPUT | Controls optogenetic laser stimulation |
| 7 | Cue Speaker (Secondary) | OUTPUT | Plays tones through a second speaker |
| 8 | Pump (Secondary) | OUTPUT | Activates the secondary infusion pump |
| 9 | Microscope Trigger | OUTPUT | Sends trigger pulses to imaging system |
| 10 | Right-Hand (RH) Lever | INPUT_PULLUP | Detects right lever presses |
| 12 | Left-Hand (LH) Lever | INPUT_PULLUP | Detects left lever presses |

</details>

<details>
<summary><strong>Wiring Reference Diagram</strong></summary>

```
                            Arduino UNO
                    +-------------------------+
                    |                     [13] |
                    |                     [12] |-----> Left-Hand Lever
                    |                     [11] |
                    |                     [10] |-----> Right-Hand Lever
                    |                      [9] |-----> Microscope Trigger (OUTPUT)
                    |                      [8] |-----> Secondary Pump
                    |                      [7] |-----> Secondary Cue Speaker
                    |                      [6] |-----> Laser
                    |                      [5] |-----> Lick Circuit
                    |                      [4] |-----> Primary Pump
                    |                      [3] |-----> Primary Cue Speaker
                    |                      [2] |-----> Microscope Timestamp (INPUT/INT0)
                    |                      [1] |
                    |                      [0] |
                    |                          |
                    |  [5V] [3.3V] [GND] [GND] |
                    |    |     |     |     |   |
                    +-------------------------+
                              |   |
                              +---+
                            USB-B Port
                               |
                         USB-A to USB-B
                             Cable
                               |
                          To Computer
```

</details>

> **Note:**
> Each hardware component also needs a ground (GND) connection. Use the
> GND pins on the Arduino. Some components (like the laser) may also require an
> external power supply.

### Serial Communication

- **Cable:** USB-A to USB-B (the same type used by many printers)
- **Baud rate:** 115200 (this is set automatically — you do not need to configure it)
- **Protocol:** JSON messages sent over the USB serial connection, newline-delimited

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="installation"></a>
<details>
<summary><h2>2. Installation & First Launch</h2></summary>

### 2.1 Download & Install

Download the latest installer for your platform from the
[Labrynth releases page](https://github.com/Otis-Lab-MUSC/labrynth/releases).
Release artifacts follow a `labrynth-{VERSION}-{platform}.{ext}` naming convention
(the `.deb` uses Debian's underscore form: `labrynth_{VERSION}_amd64.deb`).

<details>
<summary><h3>Windows</h3></summary>

1. Download `labrynth-{VERSION}-windows-x64.exe`
2. Double-click the installer
3. **You will need administrator privileges** — click "Yes" when prompted by Windows
4. Follow the Inno Setup wizard (accept defaults)
5. A desktop shortcut is created (optional)
6. The REACHER backend starts automatically after installation

</details>

<details>
<summary><h3>macOS</h3></summary>

1. Download `labrynth-{VERSION}-macos-arm64.dmg`
2. Double-click the `.dmg` file to mount it
3. Drag "Labrynth" to your Applications folder
4. **First launch — Gatekeeper bypass:**
   - Right-click (or Control-click) the app → select **Open**
   - Click **Open** in the dialog that appears
   - Alternatively: System Preferences → Security & Privacy → click **Open Anyway**
5. You only need to do step 4 once

</details>

<details>
<summary><h3>Linux</h3></summary>

Three artifacts are produced for Linux. Pick the one that matches your environment:

- **`.deb` package** (Debian / Ubuntu / Pop!_OS):
   ```bash
   sudo apt install ./labrynth_{VERSION}_amd64.deb
   labrynth
   ```
- **Portable tarball** (any glibc-based distro):
   ```bash
   tar -xzf labrynth-{VERSION}-linux-x64.tar.gz
   cd labrynth && ./labrynth
   ```
- **AppImage** (any glibc-based distro, no install):
   ```bash
   chmod +x labrynth-{VERSION}-linux-x64.AppImage
   ./labrynth-{VERSION}-linux-x64.AppImage
   ```

</details>

### 2.2 First Launch

When you launch REACHER:

1. The **backend starts headlessly** — there is no launcher window. The FastAPI/Uvicorn
   server starts in the background on port 6229.
2. Your default web browser opens automatically to `http://localhost:6229`
3. The dashboard loads in your browser with a dark theme
4. Alternatively, you can use the [CLI](#cli-usage) from a terminal

> **Important:**
> The backend runs as a background process. Closing the browser tab does not
> shut down the backend — it continues running so you can reconnect. To shut down
> the backend, use the shutdown endpoint or close the process.

<details>
<summary><strong>First Launch Troubleshooting</strong></summary>

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Browser doesn't open automatically | No default browser set, or port 6229 already in use | Manually open your browser and go to `http://localhost:6229`; if port is in use, close the other application using it |
| "Connection refused" in browser | The backend failed to start | Check the terminal/console for error messages; verify port 6229 is not already in use |
| Blank white page in browser | Slow initial load | Wait 10–15 seconds, then refresh the browser page |
| Backend doesn't start | Port 6229 is occupied by another process | Find and stop the process using port 6229, then restart REACHER |

</details>

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="arduino-firmware"></a>
<details>
<summary><h2>3. Arduino & Firmware Setup</h2></summary>

### 3.1 Required Hardware

Before connecting to the software, make sure you have:

- [ ] **Arduino UNO** board (or Arduino Mega for expanded I/O)
- [ ] **USB-A to USB-B cable** (connects the Arduino to your computer)
- [ ] **Lever switch(es)** — wired to pin 10 (right) and/or pin 12 (left)
- [ ] **Primary infusion pump** — wired to pin 4
- [ ] **Primary cue speaker** (8Ω impedance typical) — wired to pin 3
- [ ] **Laser** (if using optogenetics) — wired to pin 6, with external power supply
- [ ] **Lick circuit** (if measuring licks) — wired to pin 5
- [ ] **Imaging trigger/timestamp** (if using microscopy) — trigger on pin 9, timestamp on pin 2
- [ ] **Secondary cue speaker** (optional) — wired to pin 7
- [ ] **Secondary infusion pump** (optional) — wired to pin 8

### 3.2 Uploading Firmware

There are two methods for uploading firmware to the Arduino:

<details>
<summary><h4>Method 1: Built-in Upload (Dashboard or CLI)</h4></summary>

REACHER includes built-in firmware upload using `avrdude` and precompiled hex files.
No Arduino IDE installation is required.

1. Connect the Arduino via USB
2. In the dashboard, go to the **Session** panel
3. Select the target **port** from the dropdown
4. Select the **paradigm** (FR, PR, VI, Omission, or Pavlovian)
5. Select the **board** (Arduino UNO or Arduino Mega)
6. Click **Upload Firmware**
7. Wait for the upload to complete — a success message appears when finished

**Via CLI:**
```
reacher-cli
> Select: Upload Firmware
> Select port: COM3
> Select paradigm: fr
> Select board: uno
```

The built-in uploader uses precompiled `.hex` files from the `hex/` directory. These
are compiled from the source sketches during the build process.

</details>

<details>
<summary><h4>Method 2: Manual Upload (Arduino IDE)</h4></summary>

Use this method if you need to modify firmware source code before uploading.

1. **Install the Arduino IDE:**
   - Download from [https://www.arduino.cc/en/software](https://www.arduino.cc/en/software)
2. **Install the ArduinoJson library:**
   - Open Arduino IDE
   - Go to **Sketch → Include Library → Manage Libraries...**
   - Search for **"ArduinoJson"**
   - Click **Install**
3. **Install the REACHERDevices library:**
   - Copy the `libraries/REACHERDevices/` folder from the `reacher-firmware` repository
     into your Arduino libraries folder (typically `~/Arduino/libraries/`)
4. **Open the firmware file:**
   - Navigate to the paradigm folder (e.g., `reacher-firmware/fr/`)
   - Open the sketch file (e.g., `fr.ino`) in the Arduino IDE
5. **Select your board:**
   - Go to **Tools → Board → Arduino AVR Boards → Arduino UNO** (or Arduino Mega)
6. **Select your port:**
   - Go to **Tools → Port** and select the port showing your Arduino
   - Windows: `COM3`, `COM4`, etc.
   - macOS: `/dev/cu.usbserial-*` or `/dev/cu.usbmodem*`
   - Linux: `/dev/ttyUSB0` or `/dev/ttyACM0`
7. **Upload:**
   - Click the **Upload** button (right arrow icon) or press <kbd>Ctrl</kbd>+<kbd>U</kbd> / <kbd>Cmd</kbd>+<kbd>U</kbd>
   - Wait for "Done uploading" message

</details>

<details>
<summary><strong>Which Firmware for Which Paradigm</strong></summary>

| Paradigm | Firmware Folder | Sketch File | Status |
|----------|----------------|-------------|--------|
| Fixed-Ratio (FR) | `fr/` | `fr.ino` | Stable (v2 series) |
| Progressive-Ratio (PR) | `pr/` | `pr.ino` | Stable (v2 series) |
| Variable-Interval (VI) | `vi/` | `vi.ino` | Stable (v2 series) |
| Omission | `omission/` | `omission.ino` | Stable (v2 series) |
| Pavlovian | `pavlovian/` | `pavlovian.ino` | Stable (v2 series) |

All paradigms use the shared **REACHERDevices** library (`libraries/REACHERDevices/`)
which provides common device classes, pin assignments, and the Scheduler engine.

</details>

<details>
<summary><strong>Firmware Upload Troubleshooting</strong></summary>

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| "Board not found" in Arduino IDE | Wrong board selected or USB not connected | Go to Tools → Board → select **Arduino UNO** (or Mega); check USB cable is plugged in |
| Upload fails with "avrdude" error | Wrong COM port selected or Arduino not connected | Go to Tools → Port → select the correct port; try unplugging and replugging USB |
| `ArduinoJson.h: No such file or directory` | ArduinoJson library not installed | Install it: Sketch → Include Library → Manage Libraries → search "ArduinoJson" → Install |
| `REACHERDevices.h: No such file or directory` | REACHERDevices library not installed | Copy `libraries/REACHERDevices/` to your Arduino libraries folder |
| Built-in upload fails | avrdude not found or port is busy | Ensure no other program is using the port (close Arduino IDE Serial Monitor); check that avrdude is bundled with the installation |
| Upload succeeds but nothing happens | This is normal | The firmware waits for a connection from the dashboard before doing anything |
| Watchdog timer resets the Arduino | Firmware loop took longer than 8 seconds | This is a safety feature — if the main loop stalls, the watchdog resets the Arduino to prevent hardware damage |

</details>

### 3.3 Wiring Verification

After uploading firmware, verify each component is wired correctly:

<details>
<summary><strong>Wiring Verification Table</strong></summary>

| Pin | Device | How to Test |
|-----|--------|-------------|
| 10 | Right Lever | Press lever → check for response in dashboard ([Section 8.1](#81-lever-switch-testing)) |
| 12 | Left Lever | Press lever → check for response in dashboard ([Section 8.1](#81-lever-switch-testing)) |
| 3 | Primary Cue Speaker | Connect to Arduino → should hear jingle when dashboard connects ([Section 8.2](#82-cue-speaker-testing)) |
| 4 | Primary Pump | Use test button in Hardware Tab ([Section 8.3](#83-pump-testing)) |
| 5 | Lick Circuit | Touch spout → check for lick events in dashboard ([Section 8.5](#85-lick-circuit-testing)) |
| 6 | Laser | Use test button in Hardware Tab ([Section 8.4](#84-laser-testing)) |
| 7 | Secondary Cue Speaker | Arm secondary cue → use test button ([Section 8.2](#82-cue-speaker-testing)) |
| 8 | Secondary Pump | Arm secondary pump → use test button ([Section 8.3](#83-pump-testing)) |
| 9 | Microscope Trigger | Verify trigger output with oscilloscope or imaging software ([Section 8.6](#86-imaging-trigger--frame-timestamps)) |
| 2 | Microscope Timestamp | Verify frame signals are received ([Section 8.6](#86-imaging-trigger--frame-timestamps)) |

</details>

> **Important:**
> Always connect GND wires between the Arduino and each component.

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="connecting"></a>
<details>
<summary><h2>4. Connecting to the Arduino</h2></summary>

### 4.1 Creating a Session

1. In the dashboard, find the text field at the top of the page
2. Type a **unique name** for your behavior box (e.g., "Box1", "Chamber_A")
3. Click **"New wired session"**
4. A new session tab appears labeled with your name
5. Each session receives a unique **Session ID** for tracking

<details>
<summary><strong>Session State Machine</strong></summary>

Each session moves through these states:

```
idle → uploading → connected → running → paused → stopped
 │                    │           │          │
 │                    │           │          └──→ running (resume)
 │                    │           └──→ stopped
 │                    └──→ idle (disconnect)
 └──→ uploading (firmware upload)
```

| State | Meaning |
|-------|---------|
| **idle** | Session created, no Arduino connected |
| **uploading** | Firmware is being uploaded to the Arduino |
| **connected** | Arduino connected, ready to configure |
| **running** | Experiment is actively running |
| **paused** | Experiment paused — can resume or stop |
| **stopped** | Experiment ended — data auto-exported |

</details>

<details>
<summary><strong>Troubleshooting — Session Creation</strong></summary>

| Problem | Error Message | Fix |
|---------|--------------|-----|
| Left the name field empty | "Please enter a name and try again." | Type a name before clicking the button |
| Used a name that's already in use | "Name entered already exists. Please enter a different name." | Choose a different name |

</details>

### 4.2 Finding the COM Port

1. Make sure your Arduino is connected via USB
2. In the **Session** panel, click **"Search Microcontrollers"**
3. A dropdown list of available ports appears
4. Select your Arduino's port from the dropdown
5. Click **Connect**

**Port names by platform:**

| Platform | Typical Port Name |
|----------|------------------|
| Windows | `COM3`, `COM4`, `COM5`, etc. |
| Linux | `/dev/ttyUSB0` or `/dev/ttyACM0` |
| macOS | `/dev/cu.usbserial-*` or `/dev/cu.usbmodem*` |

> **Important:**
> **Port locking:** Each serial port can only be used by one session at a time.
> If a port is already in use by another session, it will not appear in the
> dropdown or will show as locked. Disconnect the other session first.

<details>
<summary><strong>Connection Troubleshooting</strong></summary>

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| No ports appear in dropdown | Arduino not plugged in | Plug in the USB cable, then click "Search Microcontrollers" again |
| No ports appear (Arduino IS plugged in) | Missing USB-to-serial driver | Install the driver for your Arduino's USB chip (see below) |
| Port appears but connection fails | Port is in use by another program or session | Close the Arduino IDE Serial Monitor or any other serial program; check for other REACHER sessions using the port |
| "Loaded firmware is incompatible. Please upload a qualified file." | Wrong firmware uploaded to Arduino | Upload the correct firmware ([Section 3.2](#32-uploading-firmware)) |
| Connection succeeds then immediately drops | Faulty USB cable or loose connection | Try a different USB cable; ensure both ends are firmly plugged in |
| Port shows as locked | Another REACHER session is using this port | Disconnect the other session first, or use a different Arduino |

</details>

<details>
<summary><strong>Multi-Session Usage</strong></summary>

REACHER supports running multiple sessions simultaneously, each connected to a
different Arduino on a different serial port. This is useful for running multiple
behavior boxes at the same time.

**Rules:**
- Each session must use a unique serial port (port locking is enforced)
- Each session has its own Session ID and independent state
- Sessions can be in different states (one running, another configuring)
- Each session exports its own independent data files

</details>

<details>
<summary><strong>Driver Installation</strong></summary>

Many Arduino clones use USB-to-serial chips that require separate drivers:

| Chip | Common On | Driver Download |
|------|-----------|----------------|
| CH340/CH341 | Most Arduino clones | [CH340 Driver](https://www.wch-ic.com/downloads/CH341SER_EXE.html) |
| CP2102 | Some Arduino clones | [CP210x Driver](https://www.silabs.com/developers/usb-to-uart-bridge-vcp-drivers) |
| FTDI FT232R | Official Arduinos, some clones | [FTDI Driver](https://ftdichip.com/drivers/vcp-drivers/) |

**Installation steps:**
1. Download the correct driver for your chip (check the markings on the chip near the USB port)
2. Run the installer
3. Restart your computer
4. Reconnect the Arduino and try again

</details>

#### Linux Serial Port Permissions

On Linux, you may not have permission to access serial ports by default:

```bash
sudo usermod -a -G dialout $USER
```

**You must log out and log back in** (or restart) for this to take effect.

### 4.3 Verifying Connection

When the connection succeeds:

1. **Connection jingle plays** — three ascending tones (500 Hz → 1000 Hz → 1500 Hz)
   from the cue speaker
2. **Firmware information appears** in the Session panel, showing:
   - Sketch name: e.g., `fr.ino`
   - Version: `v2.0.0`
   - Baud rate: `115200`
   - Schedule: e.g., `FIXED_RATIO`

**If no jingle plays:**
- Check that the cue speaker is wired to **pin 3** and a **GND** pin
- The jingle only plays if the speaker is physically connected — it does not require
  arming the cue in the Hardware Tab
- See [Section 8.2](#82-cue-speaker-testing) for speaker diagnostics

### 4.4 Remote-Peripheral Pairing (Raspberry Pi)

If you're running the Arduino on a separate networked peripheral (e.g., a Raspberry Pi running headless `reacher`) instead of the same machine that hosts Labrynth, the Machines panel handles discovery and pairing. Most failures fall into a few categories.

<details>
<summary><strong>Pairing Troubleshooting</strong></summary>

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Peripheral never appears in the Machines panel | mDNS multicast blocked on the LAN | On the peripheral, set `REACHER_BROKER_URL` to a reachable broker URL and restart `reacher`. The peripheral will then self-register via unicast HTTP. |
| Peripheral appears, but pairing fails with "invalid code" | Code expired (codes rotate periodically) | Read the current code from the peripheral's terminal (or `journalctl -u reacher.service`) and try again. |
| "Cannot reach machine" after pairing | Firewall blocking port 6229 on the peripheral | Open inbound TCP 6229 on the peripheral. On Raspberry Pi OS: `sudo ufw allow 6229/tcp`. |
| Peripheral on a different VLAN than the host | Routing / multicast domain mismatch | Move both onto the same VLAN, or use the `REACHER_BROKER_URL` fallback. |
| Peripheral shows offline despite being powered | `reacher.service` not running on the Pi | `sudo systemctl status reacher.service`. If inactive, `sudo systemctl start reacher.service` and check the journal for errors. |
| Pairing succeeds but COM ports list is empty for the remote machine | Arduino is plugged into the host, not the peripheral | The Arduino must be plugged into the peripheral whose backend owns it. Move the cable to the Pi. |

</details>

> **Note:**
> All host ↔ peripheral traffic is proxied through the host's local backend
> (`/api/proxy/{deviceId}/...`); the peripheral's API key is stored on the host's
> backend, never in the browser. WebSocket connections fetch a short-lived token
> before opening, so the API key is never sent over the wire from the browser.

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="configuring"></a>
<details>
<summary><h2>5. Configuring an Experiment</h2></summary>

After connecting to the Arduino, configure your experiment across three tabs:
Hardware, Schedule, and Program.

### 5.1 Hardware Tab — Arming Devices

**What "arming" means:** When a device is armed, it is enabled and will participate in
the experiment. When disarmed, the device is ignored — it will not respond to events
or generate data. Think of it like a safety switch.

Each device has a toggle button showing a lock icon:
- **Locked:** Device is disarmed (off)
- **Unlocked:** Device is armed (on)

**Arming each component:**

| Device | Pin | What to Do | What It Enables |
|--------|-----|-----------|-----------------|
| RH Lever | 10 | Toggle the Right-Hand Lever arm button | Detects presses on the right lever |
| LH Lever | 12 | Toggle the Left-Hand Lever arm button | Detects presses on the left lever |
| Active Lever | — | Select which lever (LH or RH) is the "active" (reinforced) lever | Presses on the active lever count toward rewards |
| Primary Cue | 3 | Toggle the Cue arm button | Plays a tone when the animal earns a reward |
| Primary Pump | 4 | Toggle the Pump arm button | Delivers infusion when the animal earns a reward |
| Secondary Cue | 7 | Toggle the Secondary Cue arm button | Plays a tone on the secondary speaker (for Pavlovian CS+/CS- or advanced paradigms) |
| Secondary Pump | 8 | Toggle the Secondary Pump arm button | Delivers infusion through the secondary pump |
| Laser | 6 | Toggle the Laser arm button | Activates laser stimulation (if using optogenetics) |
| Lick Circuit | 5 | Toggle the Lick Circuit arm button | Records lick events |
| Microscope | 9, 2 | Toggle the Microscope arm button | Records imaging frame timestamps |

> **Note:**
> The Arduino Mega provides additional I/O pins for expanded hardware configurations.
> When using a Mega, select "Arduino Mega" as the board type during firmware upload.

<details>
<summary><strong>Hardware Troubleshooting</strong></summary>

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Device won't arm (no response when clicking) | Arduino not connected | Go to the Session panel and verify connection status |
| Pump test runs but no fluid delivers | Tubing disconnected or pump wired incorrectly | Check tubing connections; verify pump is wired to **pin 4** (primary) or **pin 8** (secondary) |
| Cue test plays no sound | Speaker not connected or wired wrong | Check speaker is wired to **pin 3** (primary) or **pin 7** (secondary) and GND; test speaker independently |
| Laser test does nothing | Laser not connected or not powered | Check wiring to **pin 6**; verify external power supply for the laser |
| Lever shows no response to press | Wrong pin or wiring issue | Right lever → **pin 10**, Left lever → **pin 12**; check switch continuity with a multimeter |

</details>

### 5.2 Schedule Tab — Timing Parameters

These parameters control the timing and rules of your experiment. All times are
entered in **seconds** in the dashboard and converted to milliseconds for the Arduino.

<details>
<summary><strong>Fixed-Ratio (FR) Parameters</strong></summary>

| Parameter | What It Means | Default | Range |
|-----------|--------------|---------|-------|
| **Fixed Ratio** | How many lever presses before a reward is given | 1 (FR1 = every press is rewarded) | 1–50 |
| **Timeout Duration** | Period following reward delivery where rewards do not occur — active presses during this period do not trigger cue or reward | 20 seconds | 0–600 seconds |
| **Trace Duration** | Gap between the cue tone ending and the pump starting | 0 seconds | 0–60 seconds |

**How these work together (FR1 example with defaults):**

```
Animal presses active lever
        |
        v
  Cue tone plays (8000 Hz for 1.6 seconds)
        |
        v
  Pump activates (runs for 2.0 seconds)
        |
        v
  Timeout starts (20 seconds)
  During timeout, active presses are classified as TIMEOUT
        |
        v
  Timeout ends — next press can earn another reward
```

</details>

<details>
<summary><strong>Progressive-Ratio (PR) Parameters</strong></summary>

| Parameter | What It Means | Default |
|-----------|--------------|---------|
| **Starting Ratio** | Initial number of presses required for the first reward | 1 |
| **Step Size** | How much the ratio increases after each reward | Configurable |
| **Timeout Duration** | Post-reward timeout period | 20 seconds |
| **Breakpoint** | Session ends when the animal fails to complete the current ratio within the breakpoint time | Configurable |

The PR paradigm requires the `pr.ino` firmware.

</details>

<details>
<summary><strong>Variable-Interval (VI) Parameters</strong></summary>

| Parameter | What It Means | Default |
|-----------|--------------|---------|
| **Mean Interval** | Average time between reward availability windows | Configurable |
| **Timeout Duration** | Post-reward timeout period | 20 seconds |

The VI paradigm requires the `vi.ino` firmware. A press during the availability
window triggers the reward chain.

</details>

<details>
<summary><strong>Omission Parameters</strong></summary>

| Parameter | What It Means | Default |
|-----------|--------------|---------|
| **Omission Duration** | How long the animal must withhold pressing to earn a reward | Configurable |
| **Timeout Duration** | Post-reward timeout period | 20 seconds |

The Omission paradigm requires the `omission.ino` firmware. A reward is delivered
when the animal does NOT press for the configured duration.

</details>

<details>
<summary><strong>Pavlovian Parameters</strong></summary>

| Parameter | What It Means | Default |
|-----------|--------------|---------|
| **CS+ Duration** | Duration of the conditioned stimulus (rewarded) | Configurable |
| **CS- Duration** | Duration of the non-rewarded stimulus | Configurable |
| **ITI** | Inter-trial interval between trials | Configurable |
| **Number of Trials** | Total trials per session | Configurable |

The Pavlovian paradigm requires the `pavlovian.ino` firmware and uses a separate
`PavlovianScheduler` with a trial-based state machine. This paradigm may use the
secondary cue speaker (pin 7) for CS- tones and the secondary pump (pin 8).

</details>

> **Warning:**
> **Paradigm-specific parameters** are only active when the matching firmware is
> uploaded to the Arduino. For example, PR parameters are ignored if the Arduino is
> running `fr.ino`. Always ensure the firmware matches the paradigm you intend to run.

> **Warning:**
> **Timeout set to 0:** The animal can earn rewards as fast as it can press — this is
> usually not desired.

### 5.3 Program Tab — Session Limits & File Configuration

#### Session Limits

You can set the experiment to stop automatically based on time, infusion count, or both:

| Limit Type | What It Does |
|-----------|--------------|
| **Time** | Stops the session after a set duration (hours, minutes, seconds) |
| **Infusion** | Stops the session after a set number of infusions are delivered |
| **Both** | Stops when EITHER the time limit OR infusion limit is reached (whichever comes first) |

**Presets** are available for common configurations:

| Preset | Limit Type | Infusions | Time | Stop Delay |
|--------|-----------|-----------|------|------------|
| SA High | Both | 10 | 1 hour | 10 seconds |
| SA Mid | Both | 20 | 1 hour | 10 seconds |
| SA Low | Both | 40 | 1 hour | 10 seconds |
| SA Extinction | Time | — | 1 hour | — |
| Custom | — | Set your own | Set your own | Set your own |

#### File Configuration

- **Filename:** Enter a name for your data files (e.g., `experiment1`)
- **Save folder:** Choose where the user-initiated **Export ZIP** archive will land
  - Recommended (placeholder shown in the field): `~/REACHER/DATA/`
  - If the field is left blank: `~/Downloads/`
- **Backend logs** are written automatically to `~/REACHER/LOG/{timestamp}/` every session — separate from the user-initiated ZIP export. See [Section 9](#data-export) for the distinction.

<details>
<summary><strong>Program Tab Troubleshooting</strong></summary>

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Can't select save directory | Permissions issue | Choose a folder in your home directory (e.g., Desktop, Documents) |
| Session stops immediately after starting | Time limit set to 0 or very low value | Check the time limit in the Program Tab |

</details>

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="running"></a>
<details>
<summary><h2>6. Running an Experiment</h2></summary>

### 6.1 Starting the Experiment

1. Go to the **Monitor Tab**
2. A confirmation dialog appears showing your settings:
   - Schedule type, ratio, timeout, armed devices, limits
3. Review the settings and click **Start**
4. The Arduino enters the program state, timestamps begin at 0, and data flows

**Verification:** The first lever press should appear on the real-time graph within
a few seconds.

### 6.2 Real-Time Monitoring

The dashboard shows a timeline plot that updates every **5 seconds**.

**Event colors on the timeline:**

| Event | Color |
|-------|-------|
| Active Press | Red |
| Timeout Press | Grey |
| Inactive Press | Black |
| Lick | Pink |
| Infusion | Red |
| Laser Stimulation | Green |
| Session Start | Red dashed line |

<details>
<summary><strong>Monitoring Troubleshooting</strong></summary>

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Graph shows no data | No behavioral events have occurred yet | Wait for the animal to press the lever |
| Graph shows no data (events ARE happening) | Dashboard update stalled | Refresh the browser page; check that the experiment is not paused |
| Graph stopped updating mid-session | Serial connection lost or WebSocket disconnected | Check the USB cable; the WebSocket will auto-reconnect if the backend is still running |
| Lever presses show as TIMEOUT when they shouldn't | Timeout period is still active from the previous reward | Wait for the timeout to expire (default 20 seconds); adjust timeout in the Schedule Tab if needed |
| All presses show as INACTIVE | Wrong lever set as active | Go to the Hardware Tab and check which lever is set as the "active" lever |

</details>

### 6.3 Pausing & Resuming

- **Pause:** Click the Pause button in the Monitor Tab
  - The Arduino keeps running, but the dashboard stops processing incoming data
  - Use this when: animal is in distress, equipment needs adjustment
- **Resume:** Click Resume to continue
  - Data collection picks up where it left off
  - Paused time is subtracted from the session duration

### 6.4 Stopping the Experiment

1. Click the **Stop** button in the Monitor Tab
2. This sends an END command to the Arduino, closes the serial connection, and
   finalizes the data
3. **Data is auto-exported** to `~/REACHER/LOG/{TIMESTAMP}/` containing:
   - `behavior_events.csv` — all behavioral event data
   - `frame_timestamps.csv` — imaging frame timestamps (if microscope was armed)
   - `controller_log.json` — raw JSON messages from the Arduino
   - `interface_log.log` — application debug log
   - `event_log.jsonl` — structured event log (one JSON object per line)
4. **Always stop the experiment before disconnecting the USB cable**

> **Caution:**
> **If USB is pulled without stopping:** Data collected up to that point is
> preserved in the log files (`~/REACHER/LOG/`), but the session metadata may be
> incomplete and the auto-export may not complete properly.

### 6.5 Session Recovery

**Single-tab enforcement:** The dashboard enforces a single active tab per session.
If you open the same session URL in a second tab, it will warn you.

**Page reload recovery:** If you refresh the browser page during a running experiment,
the dashboard reconnects to the backend via WebSocket and restores the session state.
The backend continues running independently of the browser.

**Tab close behavior:** Closing the browser tab sends a shutdown signal to the backend.
If an experiment is running, the backend will auto-export data before shutting down.

<a id="65-unexpected-stops--recovery"></a>

### 6.6 Unexpected Stops & Recovery

If your experiment stops unexpectedly, follow this decision tree:

```
Experiment stopped unexpectedly
|
+-- Did the backend crash?
|   |
|   +-- YES --> Restart REACHER
|   |           Your data is saved in ~/REACHER/LOG/
|   |           Look for the most recent timestamped folder
|   |
|   +-- NO
|       |
|       +-- Did the Arduino reset? (power LED blinks off then on)
|       |   |
|       |   +-- YES --> USB cable issue or watchdog timer reset
|       |   |           Try a different cable
|       |   |           If watchdog: check for firmware loop stalls
|       |   |           Reconnect and restart the session
|       |   |
|       |   +-- NO
|       |       |
|       |       +-- Was a session limit reached?
|       |       |   |
|       |       |   +-- Check Program Tab for time/infusion limits
|       |       |   |   The session may have ended normally
|       |       |   |
|       |       |   +-- NO
|       |       |       |
|       |       |       +-- Check the terminal/console for error messages
|       |       |           Report the error if you cannot resolve it
```

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="cli-usage"></a>
<details>
<summary><h2>7. CLI Usage</h2></summary>

The REACHER CLI provides a terminal-based interface for full experiment control
without a browser. It uses `prompt_toolkit` for interactive menus and is installed
as the `reacher-cli` command.

### 7.1 Starting the CLI

```bash
reacher-cli
```

The CLI auto-detects whether the REACHER backend is running. If not, it starts
the backend automatically before presenting the main menu.

### 7.2 CLI Modes

| Mode | Description |
|------|-------------|
| **Menu** | Navigate options using arrow keys and Enter |
| **Input** | Type values (session names, file paths, parameters) |
| **Select** | Choose from lists (ports, paradigms, boards) |
| **Monitor** | Live session monitoring — displays events as they arrive |

### 7.3 Available Commands

From the main menu, you can:

- **Create a session** — specify a name and create a new wired session
- **Upload firmware** — select port, paradigm, and board for firmware upload
- **Connect to Arduino** — select a port and connect
- **Configure hardware** — arm/disarm devices
- **Set schedule parameters** — configure timing and ratio settings
- **Start / Pause / Stop** — control the experiment
- **Export data** — trigger a ZIP export of session data
- **View session status** — see current state, events, and counters

### 7.4 CLI Troubleshooting

<details>
<summary><strong>CLI Issues</strong></summary>

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| `reacher-cli: command not found` | CLI not installed | Install with `pip install -e ".[cli]"` from the labrynth directory |
| CLI starts but can't detect backend | Backend is not running and auto-start failed | Start the backend manually: `python -m reacher` |
| CLI is unresponsive or garbled display | Terminal doesn't support prompt_toolkit features | Try a different terminal emulator (e.g., Windows Terminal, iTerm2, GNOME Terminal) |
| "Connection refused" when connecting | Backend crashed or port 6229 is unavailable | Restart the backend: `python -m reacher` |
| Serial port not listed | Port permissions (Linux) or missing driver | See [Section 4.2](#42-finding-the-com-port) for driver installation; on Linux, add your user to the `dialout` group |

</details>

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="hardware-testing"></a>
<details>
<summary><h2>8. Hardware Testing & Diagnostics</h2></summary>

Use these procedures to test each hardware component independently. This helps
isolate whether a problem is with the wiring, the component, or the software.

<details>
<summary><h3>8.1 Lever Switch Testing</h3></summary>

**Physical test:**
1. Use a multimeter set to continuity mode
2. Place one probe on the lever's signal wire and one on the common wire
3. Press the lever — the multimeter should beep (circuit closed)
4. Release — the circuit should open

**Software test:**
1. Connect to the Arduino in the dashboard
2. Arm the lever (RH or LH) in the Hardware Tab
3. Press the lever
4. Check the terminal/log for press events

**Expected output format:**
```
Device: SWITCH_LEVER | Event: PRESS | Class: ACTIVE/TIMEOUT/INACTIVE
Pin: 10 (RH) or 12 (LH)
Orientation: RH or LH
```

**In the dashboard data, events appear as:**
- `RH_LEVER` / `LH_LEVER` (device)
- `ACTIVE_PRESS` / `TIMEOUT_PRESS` / `INACTIVE_PRESS` (event type)

**Debounce note:** The firmware uses a 20ms debounce delay. This means very rapid
presses (faster than 20ms apart) may be missed. This is normal and prevents
false readings from switch bounce.

</details>

<a id="82-cue-speaker-testing"></a>
<details>
<summary><h3>8.2 Cue Speaker Testing</h3></summary>

**Physical test:**
1. Briefly touch the speaker wires to the Arduino's 3.3V and GND pins
2. You should hear a faint click or pop — this confirms the speaker works

**Software test (primary cue — pin 3):**
1. Connect to the Arduino — the **connection jingle** should play automatically
   (three ascending tones: 500 Hz → 1000 Hz → 1500 Hz, each 100ms long)
2. Arm the primary cue in the Hardware Tab
3. Trigger a reward (press the active lever enough times to meet the ratio)
4. Listen for the cue tone

**Software test (secondary cue — pin 7):**
1. Arm the secondary cue in the Hardware Tab
2. Use the test button for the secondary cue
3. Listen for the tone from the second speaker

**Default tone settings:**
- Frequency: 8000 Hz
- Duration: 1600 ms (1.6 seconds)
- Trace interval: 0 ms

**No sound troubleshooting:**
1. Verify speaker is wired to **pin 3** (primary) or **pin 7** (secondary) and **GND**
2. Check speaker impedance — 8Ω is typical
3. Try a different speaker
4. If the connection jingle doesn't play, the issue is wiring or the speaker itself

</details>

<details>
<summary><h3>8.3 Pump Testing</h3></summary>

**Software test (primary pump — pin 4):**
1. Connect to the Arduino
2. Arm the primary pump in the Hardware Tab
3. Use the test button in the Hardware Tab to run the pump

**Software test (secondary pump — pin 8):**
1. Arm the secondary pump in the Hardware Tab
2. Use the test button for the secondary pump

**What to verify:**
- Pump motor runs when activated
- Tubing is connected and fluid flows
- Default infusion duration: 2000 ms (2 seconds)

**No delivery troubleshooting:**
1. Check pump is wired to **pin 4** (primary) or **pin 8** (secondary) and **GND**
2. Verify tubing connections at both ends
3. Check pump power supply (some pumps need external power)
4. Verify pump runs during the test — if the motor spins but no fluid, the tubing
   is the issue

</details>

<details>
<summary><h3>8.4 Laser Testing</h3></summary>

> **Caution:**
> **SAFETY: ALWAYS wear appropriate laser safety eyewear when testing the laser.**

**Software test:**
1. Connect to the Arduino
2. Arm the laser in the Hardware Tab
3. Use the laser test button (sends test command 603)
4. The laser should activate for the configured duration

**Laser modes:**
- **Contingent (Active-Press):** Laser fires only when triggered by a lever press reward
- **Independent (Cycle):** Laser cycles on/off continuously at the set duration

**Default settings:**
- Frequency: 40 Hz (pulsed, 50% duty cycle — 12.5ms on, 12.5ms off)
- Duration: 5000 ms (5 seconds)
- Trace interval: 1600 ms (starts after cue finishes)

**No output troubleshooting:**
1. Check laser is wired to **pin 6** and **GND**
2. Verify external power supply is connected and on
3. Check laser interlock (safety switch) is in the correct position
4. Test with a lower frequency (frequency = 1 gives continuous ON)

</details>

<details>
<summary><h3>8.5 Lick Circuit Testing</h3></summary>

**Software test:**
1. Connect to the Arduino
2. Arm the lick circuit in the Hardware Tab
3. Touch the lick spout (or bridge the lick circuit contacts)
4. Check the terminal/log for lick events

**Expected output:**
- Device: `LICK_CIRCUIT`
- Event: `LICK`
- Start timestamp (when contact was made) and end timestamp (when contact was broken)

**Sensitivity issues:**
1. Check wiring to **pin 5** and **GND**
2. Ensure proper grounding — the lick circuit requires a good ground connection
3. Check for moisture on contacts (can cause false readings)
4. The lick circuit uses the same 20ms debounce as the levers

</details>

<a id="86-imaging-trigger--frame-timestamps"></a>
<details>
<summary><h3>8.6 Imaging Trigger / Frame Timestamps</h3></summary>

**Software test:**
1. Connect to the Arduino
2. Arm the microscope in the Hardware Tab
3. Start a session — the Arduino sends a trigger pulse on **pin 9**
4. Verify frame timestamp signals are being received on **pin 2**

**How it works:**
- **Pin 9 (OUTPUT):** Sends a 50ms HIGH pulse to trigger the imaging system
- **Pin 2 (INPUT, Interrupt):** Receives frame sync signals from the camera/microscope
  on the RISING edge
- Each frame signal triggers an interrupt (ISR) that records the timestamp

**No timestamps troubleshooting:**
1. Verify trigger wire is connected to **pin 9**
2. Verify frame signal wire is connected to **pin 2**
3. Check that the microscope/camera is actually sending frame sync signals
4. Verify the microscope is armed in the Hardware Tab

</details>

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="data-export"></a>
<details>
<summary><h2>9. Data Export & Validation</h2></summary>

### 9.1 Auto-Export on Stop

When you stop an experiment, data is **automatically exported** to:

```
~/REACHER/LOG/<YYYY-MM-DD_HH-MM-SS>/
```

Each session folder contains:

| File | What It Contains |
|------|-----------------|
| `behavior_events.csv` | Columns: device, event, class, orientation, start_timestamp, end_timestamp |
| `frame_timestamps.csv` | List of imaging frame timestamps (if microscope was armed) |
| `controller_log.json` | Raw JSON messages received from the Arduino |
| `interface_log.log` | Application debug log |
| `event_log.jsonl` | Structured event log (one JSON object per line) |

### 9.2 ZIP Export

You can also export a ZIP archive containing all session data. **This is a separate, user-initiated step from the auto-export of backend logs in Section 9.1.**

1. Go to the **Data Tab**
2. Click the **Export ZIP** button
3. The ZIP file is saved to the location specified in the Program Tab:
   - Recommended (placeholder shown in the export field): `~/REACHER/DATA/`
   - If the destination field is left blank: `~/Downloads/`

**ZIP archive contents:**

| File | What It Contains |
|------|-----------------|
| `behavior_events.csv` | All behavioral event data |
| `frame_timestamps.csv` | Imaging frame timestamps |
| `metadata.json` | Session summary: start time, end time, chamber name, event counts |
| `arduino_config.json` | Hardware and schedule configuration as sent to the Arduino |
| `notes.txt` | Session notes (if any were entered in the Data Tab) |

### 9.3 Verifying Exported Data

After exporting, open the CSV files in any spreadsheet application or data analysis tool:

1. **Check row counts:** The number of rows in `behavior_events.csv` should match the
   number of events you observed during the session
2. **Verify timestamps are sequential:** Each start_timestamp should be greater than
   the previous one
3. **Cross-reference infusion count:** The number of rows with event "INFUSION" should
   match what you expected
4. **Check metadata:** Open `metadata.json` in the ZIP to see event counts broken down
   by type

<details>
<summary><strong>Data Troubleshooting</strong></summary>

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Auto-export didn't run | Session was interrupted (USB pulled, backend crash) | Check `~/REACHER/LOG/` for the most recent folder — raw logs are still saved |
| Export ZIP button does nothing | No data has been collected | Run an experiment first — there must be at least one event |
| CSV files are empty | Experiment stopped before any events occurred | Verify that events actually happened during the session |
| Timestamps are very large numbers | This is normal — timestamps are in **milliseconds** from program start, not wall-clock time | Divide by 1000 to get seconds, or by 60000 for minutes |
| Missing behavioral events | Events may have occurred during a pause or before the program started | Check the metadata file for pause/resume times |
| Frame timestamps file is empty | Imaging trigger not armed or microscope not sending frames | Verify microscope is armed and frame signal wiring is correct ([Section 8.6](#86-imaging-trigger--frame-timestamps)) |
| Permission denied on export | Write permission issue on save folder | Choose a writable directory (e.g., Desktop, Documents, or `~/REACHER/DATA/`) |

</details>

### 9.4 Finding Raw Logs

If auto-export fails or you need to recover data, raw logs are always stored at:

```
~/REACHER/LOG/<YYYY-MM-DD_HH-MM-SS>/
```

**When to use raw logs:**
- Auto-export failed or produced incomplete data
- You need to investigate a specific event or error
- You need to recover data from a crashed session

The `controller_log.json` file contains every message the Arduino sent, in the order
it was received. Each line is a JSON object you can parse with any JSON-compatible tool.

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="platform-issues"></a>
<details>
<summary><h2>10. Platform-Specific Issues</h2></summary>

<details>
<summary><h3>10.1 Windows</h3></summary>

**COM Port Numbering:**
- Windows assigns COM port numbers (COM3, COM4, etc.) to USB devices
- The number may change if you plug the Arduino into a different USB port
- Check Device Manager → Ports (COM & LPT) to see which port your Arduino is using

**Administrator Privileges:**
- The installer requires admin rights — click "Yes" at the UAC prompt
- Running the application itself does not require admin

**Windows Defender / Antivirus:**
- Windows Defender may flag the application as unrecognized
- Click "More info" → "Run anyway" to proceed
- You can add an exception in Windows Defender settings:
  Settings → Update & Security → Windows Security → Virus & threat protection →
  Manage settings → Exclusions → Add exclusion → Folder →
  `C:\Program Files\REACHER\`

</details>

<details>
<summary><h3>10.2 macOS</h3></summary>

**Gatekeeper ("App from unidentified developer"):**
- First time only: Right-click (Control-click) REACHER.app → select **Open**
- Click **Open** in the confirmation dialog
- Alternative: System Preferences → Security & Privacy → General → click
  **Open Anyway** next to the blocked app message

**Port Naming:**
- macOS uses `/dev/cu.usbserial-*` or `/dev/cu.usbmodem*` for serial ports
- If multiple ports appear, try each one until you find the Arduino

**macOS Sonoma+ USB Permissions:**
- Recent macOS versions may prompt for USB accessory permissions
- Click **Allow** when prompted to let REACHER communicate with the Arduino

</details>

<details>
<summary><h3>10.3 Linux</h3></summary>

**Serial Port Permissions:**
The most common Linux issue — your user account needs permission to access serial ports:

```bash
sudo usermod -a -G dialout $USER
```

Then **log out and log back in** (or restart the computer). This only needs to be done once.

**Port Naming:**
- Arduino UNO typically appears as `/dev/ttyACM0`
- Arduino clones with CH340 chips typically appear as `/dev/ttyUSB0`
- If unsure, run `ls /dev/tty*` before and after plugging in the Arduino to see
  which port appears

</details>

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="flowcharts"></a>
<details>
<summary><h2>11. Quick-Reference Troubleshooting Flowcharts</h2></summary>

<details>
<summary><h3>Flowchart 1: "I can't connect to the Arduino"</h3></summary>

```
I can't connect to the Arduino
|
+-- Are any ports listed in the dropdown?
|   |
|   +-- NO
|   |   |
|   |   +-- Is the Arduino plugged in via USB?
|   |       |
|   |       +-- NO --> Plug in the USB-A to USB-B cable
|   |       |         Click "Search Microcontrollers" again
|   |       |
|   |       +-- YES --> Is the USB cable working?
|   |           |
|   |           +-- Try a different cable
|   |           |
|   |           +-- Still no ports?
|   |               |
|   |               +-- You likely need a USB-to-serial driver
|   |                   See "Driver Installation" in Section 4.2
|   |                   Common chips: CH340, CP2102, FTDI
|   |
|   +-- YES
|       |
|       +-- Did you select the correct port?
|       |   |
|       |   +-- NOT SURE --> Try each port in the dropdown
|       |   |
|       |   +-- YES --> Is the port locked by another session?
|       |       |
|       |       +-- YES --> Disconnect the other session first
|       |       |           (port locking: one session per port)
|       |       |
|       |       +-- NO --> Is the Arduino IDE Serial Monitor open?
|       |           |
|       |           +-- YES --> Close it (only one program can use a port at a time)
|       |           |
|       |           +-- NO --> Is the firmware uploaded?
|       |               |
|       |               +-- NO --> Upload firmware first (Section 3.2)
|       |               |
|       |               +-- YES --> Try unplugging and replugging the Arduino
|       |                           Restart REACHER and try again
|       |
|       +-- "Loaded firmware is incompatible"
|           |
|           +-- Upload the correct firmware file
|               Use fr.ino for Fixed-Ratio, pr.ino for Progressive-Ratio, etc.
|
+-- LINUX ONLY: "Permission denied" on /dev/ttyUSB0 or /dev/ttyACM0
    |
    +-- Run: sudo usermod -a -G dialout $USER
        Log out and log back in, then try again
```

</details>

<details>
<summary><h3>Flowchart 2: "My experiment isn't working correctly"</h3></summary>

```
My experiment isn't working correctly
|
+-- No data appears on the graph
|   |
|   +-- Has the animal pressed the lever?
|   |   |
|   |   +-- NO --> Wait for a behavioral event
|   |   |
|   |   +-- YES --> Is the lever armed?
|   |       |
|   |       +-- NO --> Arm the lever in the Hardware Tab
|   |       |
|   |       +-- YES --> Is the correct lever set as "active"?
|   |           |
|   |           +-- NO --> Change the active lever in the Hardware Tab
|   |           |
|   |           +-- YES --> Try refreshing the browser
|   |                       Graph updates every 5 seconds
|   |
+-- Presses show as wrong type
|   |
|   +-- All presses show INACTIVE
|   |   |
|   |   +-- Wrong lever is set as "active"
|   |       Check Hardware Tab and select the correct active lever
|   |
|   +-- Presses show TIMEOUT when they shouldn't
|   |   |
|   |   +-- The timeout period hasn't expired yet
|   |       Default timeout is 20 seconds after each reward
|   |       Wait for timeout to expire, or reduce it in Schedule Tab
|   |
|   +-- No ACTIVE presses even though ratio should be met
|       |
|       +-- Check the ratio setting in Schedule Tab
|           FR1 = every press is rewarded
|           FR5 = every 5th press is rewarded
|
+-- Reward cycle isn't working (no tone/pump/laser after active press)
|   |
|   +-- Is the correct firmware uploaded for your paradigm?
|   |   +-- FR requires fr.ino, PR requires pr.ino, etc.
|   |
|   +-- Is the cue speaker armed?
|   |   +-- Check Hardware Tab
|   |
|   +-- Is the pump armed?
|   |   +-- Check Hardware Tab
|   |
|   +-- Is the device physically connected?
|       +-- See Section 8 for individual device testing
|
+-- Timing seems wrong
    |
    +-- Check Schedule Tab settings:
        - Timeout Duration (default: 20 seconds)
        - Trace Duration (default: 0 seconds)
        - Ratio value (default: 1)
        - Verify paradigm-specific parameters match your firmware
```

</details>

<details>
<summary><h3>Flowchart 3: "I can't export or find my data"</h3></summary>

```
I can't export or find my data
|
+-- Auto-export didn't run on stop
|   |
|   +-- Was the session stopped cleanly (Stop button)?
|   |   |
|   |   +-- NO --> Check ~/REACHER/LOG/ for raw logs
|   |   |         Data is still recoverable from controller_log.json
|   |   |
|   |   +-- YES --> Check ~/REACHER/LOG/ for the timestamped folder
|   |               Auto-export creates behavior_events.csv and
|   |               frame_timestamps.csv automatically
|   |
|   +-- Did the backend crash during export?
|       --> Restart REACHER, raw logs are preserved
|
+-- Export ZIP button doesn't work
|   |
|   +-- Was any data collected?
|   |   |
|   |   +-- NO --> Run an experiment first
|   |   |
|   |   +-- YES --> Do you have write permission to the save folder?
|   |       |
|   |       +-- Try saving to ~/REACHER/DATA/ or your Desktop
|   |
+-- Can't find the exported files
|   |
|   +-- Check the save location in the Program Tab
|   |   Default: ~/REACHER/DATA/
|   |
|   +-- Search your computer for .csv or .zip files created today
|   |
|   +-- Check ~/REACHER/LOG/ for timestamped folders
|
+-- Need to recover data from a crash
|   |
|   +-- Raw logs are stored at:
|       ~/REACHER/LOG/<YYYY-MM-DD_HH-MM-SS>/
|       |
|       +-- controller_log.json  --> Raw Arduino messages (JSON)
|       +-- interface_log.log    --> Application debug log
|       +-- event_log.jsonl      --> Structured event log
|
+-- Data format questions
    |
    +-- Timestamps are very large numbers?
    |   --> They are in milliseconds from session start
    |       Divide by 1000 for seconds
    |       Divide by 60000 for minutes
    |
    +-- What do the CSV columns contain?
    |   --> See Section 9.1 for column descriptions
    |
    +-- What's in the ZIP archive?
    |   --> See Section 9.2 for ZIP contents
    |
    +-- Frame timestamps file is empty?
        --> Microscope was not armed, or no frame signals were received
            See Section 8.6
```

</details>

<details>
<summary><h3>Flowchart 4: "The CLI isn't working"</h3></summary>

```
The CLI isn't working
|
+-- "reacher-cli: command not found"
|   |
|   +-- Is the CLI installed?
|       |
|       +-- NO --> Install it:
|       |         cd labrynth
|       |         pip install -e ".[cli]"
|       |
|       +-- YES --> Is the virtual environment activated?
|           |
|           +-- NO --> Activate it:
|           |         source venv/bin/activate  (Linux/macOS)
|           |         .\venv\Scripts\Activate.ps1  (Windows)
|           |
|           +-- YES --> Check that the install path is in your PATH
|
+-- CLI starts but can't connect to backend
|   |
|   +-- Is the backend running?
|       |
|       +-- The CLI should auto-start the backend
|       |   If auto-start fails:
|       |
|       +-- Start manually: python -m reacher
|       |
|       +-- Check that port 6229 is not blocked or in use
|
+-- Display is garbled or unresponsive
|   |
|   +-- Your terminal may not support prompt_toolkit
|       |
|       +-- Try a different terminal:
|           Windows: Windows Terminal (recommended)
|           macOS: iTerm2 or Terminal.app
|           Linux: GNOME Terminal, Konsole, or xterm
|
+-- Serial ports not listed in CLI
|   |
|   +-- Same causes as browser dashboard
|       See Flowchart 1 above
|       |
|       +-- Linux: check dialout group membership
|       +-- All platforms: check USB cable and drivers
|
+-- CLI freezes during firmware upload
    |
    +-- Upload may take 15-30 seconds — wait for completion
    +-- If truly stuck, press Ctrl+C and try again
    +-- Ensure no other program is using the serial port
```

</details>

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="glossary"></a>
<details>
<summary><h2>12. Glossary</h2></summary>

| Term | Definition |
|------|-----------|
| **Active press** | A lever press on the "active" lever that counts toward earning a reward. If the Fixed Ratio is 1 (FR1), every active press earns a reward. If FR5, every 5th active press earns a reward. |
| **Arming / Disarming** | Turning a hardware component on (armed) or off (disarmed) in the software. A disarmed device is ignored during the experiment. |
| **Auto-export** | The automatic saving of session data (CSV files and logs) that occurs when an experiment is stopped. Data is written to `~/REACHER/LOG/{TIMESTAMP}/`. |
| **Baud rate** | The speed of communication between the Arduino and your computer. REACHER uses 115200 baud. You do not need to set this — it is configured automatically. |
| **CLI** | Command-Line Interface. The terminal-based alternative to the browser dashboard for controlling experiments. Uses `prompt_toolkit` for interactive menus. Installed as the `reacher-cli` command. |
| **COM port** | The name your computer gives to the USB connection with the Arduino. On Windows it looks like "COM3", on macOS like "/dev/cu.usbserial-1420", and on Linux like "/dev/ttyUSB0". |
| **Cue / Conditioned Stimulus** | A sound (tone) played through the speaker to signal that a reward is coming. The animal learns to associate this sound with the reward. |
| **Debounce** | A short delay (20 ms) built into the firmware that prevents a single lever press or lick from being counted multiple times due to mechanical vibration of the switch. |
| **Demo mode** | A mode in the dashboard that uses simulated data to demonstrate the interface without requiring connected hardware. Useful for training and familiarization. |
| **Fixed Ratio (FR)** | A schedule where the animal must press the lever a fixed number of times to earn a reward. FR1 = 1 press per reward, FR5 = 5 presses per reward. |
| **Inactive press** | A lever press on the "inactive" lever (the one NOT set as the active lever). These presses are recorded but never earn rewards, regardless of timing. |
| **Infusion** | A single delivery of fluid through the pump. The pump runs for a set duration (default: 2000 ms / 2 seconds) to push fluid through the tubing. |
| **ISR (Interrupt Service Routine)** | A special function in the Arduino firmware that runs immediately when a specific event happens (like a frame signal arriving on pin 2). Used for precise timing of imaging frame timestamps. |
| **Omission** | A schedule where the animal must withhold responding for a configured duration to earn a reward. If the animal presses during the omission window, the timer resets. |
| **Paradigm** | The set of rules governing how the experiment runs. Different paradigms (FR, PR, VI, Omission, Pavlovian) define different relationships between the animal's behavior and the outcomes. |
| **Pavlovian** | A schedule where rewards are delivered based on trial structure (CS+/CS- trials) rather than the animal's lever pressing. Uses a `PavlovianScheduler` with configurable inter-trial intervals. |
| **Port locking** | The mechanism that prevents two REACHER sessions from using the same serial port simultaneously. Each Arduino port can only be claimed by one active session. |
| **Progressive Ratio (PR)** | A schedule where the number of presses required for each successive reward increases. The session ends when the animal fails to complete the current ratio within the breakpoint time. |
| **Scheduler engine** | The firmware component (`Scheduler` class) that implements the trigger → chain → action pattern for all operant paradigms (FR, PR, VI, Omission). Pavlovian uses a separate `PavlovianScheduler`. |
| **Secondary cue** | A second tone speaker connected to pin 7, used in Pavlovian (CS- tones) and other advanced paradigm configurations. |
| **Secondary pump** | A second infusion pump connected to pin 8, used in paradigms that require dual-reward delivery. |
| **Session ID** | A unique identifier assigned to each REACHER session. Used internally by the backend to track and route commands to the correct session. |
| **Timeout period** | A cooldown period after a reward during which lever presses are counted but do not earn rewards. These presses are classified as "timeout presses." Default: 20 seconds. |
| **Timeout press** | A lever press on the active lever that occurs during the timeout period (after a reward). The press is recorded but does not count toward the next reward. |
| **Trace interval** | A gap in time between two events — for example, between the cue tone ending and the pump starting. A trace interval of 0 means the pump starts immediately after the cue would finish. |
| **Variable Interval (VI)** | A schedule where rewards become available after a variable amount of time. The animal must press during the availability window to earn the reward. |
| **Watchdog timer** | A hardware safety timer (8-second timeout) in the Arduino firmware. If the main loop stalls for longer than 8 seconds, the watchdog automatically resets the Arduino to prevent hardware damage. |

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

*This guide covers REACHER-Suite v2 (develop) (all paradigms: FR, PR, VI, Omission, Pavlovian — wired sessions).*
*For issues not covered here, check the [GitHub repository](https://github.com/Otis-Lab-MUSC/labrynth) or file an issue.*

*Last updated: April 2026.*
