# REACHER System Protocol Guide

> 📝 **Note:**
> **REACHER** — Rodent Experiment Application Controls and Handling Ecosystem for Research
>
> This guide covers every step from installing the
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
| 4 | [Installing REACHER Arduino Firmware](#installing-firmware) | Upload the fr firmware from the app |
| 5 | [Launching the Application](#launching-application) | Open Labrynth, open the dashboard |
| 6 | [Creating a New Session](#creating-session) | Name and create a session |
| 7 | [Device Setup — Connecting an Arduino](#home-page) | COM port search and connect |
| 8 | [Session Configuration — Program Parameters](#program-page) | Presets, limits, components, file config |
| 9 | [Session Configuration — Hardware Controls](#hardware-page) | Arm and configure devices |
| 10 | [Session Configuration — Paradigm Settings](#schedule-page) | Timeout, trace, ratio |
| 11 | [Session — Running the Session](#monitor-page) | Start, pause, stop, export |
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
- An **Arduino Mega 2560** (default; an Arduino UNO is also supported) connected via **USB-A to USB-B cable**
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

> 📝 **Note:**
> The Arduino IDE is **optional**. Labrynth ships pre-compiled firmware and uploads
> it for you from the **Firmware Upload** card (see [Section 4](#installing-firmware)),
> so the IDE is only needed if you want to build or modify the firmware from source.

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

Release assets are version-stamped — substitute the current `<version>` (e.g. `3.0.0`).

| Platform | Installer | Notes |
|----------|-----------|-------|
| **Windows** | `labrynth-<version>-windows-x64.exe` | Requires administrator privileges |
| **macOS** (Apple Silicon) | `labrynth-<version>-macos-arm64.dmg` | Right-click → Open on first launch (Gatekeeper bypass) |
| **Linux** | `labrynth_<version>_amd64.deb` | Install with `sudo dpkg -i labrynth_<version>_amd64.deb` |

<details>
<summary><strong>Windows Installation Details</strong></summary>

1. Download `labrynth-<version>-windows-x64.exe`
2. Double-click the installer
3. Click "Yes" when prompted by Windows for administrator privileges
4. Follow the Inno Setup wizard (accept defaults)
5. The application installs to `C:\Program Files\The Labrynth\`
6. A desktop shortcut is created (optional)

</details>

<details>
<summary><strong>macOS Installation Details</strong></summary>

1. Download `labrynth-<version>-macos-arm64.dmg`
2. Double-click the `.dmg` file to mount it
3. Drag "Labrynth" to your Applications folder
4. **First launch — Gatekeeper bypass:**
   - Right-click (or Control-click) the app → select **Open**
   - Click **Open** in the dialog that appears
5. You only need to do step 4 once

</details>

<details>
<summary><strong>Linux Installation Details</strong></summary>

1. Download `labrynth_<version>_amd64.deb`
2. Open a terminal and run:
   ```bash
   sudo dpkg -i labrynth_<version>_amd64.deb
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

### 4.1 Upload Firmware from the App

Labrynth ships **pre-compiled firmware** for every paradigm and flashes it for you, so
you do **not** need the Arduino IDE. You upload firmware from within the app, so first
launch Labrynth (see [Section 5](#launching-application)) and have your Arduino plugged in.

1. Connect the Arduino to your computer via the USB-A to USB-B cable
2. In the **Device Setup** panel, open the **Firmware Upload** card
3. Select your **board** — `MEGA` for the Arduino Mega 2560 (the default), or `UNO` for an Arduino UNO
4. Select the **paradigm** to flash:
   - `fr` — Fixed Ratio (the standard self-administration paradigm)
   - `pr` — Progressive Ratio
   - `vi` — Variable Interval
   - `omission` — Omission
   - `pavlovian` — Pavlovian
5. Start the upload and wait for the progress indicator to complete

> ❗ **Important:**
> Only one program can use the serial port at a time. Close the Arduino IDE Serial
> Monitor (or any other serial program) before uploading or connecting in Labrynth.

> 📝 **Note:**
> The firmware source lives inside the `reacher` package under `firmware/` (folders
> `fr`, `pr`, `vi`, `omission`, `pavlovian`); the former standalone
> [reacher-firmware repository](https://github.com/Otis-Lab-MUSC/reacher-firmware) is
> archived. To build or modify the firmware from source, open a sketch (e.g.
> `fr/fr.ino`) in the Arduino IDE — the only external library required is ArduinoJson —
> but for the standard protocol the in-app upload above is all you need.

<details>
<summary><strong>Pin Assignment Reference</strong></summary>

The firmware expects hardware to be connected to these default Arduino pins:

| Pin | Device | Direction | Description |
|-----|--------|-----------|-------------|
| 2 | Microscope Timestamp | INPUT (INT0, fixed) | Receives frame sync signals from imaging system |
| 3 | Cue 1 Speaker | OUTPUT | Plays tones through a connected speaker |
| 4 | Pump 1 | OUTPUT | Activates the infusion pump |
| 5 | Lick Circuit | INPUT_PULLUP | Detects lick contact on the spout |
| 6 | Laser | OUTPUT | Controls optogenetic laser stimulation |
| 7 | Cue 2 Speaker | OUTPUT | Secondary cue speaker |
| 8 | Pump 2 | OUTPUT | Secondary infusion pump |
| 9 | Microscope Trigger | OUTPUT | Sends trigger pulses to imaging system |
| 10 | Right-Hand (RH) Lever | INPUT_PULLUP | Detects right lever presses |
| 11 | SLM Timestamp | INPUT (PCINT) | Spatial light modulator frame sync |
| 13 | Left-Hand (LH) Lever | INPUT_PULLUP | Detects left lever presses |

> 📝 **Note:**
> Each hardware component also needs a ground (GND) connection. Some components
> (like the laser) may also require an external power supply. All pins are
> runtime-remappable **except pin 2** (the microscope timestamp, fixed on INT0).

</details>

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="launching-application"></a>
<details>
<summary><h2>5. Launching the Application</h2></summary>

1. Launch **The Labrynth** from your desktop shortcut, Start menu, or Applications folder
2. Two things happen simultaneously:
   - A **system-tray icon** appears (the application keeps running in the background)
   - Your default web browser opens automatically to `http://localhost:6229`

The REACHER backend (a local FastAPI server) starts and serves the dashboard to your browser.

> ❗ **Important:**
> Leave the application running (its tray icon present) for the entire session.
> Quitting it from the tray shuts down the backend server that powers the dashboard.

> 📝 **Note:**
> If the browser does not open automatically, manually navigate to
> `http://localhost:6229` in any modern browser.

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="creating-session"></a>
<details>
<summary><h2>6. Creating a New Session</h2></summary>

Once the dashboard loads in your browser, you start on the **Device Setup** panel,
which contains the session-creation area.

1. Type a name for your session in the text input
   (placeholder text: `"What would you like to name this session? (e.g., BOX_1)"`)
   - Use a descriptive name (e.g., `BOX_1`, `Mouse_001_Session_3`)
2. Create the session to open it

> 📝 **Note:**
> Labrynth can also connect to additional REACHER machines on your network via
> the **Add Machine** control in Device Setup (using a pairing code), but
> network-based multi-machine setups are outside the scope of this protocol.

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="home-page"></a>
<details>
<summary><h2>7. Device Setup — Connecting an Arduino</h2></summary>

When you open your session, you are on the **Device Setup** panel. The left sidebar
shows three panels: **Device Setup**, **Session Configuration**, and **Session**.

### 7.1 Select the COM Port

1. In the **"COM Port"** section, open the port dropdown (placeholder **"Select port..."**)
   - The application scans for all available serial ports
   - Any connected Arduino devices appear in the dropdown, with the detected board

### 7.2 Connect

2. Select the correct COM port from the dropdown
3. Connect to the device
   - The application establishes a serial connection at **115200 baud**
   - On successful connection, the panel populates the loaded firmware's
     **Board**, **Paradigm**, sketch name, and version

> 📝 **Note:**
> The firmware running on the Arduino must match the paradigm you intend to use.
> If the **Board**, **Paradigm**, or version fields show unexpected values
> after connecting, re-upload the correct firmware from the **Firmware Upload**
> card (see [Section 4](#installing-firmware)).

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="program-page"></a>
<details>
<summary><h2>8. Session Configuration — Program Parameters</h2></summary>

Open the **Session Configuration** panel from the sidebar. This panel controls session
presets, active hardware components, session limits, and file output configuration.

### 8.1 Select a Preset

Use the **"Session Preset"** dropdown (placeholder **"Select a session preset..."**)
to choose a configuration template:

| Preset | Limit Type | Infusion Limit | Time Limit | Stop Delay |
|--------|-----------|---------------|------------|------------|
| **SA High** | Both | 10 | 3600 s | 10 s |
| **SA Mid** | Both | 20 | 3600 s | 10 s |
| **SA Low** | Both | 40 | 3600 s | 10 s |
| **SA Extinction** | Time | 30 | 3600 s | 10 s |

Self-administration presets target the Fixed-Ratio (`fr`) paradigm. Pavlovian presets
(**Pavlovian - Acquisition**, **Pavlovian - Reversal**) are also available. Selecting a
preset automatically fills in the limit fields; adjust any field to configure manually.

### 8.2 Select Hardware Components

Check the boxes for each hardware component used in your experiment:

| Component Name | Description |
|----------------|-------------|
| **LH Lever** | Left-hand lever (pin 13) |
| **RH Lever** | Right-hand lever (pin 10) |
| **Cue 1** | Primary cue speaker (pin 3) |
| **Pump 1** | Primary infusion pump (pin 4) |
| **Lick Circuit** | Lick detection spout (pin 5) |
| **Laser** | Optogenetic laser (pin 6) |
| **Microscope** | Microscope trigger and timestamp (pins 9, 2) |

> 📝 **Note:**
> Secondary devices **Cue 2** (pin 7) and **Pump 2** (pin 8), and an **SLM**
> timestamp input (pin 11), are also available under Hardware Controls.

> 📝 **Note:**
> When a new session is created, **no components are pre-selected** and all
> devices start **disarmed** in Hardware Controls. You must enable each component
> you intend to use before starting the session. Selecting a preset does not
> change the component selection.

| Component | Initial State | After Reset |
|-----------|--------------|-------------|
| **LH Lever** | Not selected | Selected |
| **RH Lever** | Not selected | Selected |
| **Cue 1** | Not selected | Selected |
| **Pump 1** | Not selected | Selected |
| **Lick Circuit** | Not selected | Not selected |
| **Laser** | Not selected | Not selected |
| **Microscope** | Not selected | Not selected |

> 📝 **Note:**
> When using the **Reset** control, the components revert to the
> default set: **LH Lever**, **RH Lever**, **Cue 1**, and **Pump 1**.

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

In the **"File Configuration"** section, configure where session data will be saved:

1. Enter a base name in the **"Filename:"** field
2. Enter a folder path in the **"Destination:"** field
3. The configuration is applied to the active session

> 📝 **Note:**
> Exported session data is saved as a **`.zip` archive** (containing CSV and JSON
> files), not a single spreadsheet. If no destination is set, the archive is saved
> to your **`~/Downloads`** folder. See [Section 11](#monitor-page) for export details.

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="hardware-page"></a>
<details>
<summary><h2>9. Session Configuration — Hardware Controls</h2></summary>

This section corresponds to **Step 21** in the manuscript.

In the **Session Configuration** panel, open the **"Hardware Controls"** section
(grouped into **System Controls**, **Input Devices**, **Output Devices**, and
**Two-Photon Devices**). Here you arm (enable) individual devices and configure their
operating parameters before starting the session.

### 9.1 Arming Devices

Each hardware component has an arm/disarm toggle. Arming a device sends the
corresponding command to the Arduino over the serial connection:

| Device | Function |
|--------|----------|
| **LH Lever** | Left-hand lever |
| **RH Lever** | Right-hand lever |
| **Cue 1** | Primary cue speaker |
| **Pump 1** | Primary infusion pump |
| **Lick Circuit** | Lick detection circuit |
| **Laser** | Optogenetic laser |
| **Microscope** | Imaging microscope |

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
<summary><h2>10. Session Configuration — Paradigm Settings</h2></summary>

In the **Session Configuration** panel, the **Paradigm Settings** control the timing
and ratio parameters for each trial.

### Within-Trial Dynamics

| Parameter | Default | Range | Step | Description |
|-----------|---------|-------|------|-------------|
| **Timeout Duration(s)** | 20 | 0–600 | 5 | Post-reward lockout period (seconds) |
| **Trace Duration(s)** | 0 | 0–60 | 1 | Delay between cue offset and reward availability (seconds) |

### Training Schedule

| Parameter | Default | Range | Step | Description |
|-----------|---------|-------|------|-------------|
| **Fixed Ratio Interval** | 1 | 1–50 | 1 | Number of active lever presses required per reward |

After adjusting a value, send it to the Arduino using the adjacent control.

> 📝 **Note:**
> The Paradigm Settings adapt to the loaded firmware. Parameters for **Progressive
> Ratio** (`pr`), **Variable Interval** (`vi`), **Omission** (`omission`), and
> **Pavlovian** (`pavlovian`) apply only when the corresponding firmware is running;
> they are not active under the Fixed-Ratio (`fr`) sketch.

<p align="right"><a href="#table-of-contents">↑ Back to top</a></p>

</details>

<!-- ============================================================ -->

<a id="monitor-page"></a>
<details>
<summary><h2>11. Session — Running the Session</h2></summary>

Open the **Session** panel from the sidebar. This is where you verify
settings, start the experiment, monitor progress in real time, and export data.

### 11.1 Verify Settings

Before starting, review your configuration:

- The event log/timeline displays serial communication and behavioral events
- Click the **Start** control (play icon) to open the pre-start **summary modal**

### 11.2 Start the Session

1. Review the settings displayed in the start-summary modal
2. If everything is correct, confirm to begin the session
3. The session starts and the header updates to reflect the active state

> 💡 **Tip:**
> If you need to go back and adjust your configuration, click anywhere outside
> the modal to dismiss it. This will not start the session or change any
> settings — you can return to any configuration page and reopen the modal when
> ready.

### 11.3 Monitor and Control

During the session:

- **Real-time plots** display lever presses, infusions, and other events
- The event log shows a live stream of serial events
- Use the **Pause** control to temporarily halt data processing; it toggles to **Resume**
- Use **Split segment** to mark the current segment complete and begin a new one
- Use **Restart session** to restart the run (requires confirmation)
- Use the **Stop** control (stop icon) to end the session (requires confirmation)

> 📝 **Note:**
> To save a plot image, hover over any chart and click the **camera icon** in the
> Plotly modebar that appears in the top-right corner of the chart.

### 11.4 Export Data

After stopping the session:

1. Export the session data (download icon)
2. The data is saved as a **`.zip` archive** to the destination you configured in
   [Section 8.4](#program-page), or to `~/Downloads` if none was set

The archive contains CSV and JSON files:

| File | Contents |
|------|----------|
| **behavior_events.csv** | Timestamped behavioral events — columns: `device, event, start_timestamp, end_timestamp, start_frame_index, end_frame_index` (segmented sessions produce `behavior_events_NNN.csv`) |
| **frame_timestamps.csv** | Microscope frame sync timestamps (`frame_index, timestamp_ms`), if imaging was used |
| **slm_timestamps.csv** | SLM timestamps (`event_index, timestamp_ms`), if an SLM was used |
| **arduino_config.json** | Firmware information and hardware settings |
| **metadata.json** | Session metadata (name, duration, limits, counts, paths) |
| **event_log.jsonl** | Authoritative per-event session record |
| **notes.txt** | Session notes (only if provided) |

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

*This document covers Labrynth v3.0.0 with REACHER v3.0.0 — Fixed-Ratio paradigm.*
