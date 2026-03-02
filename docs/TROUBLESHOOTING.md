# REACHER System Troubleshooting Guide

> 📝 **Note:**
> **REACHER** — Rodent Experiment Application Controls and Handling Ecosystem for Research
>
> This guide covers the entire pipeline from physical hardware setup through data export
> for the Fixed-Ratio (FR) paradigm using wired connections. It is written for users with
> no prior programming or lab hardware experience.

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
| 7 | [Hardware Testing & Diagnostics](#hardware-testing) | Per-device testing procedures |
| 8 | [Data Export & Validation](#data-export) | Export, verification, raw logs |
| 9 | [Platform-Specific Issues](#platform-issues) | Windows, macOS, Linux |
| 10 | [Troubleshooting Flowcharts](#flowcharts) | Quick-reference decision trees |
| 11 | [Glossary](#glossary) | Terms and definitions |

> 💡 **Tip:**
> Click any section heading to expand it.

<!-- ============================================================ -->

<a id="system-overview"></a>
<details open>
<summary><h2>1. System Overview</h2></summary>

The REACHER system has three main components that work together:

```
+-------------------+       USB Cable         +-------------------+      Browser       +-------------------+
|                   |  (USB-A to USB-B)       |                   |  (localhost:7007)  |                   |
|  Arduino UNO      | <-------------------->  |  Your Computer    | <----------------> |  Dashboard        |
|  (Firmware)       |    Serial @ 115200      |  (The Labrynth)   |    Panel Server    |  (Browser Tab)    |
|                   |       baud              |                   |                    |                   |
+-------------------+                         +-------------------+                    +-------------------+
  Controls hardware                            Runs the launcher                        Where you interact
  (levers, pump,                               and Panel server                         with the experiment
   speaker, laser)
```

### What Each Piece Does

| Component | What It Is | What It Does |
|-----------|-----------|--------------|
| **Arduino UNO** | A small circuit board | Controls all physical hardware (levers, pump, speaker, laser, lick circuit, imaging) |
| **Firmware** | Code uploaded to the Arduino | Listens for commands, monitors hardware, sends data back over USB |
| **The Labrynth** | Application you install on your computer | Launches the dashboard server and manages the connection to the Arduino |
| **Dashboard** | Web page in your browser | Where you connect, configure, run, and monitor experiments |
| **REACHER library** | Software running behind the dashboard | Handles serial communication, data collection, and export |

<details>
<summary><strong>Pin Assignment Reference</strong></summary>

The Arduino UNO connects to each hardware component through specific pins:

| Pin | Device | Direction | Description |
|-----|--------|-----------|-------------|
| 2 | Microscope Timestamp | INPUT (Interrupt) | Receives frame sync signals from imaging system |
| 3 | Cue Speaker | OUTPUT | Plays tones through a connected speaker |
| 4 | Pump | OUTPUT | Activates the infusion pump |
| 5 | Lick Circuit | INPUT_PULLUP | Detects lick contact on the spout |
| 6 | Laser | OUTPUT | Controls optogenetic laser stimulation |
| 9 | Microscope Trigger | OUTPUT | Sends trigger pulses to imaging system |
| 10 | Right-Hand (RH) Lever | INPUT_PULLUP | Detects right lever presses |
| 13 | Left-Hand (LH) Lever | INPUT_PULLUP | Detects left lever presses |

</details>

<details>
<summary><strong>Wiring Reference Diagram</strong></summary>

```
                            Arduino UNO
                    +-------------------------+
                    |                     [13] |-----> Left-Hand Lever
                    |                     [12] |
                    |                     [11] |
                    |                     [10] |-----> Right-Hand Lever
                    |                      [9] |-----> Microscope Trigger (OUTPUT)
                    |                      [8] |
                    |                      [7] |
                    |                      [6] |-----> Laser
                    |                      [5] |-----> Lick Circuit
                    |                      [4] |-----> Pump
                    |                      [3] |-----> Cue Speaker
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

> 📝 **Note:**
> Each hardware component also needs a ground (GND) connection. Use the
> GND pins on the Arduino. Some components (like the laser) may also require an
> external power supply.

### Serial Communication

- **Cable:** USB-A to USB-B (the same type used by many printers)
- **Baud rate:** 115200 (this is set automatically — you do not need to configure it)
- **Protocol:** JSON messages sent over the USB serial connection

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="installation"></a>
<details>
<summary><h2>2. Installation & First Launch</h2></summary>

### 2.1 Download & Install

Download the latest installer for your platform from the
[Labrynth releases page](https://github.com/Otis-Lab-MUSC/Labrynth/releases).

<details>
<summary><h3>Windows</h3></summary>

1. Download `labrynth-1.0-x64.exe`
2. Double-click the installer
3. **You will need administrator privileges** — click "Yes" when prompted by Windows
4. Follow the Inno Setup wizard (accept defaults)
5. The application installs to `C:\Program Files\The Labrynth\`
6. A desktop shortcut is created (optional)
7. The application launches automatically after installation

</details>

<details>
<summary><h3>macOS</h3></summary>

1. Download `labrynth_x64.dmg`
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

1. Download `labrynth_amd64.deb`
2. Open a terminal and run:
   ```bash
   sudo apt install ./labrynth_amd64.deb
   ```
3. The application installs to `/opt/labrynth/` with a symlink at `/usr/local/bin/labrynth`
4. If you get errors about missing Qt libraries, see [Section 9.3](#93-linux)

</details>

### 2.2 First Launch

When you open The Labrynth:

1. A small **launcher window** appears (300 x 150 pixels) with the text
   "Opening session in browser..."
2. Your default web browser opens automatically to `http://localhost:7007`
3. The launcher window updates to "Session running in browser. (keep this window open)"
4. The dashboard loads in your browser with a dark theme

> ❗ **Important:**
> Keep the launcher window open for the entire session. Closing it
> shuts down the dashboard server.

<details>
<summary><strong>First Launch Troubleshooting</strong></summary>

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Launcher window doesn't appear | Missing Qt/PySide6 libraries (Linux) | Reinstall the application; see [Section 9.3](#93-linux) for missing libraries |
| Browser doesn't open automatically | No default browser set, or port 7007 already in use | Manually open your browser and go to `http://localhost:7007`; if port is in use, close the other application using it |
| "Server Not Running" dialog appears | The dashboard server crashed on startup | Close and restart The Labrynth; check for other Panel servers running on port 7007 |
| Blank white page in browser | Slow initial load | Wait 10–15 seconds, then refresh the browser page |
| "Quit the Labrynth" dialog when closing | Normal — confirms you want to shut down | Click "Yes" to quit, or "No" to keep running |

</details>

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="arduino-firmware"></a>
<details>
<summary><h2>3. Arduino & Firmware Setup</h2></summary>

### 3.1 Required Hardware

Before connecting to the software, make sure you have:

- [ ] **Arduino UNO** board
- [ ] **USB-A to USB-B cable** (connects the Arduino to your computer)
- [ ] **Lever switch(es)** — wired to pin 10 (right) and/or pin 13 (left)
- [ ] **Infusion pump** — wired to pin 4
- [ ] **Cue speaker** (8Ω impedance typical) — wired to pin 3
- [ ] **Laser** (if using optogenetics) — wired to pin 6, with external power supply
- [ ] **Lick circuit** (if measuring licks) — wired to pin 5
- [ ] **Imaging trigger/timestamp** (if using microscopy) — trigger on pin 9, timestamp on pin 2

You will also need the **Arduino IDE** to upload firmware:
- Download from [https://www.arduino.cc/en/software](https://www.arduino.cc/en/software)

### 3.2 Uploading Firmware

Follow these steps to upload the Fixed-Ratio firmware to your Arduino:

1. **Install the Arduino IDE** (see link above)
2. **Install the ArduinoJson library:**
   - Open Arduino IDE
   - Go to **Sketch → Include Library → Manage Libraries...**
   - Search for **"ArduinoJson"**
   - Click **Install**
3. **Open the firmware file:**
   - Navigate to `reacher-firmware/operant_FR/`
   - Open `operant_FR.ino` in the Arduino IDE
4. **Select your board:**
   - Go to **Tools → Board → Arduino AVR Boards → Arduino UNO**
5. **Select your port:**
   - Go to **Tools → Port** and select the port showing your Arduino
   - Windows: `COM3`, `COM4`, etc.
   - macOS: `/dev/cu.usbserial-*` or `/dev/cu.usbmodem*`
   - Linux: `/dev/ttyUSB0` or `/dev/ttyACM0`
6. **Upload:**
   - Click the **Upload** button (right arrow icon) or press <kbd>Ctrl</kbd>+<kbd>U</kbd> / <kbd>Cmd</kbd>+<kbd>U</kbd>
   - Wait for "Done uploading" message

<details>
<summary><strong>Which Firmware for Which Paradigm</strong></summary>

| Paradigm | Firmware Folder | Status |
|----------|----------------|--------|
| Fixed-Ratio (FR) | `operant_FR/` | Stable — covered in this guide |
| Progressive-Ratio (PR) | `operant_PR/` | Beta |
| Variable-Interval (VI) | `operant_VI/` | Beta |
| Omission | `operant_OM/` | Beta |

</details>

<details>
<summary><strong>Firmware Upload Troubleshooting</strong></summary>

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| "Board not found" in Arduino IDE | Wrong board selected or USB not connected | Go to Tools → Board → select **Arduino UNO**; check USB cable is plugged in |
| Upload fails with "avrdude" error | Wrong COM port selected or Arduino not connected | Go to Tools → Port → select the correct port; try unplugging and replugging USB |
| `ArduinoJson.h: No such file or directory` | ArduinoJson library not installed | Install it: Sketch → Include Library → Manage Libraries → search "ArduinoJson" → Install |
| Upload succeeds but nothing happens | This is normal | The firmware waits for a connection from the dashboard before doing anything |

</details>

### 3.3 Wiring Verification

After uploading firmware, verify each component is wired correctly:

<details>
<summary><strong>Wiring Verification Table</strong></summary>

| Pin | Device | How to Test |
|-----|--------|-------------|
| 10 | Right Lever | Press lever → check for response in dashboard (Section 7.1) |
| 13 | Left Lever | Press lever → check for response in dashboard (Section 7.1) |
| 3 | Cue Speaker | Connect to Arduino → should hear jingle when dashboard connects (Section 7.2) |
| 4 | Pump | Use test button in Hardware Tab (Section 7.3) |
| 5 | Lick Circuit | Touch spout → check for lick events in dashboard (Section 7.5) |
| 6 | Laser | Use test button in Hardware Tab (Section 7.4) |
| 9 | Microscope Trigger | Verify trigger output with oscilloscope or imaging software (Section 7.6) |
| 2 | Microscope Timestamp | Verify frame signals are received (Section 7.6) |

</details>

> ❗ **Important:**
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
4. A new tab appears labeled `LOCAL - <your name>`

<details>
<summary><strong>Troubleshooting — Session Creation</strong></summary>

| Problem | Error Message | Fix |
|---------|--------------|-----|
| Left the name field empty | "Please enter a name and try again." | Type a name before clicking the button |
| Used a name that's already in use | "Name entered already exists. Please enter a different name." | Choose a different name |

</details>

### 4.2 Finding the COM Port

1. Make sure your Arduino is connected via USB
2. In the **Home Tab**, click **"Search Microcontrollers"**
3. A dropdown list of available ports appears
4. Select your Arduino's port from the dropdown
5. Click **Connect**

**Port names by platform:**

| Platform | Typical Port Name |
|----------|------------------|
| Windows | `COM3`, `COM4`, `COM5`, etc. |
| Linux | `/dev/ttyUSB0` or `/dev/ttyACM0` |
| macOS | `/dev/cu.usbserial-*` or `/dev/cu.usbmodem*` |

<details>
<summary><strong>Connection Troubleshooting</strong></summary>

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| No ports appear in dropdown | Arduino not plugged in | Plug in the USB cable, then click "Search Microcontrollers" again |
| No ports appear (Arduino IS plugged in) | Missing USB-to-serial driver | Install the driver for your Arduino's USB chip (see below) |
| Port appears but connection fails | Port is in use by another program | Close the Arduino IDE Serial Monitor or any other serial program |
| "Loaded firmware is incompatible. Please upload a qualified file." | Wrong firmware uploaded to Arduino | Upload the correct firmware (Section 3.2) |
| Connection succeeds then immediately drops | Faulty USB cable or loose connection | Try a different USB cable; ensure both ends are firmly plugged in |

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
2. **Firmware information appears** in the Home Tab, showing:
   - Sketch name: `operant_FR-beta.ino`
   - Version: `v1.1.1`
   - Baud rate: `115200`
   - Schedule: `FIXED_RATIO`

**If no jingle plays:**
- Check that the cue speaker is wired to **pin 3** and a **GND** pin
- The jingle only plays if the speaker is physically connected — it does not require
  arming the cue in the Hardware Tab
- See [Section 7.2](#72-cue-speaker-testing) for speaker diagnostics

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
- **Locked (🔒):** Device is disarmed (off)
- **Unlocked (🔓):** Device is armed (on)

**Arming each component:**

| Device | What to Do | What It Enables |
|--------|-----------|-----------------|
| RH Lever | Toggle the Right-Hand Lever arm button | Detects presses on the right lever |
| LH Lever | Toggle the Left-Hand Lever arm button | Detects presses on the left lever |
| Active Lever | Select which lever (LH or RH) is the "active" (reinforced) lever | Presses on the active lever count toward rewards |
| Cue | Toggle the Cue arm button | Plays a tone when the animal earns a reward |
| Pump | Toggle the Pump arm button | Delivers infusion when the animal earns a reward |
| Laser | Toggle the Laser arm button | Activates laser stimulation (if using optogenetics) |
| Lick Circuit | Toggle the Lick Circuit arm button | Records lick events |
| Microscope | Toggle the Microscope arm button | Records imaging frame timestamps |

<details>
<summary><strong>Hardware Troubleshooting</strong></summary>

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Device won't arm (no response when clicking) | Arduino not connected | Go to the Home Tab and verify connection status |
| Pump test runs but no fluid delivers | Tubing disconnected or pump wired incorrectly | Check tubing connections; verify pump is wired to **pin 4** |
| Cue test plays no sound | Speaker not connected or wired wrong | Check speaker is wired to **pin 3** and GND; test speaker independently |
| Laser test does nothing | Laser not connected or not powered | Check wiring to **pin 6**; verify external power supply for the laser |
| Lever shows no response to press | Wrong pin or wiring issue | Right lever → **pin 10**, Left lever → **pin 13**; check switch continuity with a multimeter |

</details>

### 5.2 Schedule Tab — Timing Parameters

These parameters control the timing and rules of your experiment. All times are
entered in **seconds** in the dashboard and converted to milliseconds for the Arduino.

| Parameter | What It Means | Default | Range |
|-----------|--------------|---------|-------|
| **Fixed Ratio** | How many lever presses before a reward is given | 1 (FR1 = every press is rewarded) | 1–50 |
| **Timeout Duration** | Period following reward delivery where rewards do not occur - active presses during this period do not trigger cue or reward | 20 seconds | 0–600 seconds |
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

> ⚠️ **Warning:**
> **Timeout set to 0:** The animal can earn rewards as fast as it can press — this is
> usually not desired.

> ⚠️ **Warning:**
> **Trace interval too long:** The animal may not associate the cue with the reward if
> the gap is too large.

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

- **Filename:** Enter a name for your data file (e.g., `experiment1`)
- **Save folder:** Choose where to save the data
- **Default save location:** `~/REACHER/DATA/`

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
| Graph stopped updating mid-session | Serial connection lost | Check the USB cable; the Arduino may have reset (see [Section 6.5](#65-unexpected-stops--recovery)) |
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
3. **Always stop the experiment before disconnecting the USB cable**

> 🔴 **Caution:**
> **If USB is pulled without stopping:** Data collected up to that point is
> preserved in the log files (`~/REACHER/LOG/`), but the session metadata may be
> incomplete and the export file may not be generated automatically.

<a id="65-unexpected-stops--recovery"></a>

### 6.5 Unexpected Stops & Recovery

If your experiment stops unexpectedly, follow this decision tree:

```
Experiment stopped unexpectedly
|
+-- Did the application crash?
|   |
|   +-- YES --> Restart The Labrynth
|   |           Your data is saved in ~/REACHER/LOG/
|   |           Look for the most recent timestamped folder
|   |
|   +-- NO
|       |
|       +-- Did the Arduino reset? (power LED blinks off then on)
|       |   |
|       |   +-- YES --> USB cable issue
|       |   |           Try a different cable
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

<a id="hardware-testing"></a>
<details>
<summary><h2>7. Hardware Testing & Diagnostics</h2></summary>

Use these procedures to test each hardware component independently. This helps
isolate whether a problem is with the wiring, the component, or the software.

<details>
<summary><h3>7.1 Lever Switch Testing</h3></summary>

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
Pin: 10 (RH) or 13 (LH)
Orientation: RH or LH
```

**In the dashboard data, events appear as:**
- `RH_LEVER` / `LH_LEVER` (device)
- `ACTIVE_PRESS` / `TIMEOUT_PRESS` / `INACTIVE_PRESS` (event type)

**Debounce note:** The firmware uses a 20ms debounce delay. This means very rapid
presses (faster than 20ms apart) may be missed. This is normal and prevents
false readings from switch bounce.

</details>

<a id="72-cue-speaker-testing"></a>
<details>
<summary><h3>7.2 Cue Speaker Testing</h3></summary>

**Physical test:**
1. Briefly touch the speaker wires to the Arduino's 3.3V and GND pins
2. You should hear a faint click or pop — this confirms the speaker works

**Software test:**
1. Connect to the Arduino — the **connection jingle** should play automatically
   (three ascending tones: 500 Hz → 1000 Hz → 1500 Hz, each 100ms long)
2. Arm the cue in the Hardware Tab
3. Trigger a reward (press the active lever enough times to meet the ratio)
4. Listen for the cue tone

**Default tone settings:**
- Frequency: 8000 Hz
- Duration: 1600 ms (1.6 seconds)
- Trace interval: 0 ms

**No sound troubleshooting:**
1. Verify speaker is wired to **pin 3** and **GND**
2. Check speaker impedance — 8Ω is typical
3. Try a different speaker
4. If the connection jingle doesn't play, the issue is wiring or the speaker itself

</details>

<details>
<summary><h3>7.3 Pump Testing</h3></summary>

**Software test:**
1. Connect to the Arduino
2. Arm the pump in the Hardware Tab
3. Use the test button in the Hardware Tab to run the pump

**What to verify:**
- Pump motor runs when activated
- Tubing is connected and fluid flows
- Default infusion duration: 2000 ms (2 seconds)

**No delivery troubleshooting:**
1. Check pump is wired to **pin 4** and **GND**
2. Verify tubing connections at both ends
3. Check pump power supply (some pumps need external power)
4. Verify pump runs during the test — if the motor spins but no fluid, the tubing
   is the issue

</details>

<details>
<summary><h3>7.4 Laser Testing</h3></summary>

> 🔴 **Caution:**
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
<summary><h3>7.5 Lick Circuit Testing</h3></summary>

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

<a id="76-imaging-trigger"></a>
<details>
<summary><h3>7.6 Imaging Trigger / Frame Timestamps</h3></summary>

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
<summary><h2>8. Data Export & Validation</h2></summary>

### 8.1 Exporting Data

1. Go to the **Monitor Tab**
2. Click the **Export** button
3. An Excel file (`.xlsx`) is saved to the location you specified in the Program Tab
   (default: `~/REACHER/DATA/`)

**Excel file sheets:**

| Sheet Name | What It Contains |
|------------|-----------------|
| **Session Summary** | Start time, end time, behavior chamber name, frame count, event counts |
| **Behavior Data** | Columns: device, event, start_timestamp, end_timestamp |
| **Firmware Information** | Sketch name, version, baud rate, schedule type |
| **Hardware Settings** | All hardware configuration details |
| **Frame Timestamps** | List of imaging frame timestamps (if microscope was armed) |

### 8.2 Verifying Exported Data

After exporting, open the file in Excel, LibreOffice Calc, or similar:

1. **Check row counts:** The number of rows in "Behavior Data" should match the
   number of events you observed during the session
2. **Verify timestamps are sequential:** Each start_timestamp should be greater than
   the previous one
3. **Cross-reference infusion count:** The number of "INFUSION" events should match
   what you expected
4. **Check the Session Summary:** Event counts are broken down by type (e.g.,
   "PUMP INFUSION", "RH_LEVER ACTIVE_PRESS")

<details>
<summary><strong>Data Troubleshooting</strong></summary>

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Export button does nothing | No data has been collected | Run an experiment first — there must be at least one event |
| Export fails with an error | `openpyxl` not installed or permission denied on save folder | Install openpyxl: `pip install openpyxl`; choose a writable directory |
| Excel file opens but sheets are empty | Experiment stopped before any events occurred | Verify that events actually happened during the session |
| Timestamps are very large numbers | This is normal — timestamps are in **milliseconds** from program start, not wall-clock time | Divide by 1000 to get seconds, or by 60000 for minutes |
| Missing behavioral events | Events may have occurred during a pause or before the program started | Check pause/resume times in the Session Summary sheet |
| Frame Timestamps sheet is empty | Imaging trigger not armed or microscope not sending frames | Verify microscope is armed and frame signal wiring is correct (Section 7.6) |

</details>

### 8.3 Finding Raw Logs

If export fails or you need to recover data, raw logs are stored at:

```
~/REACHER/LOG/<YYYY-MM-DD_HH-MM-SS>/
```

Each session folder contains:

| File | What It Contains |
|------|-----------------|
| `controller_log.json` | Raw JSON messages received from the Arduino |
| `interface_log.log` | Python application debug log |

**When to use raw logs:**
- Export failed or produced incomplete data
- You need to investigate a specific event or error
- You need to recover data from a crashed session

The `controller_log.json` file contains every message the Arduino sent, in the order
it was received. Each line is a JSON object you can parse with any JSON-compatible tool.

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="platform-issues"></a>
<details>
<summary><h2>9. Platform-Specific Issues</h2></summary>

<details>
<summary><h3>9.1 Windows</h3></summary>

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
  `C:\Program Files\The Labrynth\`

</details>

<details>
<summary><h3>9.2 macOS</h3></summary>

**Gatekeeper ("App from unidentified developer"):**
- First time only: Right-click (Control-click) the app → select **Open**
- Click **Open** in the confirmation dialog
- Alternative: System Preferences → Security & Privacy → General → click
  **Open Anyway** next to the blocked app message

**Port Naming:**
- macOS uses `/dev/cu.usbserial-*` or `/dev/cu.usbmodem*` for serial ports
- If multiple ports appear, try each one until you find the Arduino

**macOS Sonoma+ USB Permissions:**
- Recent macOS versions may prompt for USB accessory permissions
- Click **Allow** when prompted to let The Labrynth communicate with the Arduino

</details>

<a id="93-linux"></a>
<details>
<summary><h3>9.3 Linux</h3></summary>

**Serial Port Permissions:**
The most common Linux issue — your user account needs permission to access serial ports:

```bash
sudo usermod -a -G dialout $USER
```

Then **log out and log back in** (or restart the computer). This only needs to be done once.

**Missing Qt Libraries:**
If the launcher window doesn't appear, you may need to install Qt dependencies:

```bash
sudo apt install libxkbcommon0 libxcb-xinerama0
```

On some distributions, additional packages may be needed:

```bash
sudo apt install libxcb-cursor0 libxcb-icccm4 libxcb-keysyms1
```

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
<summary><h2>10. Quick-Reference Troubleshooting Flowcharts</h2></summary>

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
|       |   +-- YES --> Is the Arduino IDE Serial Monitor open?
|       |       |
|       |       +-- YES --> Close it (only one program can use a port at a time)
|       |       |
|       |       +-- NO --> Is the firmware uploaded?
|       |           |
|       |           +-- NO --> Upload firmware first (Section 3.2)
|       |           |
|       |           +-- YES --> Try unplugging and replugging the Arduino
|       |                       Restart The Labrynth and try again
|       |
|       +-- "Loaded firmware is incompatible"
|           |
|           +-- Upload the correct firmware file
|               Use operant_FR.ino for Fixed-Ratio experiments
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
|       +-- Check the Fixed Ratio setting in Schedule Tab
|           FR1 = every press is rewarded
|           FR5 = every 5th press is rewarded
|
+-- Reward cycle isn't working (no tone/pump/laser after active press)
|   |
|   +-- Is the cue speaker armed?
|   |   +-- Check Hardware Tab
|   |
|   +-- Is the pump armed?
|   |   +-- Check Hardware Tab
|   |
|   +-- Is the device physically connected?
|       +-- See Section 7 for individual device testing
|
+-- Timing seems wrong
    |
    +-- Check Schedule Tab settings:
        - Timeout Duration (default: 20 seconds)
        - Trace Duration (default: 0 seconds)
        - Fixed Ratio (default: 1)
```

</details>

<details>
<summary><h3>Flowchart 3: "I can't export or find my data"</h3></summary>

```
I can't export or find my data
|
+-- Export button doesn't work
|   |
|   +-- Was any data collected?
|   |   |
|   |   +-- NO --> Run an experiment first
|   |   |
|   |   +-- YES --> Do you have write permission to the save folder?
|   |       |
|   |       +-- Try saving to ~/REACHER/DATA/ or your Desktop
|   |       |
|   |       +-- Is openpyxl installed?
|   |           pip install openpyxl
|   |
+-- Can't find the exported file
|   |
|   +-- Check the save location you set in the Program Tab
|   |   Default: ~/REACHER/DATA/
|   |
|   +-- Search your computer for .xlsx files created today
|   |
|   +-- Check ~/REACHER/DATA/ for timestamped folders
|
+-- Need to recover data from a crash
|   |
|   +-- Raw logs are stored at:
|       ~/REACHER/LOG/<YYYY-MM-DD_HH-MM-SS>/
|       |
|       +-- controller_log.json --> Raw Arduino messages (JSON)
|       +-- interface_log.log  --> Application debug log
|
+-- Data format questions
    |
    +-- Timestamps are very large numbers?
    |   --> They are in milliseconds from session start
    |       Divide by 1000 for seconds
    |       Divide by 60000 for minutes
    |
    +-- What do the Excel sheets contain?
    |   --> See Section 8.1 for sheet descriptions
    |
    +-- Frame Timestamps sheet is empty?
        --> Microscope was not armed, or no frame signals were received
            See Section 7.6
```

</details>

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="glossary"></a>
<details>
<summary><h2>11. Glossary</h2></summary>

| Term | Definition |
|------|-----------|
| **Active press** | A lever press on the "active" lever that counts toward earning a reward. If the Fixed Ratio is 1 (FR1), every active press earns a reward. If FR5, every 5th active press earns a reward. |
| **Arming / Disarming** | Turning a hardware component on (armed) or off (disarmed) in the software. A disarmed device is ignored during the experiment. |
| **Baud rate** | The speed of communication between the Arduino and your computer. REACHER uses 115200 baud. You do not need to set this — it is configured automatically. |
| **COM port** | The name your computer gives to the USB connection with the Arduino. On Windows it looks like "COM3", on macOS like "/dev/cu.usbserial-1420", and on Linux like "/dev/ttyUSB0". |
| **Cue / Conditioned Stimulus** | A sound (tone) played through the speaker to signal that a reward is coming. The animal learns to associate this sound with the reward. |
| **Debounce** | A short delay (20 ms) built into the firmware that prevents a single lever press or lick from being counted multiple times due to mechanical vibration of the switch. |
| **Fixed Ratio (FR)** | A schedule where the animal must press the lever a fixed number of times to earn a reward. FR1 = 1 press per reward, FR5 = 5 presses per reward. |
| **Progressive Ratio (PR)** | A schedule where the number of presses required for each successive reward increases. (Beta — not covered in this guide.) |
| **Variable Interval (VI)** | A schedule where rewards become available after a variable amount of time. (Beta — not covered in this guide.) |
| **Omission** | A schedule where the animal must withhold responding to earn a reward. (Beta — not covered in this guide.) |
| **Infusion** | A single delivery of fluid through the pump. The pump runs for a set duration (default: 2000 ms / 2 seconds) to push fluid through the tubing. |
| **ISR (Interrupt Service Routine)** | A special function in the Arduino firmware that runs immediately when a specific event happens (like a frame signal arriving on pin 2). Used for precise timing of imaging frame timestamps. |
| **Paradigm** | The set of rules governing how the experiment runs. Different paradigms (FR, PR, VI, Omission) define different relationships between the animal's behavior and the outcomes. |
| **Trace interval** | A gap in time between two events — for example, between the cue tone ending and the pump starting. A trace interval of 0 means the pump starts immediately after the cue would finish. |
| **Timeout period** | A cooldown period after a reward during which lever presses are counted but do not earn rewards. These presses are classified as "timeout presses." Default: 20 seconds. |
| **Inactive press** | A lever press on the "inactive" lever (the one NOT set as the active lever). These presses are recorded but never earn rewards, regardless of timing. |
| **Timeout press** | A lever press on the active lever that occurs during the timeout period (after a reward). The press is recorded but does not count toward the next reward. |

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

*This guide covers REACHER firmware v1.1.1 (Fixed-Ratio paradigm, wired sessions only).*
*For issues not covered here, check the [GitHub repository](https://github.com/Otis-Lab-MUSC/Labrynth) or file an issue.*
