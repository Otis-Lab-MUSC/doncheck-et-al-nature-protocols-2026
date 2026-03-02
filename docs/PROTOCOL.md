# REACHER System Protocol Guide

> 📝 **Note:**
> **REACHER** — Rodent Experiment Application Controls and Handling Ecosystem for Research
>
> This guide walks through **Procedure 2** of the Nature Protocols manuscript:
> *Integrating circuitry and software*. It covers every step from installing the
> Arduino IDE through running and exporting a self-administration session using
> the Labrynth application. It is written for users with no prior programming
> experience.

<a id="table-of-contents"></a>

### Table of Contents

| # | Section | Description |
|:-:|---------|-------------|
| 1 | [Prerequisites](#prerequisites) | System requirements, critical step note |
| 2 | [Installing Arduino IDE](#installing-arduino-ide) | Download and library setup |
| 3 | [Installing REACHER Application](#installing-reacher-application) | Download Labrynth |
| 4 | [Installing REACHER Arduino Firmware](#installing-firmware) | Flash operant_FR sketch |
| 5 | [Launching the Application](#launching-application) | Open Labrynth, launcher window |
| 6 | [Creating a New Session](#creating-session) | Name and create session tab |
| 7 | [Home Page — Connecting an Arduino](#home-page) | COM search and connect |
| 8 | [Program Page — Setting Program Parameters](#program-page) | Presets, limits, components, file config |
| 9 | [Hardware Page — Adjusting Hardware Settings](#hardware-page) | Arm and configure devices |
| 10 | [Schedule Page — Adjusting Within-Trial Dynamics](#schedule-page) | Timeout, trace, ratio sliders |
| 11 | [Monitor Page — Running the Session](#monitor-page) | Start, pause, stop, export |
| 12 | [Testing and Troubleshooting](#testing-troubleshooting) | Link to TROUBLESHOOTING.md |

> 💡 **Tip:**
> Click any section heading to expand it.

<!-- ============================================================ -->

<a id="prerequisites"></a>
<details open>
<summary><h2>1. Prerequisites</h2></summary>

> ❗ **Important:**
> Ensure your computer meets the following minimum requirements before
> proceeding. Running self-administration sessions involves continuous serial
> communication, real-time plotting, and data logging — an under-powered machine
> can drop events or freeze mid-session.

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| **CPU** | Quad-core (e.g., Intel i3) | 6–8 core (e.g., Intel i5/i7, AMD Ryzen 5) |
| **RAM** | 8 GB | 16 GB |
| **Storage** | 256 GB SSD | 512 GB SSD |
| **OS** | Windows 10 (64-bit), macOS, or Linux | Ubuntu/Debian Linux, Windows 11, macOS |

**Additional requirements:**
- An **Arduino UNO** (or compatible) connected via **USB-A to USB-B cable**
- A modern **web browser** (Chrome, Firefox, or Edge)
- The computer must remain **powered on and connected** to the Arduino for the entire session — do not let it sleep

> 📝 **Note:**
> If you plan to run multiple sessions simultaneously (one Arduino per session),
> each additional session increases CPU and memory usage. The recommended specs
> scale accordingly.

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="installing-arduino-ide"></a>
<details>
<summary><h2>2. Installing Arduino IDE</h2></summary>

### 2.1 Download and Install the Arduino IDE

1. Visit [https://www.arduino.cc/en/software](https://www.arduino.cc/en/software)
2. Download the installer for your operating system
3. Run the installer and follow the prompts

### 2.2 Install the ArduinoJson Library

The REACHER firmware uses the ArduinoJson library for serial communication.

1. Open the Arduino IDE
2. Go to **Sketch → Include Library → Manage Libraries...**
3. In the search bar, type **"ArduinoJson"**
4. Find **"ArduinoJson by Benoit Blanchon"** and click **Install**

> 📝 **Note:**
> The ArduinoJson library (version 6.x or later) is the only external library
> required. All other libraries used by the firmware (`Arduino.h`,
> `SoftwareSerial.h`) are built in to the Arduino IDE.

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="installing-reacher-application"></a>
<details>
<summary><h2>3. Installing REACHER Application</h2></summary>

This step corresponds to **Step 3** in the manuscript.

Download the latest Labrynth installer for your platform from the
[Labrynth releases page](https://github.com/Otis-Lab-MUSC/labrynth/releases).

| Platform | Installer | Notes |
|----------|-----------|-------|
| **Windows** | `labrynth-1.0-x64.exe` | Requires administrator privileges |
| **macOS** | `labrynth_x64.dmg` | Right-click → Open on first launch (Gatekeeper bypass) |
| **Linux** | `labrynth_amd64.deb` | Install with `sudo apt install ./labrynth_amd64.deb` |

<details>
<summary><strong>Windows Installation Details</strong></summary>

1. Download `labrynth-1.0-x64.exe`
2. Double-click the installer
3. Click "Yes" when prompted by Windows for administrator privileges
4. Follow the Inno Setup wizard (accept defaults)
5. The application installs to `C:\Program Files\The Labrynth\`
6. A desktop shortcut is created (optional)

</details>

<details>
<summary><strong>macOS Installation Details</strong></summary>

1. Download `labrynth_x64.dmg`
2. Double-click the `.dmg` file to mount it
3. Drag "Labrynth" to your Applications folder
4. **First launch — Gatekeeper bypass:**
   - Right-click (or Control-click) the app → select **Open**
   - Click **Open** in the dialog that appears
5. You only need to do step 4 once

</details>

<details>
<summary><strong>Linux Installation Details</strong></summary>

1. Download `labrynth_amd64.deb`
2. Open a terminal and run:
   ```bash
   sudo apt install ./labrynth_amd64.deb
   ```
3. The application installs to `/opt/labrynth/`
4. If you get errors about missing Qt libraries, see [TROUBLESHOOTING.md](TROUBLESHOOTING.md)

</details>

> 📝 **Note:**
> If you prefer to run from source code instead of using a pre-built installer,
> see the [SETUP_AND_USAGE.md](SETUP_AND_USAGE.md) guide for detailed
> instructions on cloning repositories, creating virtual environments, and
> launching the application manually.

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="installing-firmware"></a>
<details>
<summary><h2>4. Installing REACHER Arduino Firmware</h2></summary>

### 4.1 Download the Firmware

1. Go to the [reacher-firmware repository](https://github.com/Otis-Lab-MUSC/reacher-firmware)
2. Download the `operant_FR` folder (this is the Fixed-Ratio paradigm)

### 4.2 Open the Sketch

3. In the Arduino IDE, go to **File → Open**
4. Navigate to the downloaded `operant_FR` folder and open `operant_FR.ino`

### 4.3 Connect and Upload

5. Connect the Arduino UNO to your computer via the USB-A to USB-B cable
6. In the Arduino IDE, go to **Tools → Board → Arduino AVR Boards → Arduino UNO**
7. Go to **Tools → Port** and select the COM port for your Arduino
   - Windows: `COM3`, `COM4`, etc.
   - macOS: `/dev/cu.usbmodem*` or `/dev/cu.usbserial*`
   - Linux: `/dev/ttyACM0`, `/dev/ttyUSB0`, etc.
8. Click the **Upload** button (right arrow icon) or go to **Sketch → Upload**
9. Wait for the "Done uploading" message, then **close the Arduino IDE**

> ❗ **Important:**
> You must close the Arduino IDE (or at least close its Serial Monitor) before
> launching The Labrynth. Only one program can use the serial port at a time.

> 📝 **Note:**
> The `operant_FR` sketch is the standard Fixed-Ratio paradigm. Other paradigms
> are also available in the reacher-firmware repository:
> - `operant_PR-beta` — Progressive Ratio (beta)
> - `operant_VI-beta` — Variable Interval (beta)
> - `omission-beta` — Omission (beta)

<details>
<summary><strong>Pin Assignment Reference</strong></summary>

The firmware expects hardware to be connected to these specific Arduino UNO pins:

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

> 📝 **Note:**
> Each hardware component also needs a ground (GND) connection. Some components
> (like the laser) may also require an external power supply.

</details>

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="launching-application"></a>
<details>
<summary><h2>5. Launching the Application</h2></summary>

1. Launch **The Labrynth** from your desktop shortcut, Start menu, or Applications folder
2. Two things happen simultaneously:
   - A small **launcher window** appears titled "The Labrynth Launcher"
   - Your default web browser opens automatically to `http://localhost:7007`

The launcher window will display "Opening session in browser..." and then update to
"Session running in browser. (keep this window open)".

> ❗ **Important:**
> The launcher window **must remain open** for the entire session. Closing it
> shuts down the Panel server that powers the dashboard. Do not minimize it to
> the system tray — just leave it in the background.

> 📝 **Note:**
> If the browser does not open automatically, manually navigate to
> `http://localhost:7007` in any modern browser.

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="creating-session"></a>
<details>
<summary><h2>6. Creating a New Session</h2></summary>

Once the dashboard loads in your browser, you will see the **Welcome** tab with the
session creation area.

1. Under **"Create a session"**, type a name for your session in the text input
   (placeholder text: `"Enter a box name"`)
   - Use a descriptive name (e.g., `Box1`, `Mouse_001_Session_3`)
2. Click **"New wired session"**

A new tab labeled **"LOCAL - \<your name\>"** appears in the dashboard. Click it to
open your session.

> 📝 **Note:**
> The Labrynth supports multiple simultaneous sessions. Repeat these steps to
> create additional tabs, each connected to a different Arduino on a different
> COM port. A **"New wireless session (BETA)"** button is also available for
> network-based setups but is outside the scope of this protocol.

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="home-page"></a>
<details>
<summary><h2>7. Home Page — Connecting an Arduino</h2></summary>

When you open your session tab, you start on the **Home** page. The left sidebar
shows five page tabs: Home, Program, Hardware, Schedule, and Monitor.

### 7.1 Search for Microcontrollers

1. Under the **"COM Connection"** section, click the **"Search Microcontrollers"** button
   - The application scans for all available serial ports
   - Any connected Arduino devices appear in the **"Microcontroller"** dropdown

### 7.2 Connect

2. Select the correct COM port from the dropdown
3. Click **"Connect"**
   - The application establishes a serial connection at **115200 baud**
   - On successful connection, the **Firmware Information** section populates with
     the loaded sketch's name, version, and schedule type

> 📝 **Note:**
> The firmware running on the Arduino must match the paradigm you intend to use.
> If the **"File"**, **"Version"**, or **"Schedule"** fields show unexpected values
> after connecting, verify that the correct firmware was uploaded (see
> [Section 4](#installing-firmware)). A **"Disconnect"** button is available to
> release the serial port.

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="program-page"></a>
<details>
<summary><h2>8. Program Page — Setting Program Parameters</h2></summary>

Navigate to the **Program** page using the sidebar. This page controls session
presets, active hardware components, session limits, and file output configuration.

### 8.1 Select a Preset

Use the **"Select a preset:"** dropdown to choose a configuration template:

| Preset | Limit Type | Infusion Limit | Time Limit | Stop Delay |
|--------|-----------|---------------|------------|------------|
| **Custom** | *(no changes)* | — | — | — |
| **SA High** | Both | 10 | 3600 s | 10 s |
| **SA Mid** | Both | 20 | 3600 s | 10 s |
| **SA Low** | Both | 40 | 3600 s | 10 s |
| **SA Extinction** | Time | 0 | 3600 s | 0 s |

Selecting a preset automatically fills in the limit fields. Choose **Custom** to
configure everything manually.

### 8.2 Select Hardware Components

Check the boxes for each hardware component used in your experiment:

| Component Name | Description |
|----------------|-------------|
| **LH Lever** | Left-hand lever (pin 13) |
| **RH Lever** | Right-hand lever (pin 10) |
| **Cue** | Cue speaker (pin 3) |
| **Pump** | Infusion pump (pin 4) |
| **Lick Circuit** | Lick detection spout (pin 5) |
| **Laser** | Optogenetic laser (pin 6) |
| **Imaging Microscope** | Microscope trigger and timestamp (pins 9, 2) |

> 📝 **Note:**
> When a new session is created, **no components are pre-selected** and all
> devices start **disarmed** on the Hardware page. You must check the boxes
> for each component you intend to use before starting the session. Selecting
> a preset does not change the component selection.

| Component | Initial State | After Reset |
|-----------|--------------|-------------|
| **LH Lever** | Not selected | Selected |
| **RH Lever** | Not selected | Selected |
| **Cue** | Not selected | Selected |
| **Pump** | Not selected | Selected |
| **Lick Circuit** | Not selected | Not selected |
| **Laser** | Not selected | Not selected |
| **Imaging Microscope** | Not selected | Not selected |

> 📝 **Note:**
> When using the **Reset** button on the interface, the components revert to the
> default set: **LH Lever**, **RH Lever**, **Cue**, and **Pump**.

### 8.3 Set Session Limits

Configure how and when the session ends:

1. **Limit Type** — select one of: `Time`, `Infusion`, or `Both`
   - **Time:** session ends after a set duration
   - **Infusion:** session ends after a set number of infusions
   - **Both:** session ends when *either* limit is reached first

2. Set the limit values:
   | Field | Range | Step |
   |-------|-------|------|
   | **Hour(s)** | 0–10 | 1 |
   | **Minute(s)** | 0–59 | 1 |
   | **Second(s)** | 0–59 | 5 |
   | **Infusion(s)** | 0–100 | 1 |
   | **Stop Delay (s)** | 0–59 | 1 |

3. Click **"Set Program Limit"** to apply

### 8.4 File Configuration

Configure where session data will be saved:

1. Enter a file name in the **"File name:"** field (placeholder: `"e.g., experiment1.csv"`)
2. Enter a folder path in the **"Folder name:"** field (placeholder: `"e.g., ~/REACHER/DATA"`)
3. Click **"Set File Configuration"** to apply

> 📝 **Note:**
> The exported file will be saved as an `.xlsx` Excel workbook (not CSV, despite
> the placeholder text). See [Section 11](#monitor-page) for export details.

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="hardware-page"></a>
<details>
<summary><h2>9. Hardware Page — Adjusting Hardware Settings</h2></summary>

This section corresponds to **Step 21** in the manuscript.

Navigate to the **Hardware** page using the sidebar. Here you arm (enable) individual
devices and configure their operating parameters before starting the session.

### 9.1 Arming Devices

Each hardware component has a toggle button to arm or disarm it. Arming a device
sends an arm command to the Arduino over the serial connection. The toggle buttons
are labeled:

| Button | Component |
|--------|-----------|
| **Arm LH Lever** | Left-hand lever |
| **Arm RH Lever** | Right-hand lever |
| **Arm Cue** | Cue speaker |
| **Arm Pump** | Infusion pump |
| **Arm Lick Circuit** | Lick detection circuit |
| **Arm Laser** | Optogenetic laser |
| **Arm Scope** | Imaging microscope |

### 9.2 Active Lever Selection

Use the **"Active Lever"** menu to designate which lever delivers rewards. Options
are **"LH Lever"** and **"RH Lever"**.

### 9.3 Cue Configuration

Configure the auditory cue parameters:

| Parameter | Default | Range | Step |
|-----------|---------|-------|------|
| **Cue Frequency (Hz)** | 8000 | 0–20000 | 50 |
| **Cue Duration (ms)** | 1600 | 0–10000 | 50 |

After adjusting, click the **upload** button (upload icon) next to the cue settings
to send the configuration to the Arduino.

### 9.4 Laser Configuration

Configure optogenetic stimulation parameters:

| Parameter | Default | Range | Step |
|-----------|---------|-------|------|
| **Stim Mode** | Active-Press | Cycle, Active-Press | — |
| **Frequency (Hz)** | 40 | 1–100 | 1 |
| **Stim Duration (s)** | 5 | 1–60 | 5 |

After adjusting, click the **upload** button next to the laser settings to send the
configuration to the Arduino.

A **"Test Laser"** button is available to verify laser operation before starting
the session.

### 9.5 Other Components

The remaining components (Pump, Lick Circuit, Scope) are armed via their toggle
buttons but do not have additional configuration parameters on this page. Their
behavior is controlled by the firmware and the schedule settings.

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="schedule-page"></a>
<details>
<summary><h2>10. Schedule Page — Adjusting Within-Trial Dynamics</h2></summary>

Navigate to the **Schedule** page using the sidebar. This page contains sliders that
control the timing and ratio parameters for each trial.

### Within-Trial Dynamics

| Parameter | Default | Range | Step | Description |
|-----------|---------|-------|------|-------------|
| **Timeout Duration(s)** | 20 | 0–600 | 5 | Post-reward lockout period (seconds) |
| **Trace Duration(s)** | 0 | 0–60 | 1 | Delay between cue offset and reward availability (seconds) |

### Training Schedule

| Parameter | Default | Range | Step | Description |
|-----------|---------|-------|------|-------------|
| **Fixed Ratio Interval** | 1 | 1–50 | 1 | Number of active lever presses required per reward |

Each slider has an adjacent **upload** button (upload icon) to send the updated
value to the Arduino.

> 📝 **Note:**
> The Schedule page also contains sliders for **Progressive Ratio**, **Variable
> Interval**, and **Omission Interval** — these are used by alternative firmware
> paradigms and are not active under the Fixed-Ratio sketch.

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="monitor-page"></a>
<details>
<summary><h2>11. Monitor Page — Running the Session</h2></summary>

Navigate to the **Monitor** page using the sidebar. This is where you verify
settings, start the experiment, monitor progress in real time, and export data.

### 11.1 Verify Settings

Before starting, review your configuration:

- The **"REACHER Output:"** pane (dark-themed HTML panel on the right side of the
  interface) displays serial communication logs and system messages
- Click the **Start** button (play icon, green) to open the **"Settings Overview"**
  confirmation modal

### 11.2 Start the Session

1. Review the settings displayed in the **Settings Overview** modal
2. If everything is correct, click **"Ready to run?"** inside the modal to begin
3. The session starts and the header banner updates from "Program not started..." to
   reflect the active state

> 💡 **Tip:**
> If you need to go back and adjust your configuration, click anywhere outside
> the modal to dismiss it. This will not start the session or change any
> settings — you can return to any configuration page and reopen the modal when
> ready.

### 11.3 Monitor and Control

During the session:

- **Real-time plots** display lever presses, infusions, and other events
- The **REACHER Output:** pane shows a live log of serial events
- Use the **Pause** button (pause icon, yellow/warning) to temporarily halt the session
- Use the **Stop** button (stop icon, red/danger) to end the session permanently

> 📝 **Note:**
> To save a plot image, hover over any chart and click the **camera icon** in the
> Plotly modebar that appears in the top-right corner of the chart.

### 11.4 Export Data

After stopping the session:

1. Click **"Export data"** (download icon)
2. The data is saved as an **`.xlsx`** Excel workbook to the path you configured in
   [Section 8.4](#program-page)

The exported workbook contains **five sheets**:

| Sheet | Contents |
|-------|----------|
| **Session Summary** | Session metadata (name, duration, limits, file paths) |
| **Behavior Data** | Timestamped behavioral events (lever presses, infusions, etc.) |
| **Firmware Information** | Sketch name, version, and schedule type |
| **Hardware Settings** | Armed components and their parameter values |
| **Frame Timestamps** | Microscope frame sync timestamps (if imaging was used) |

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="testing-troubleshooting"></a>
<details>
<summary><h2>12. Testing and Troubleshooting</h2></summary>

After completing a session, it is good practice to test each hardware component
individually to verify correct operation. For detailed testing procedures,
diagnostic steps, and solutions to common problems, refer to the companion
troubleshooting guide:

**→ [TROUBLESHOOTING.md](TROUBLESHOOTING.md)**

The troubleshooting guide covers:
- Per-device hardware testing procedures
- Serial connection and COM port issues
- Platform-specific problems (Windows, macOS, Linux)
- Data export validation
- Troubleshooting flowcharts for quick diagnosis

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

---

*This document covers Labrynth v1.0.1 with REACHER v1.1.1 — Fixed-Ratio paradigm, wired sessions.*
