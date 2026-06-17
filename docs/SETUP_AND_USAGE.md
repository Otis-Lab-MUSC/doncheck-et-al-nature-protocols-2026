# REACHER System - Installation and Usage Guide

This guide provides comprehensive instructions for setting up and using the REACHER (Rodent Experiment Application Controls and Handling Ecosystem for Research) system. The system consists of multiple coordinated repositories that work together to provide a complete drug self-administration behavioral control platform for head-fixed mice.

---

## Table of Contents

1. [System Overview](#system-overview)
2. [Prerequisites](#prerequisites)
3. [Git Installation and Configuration](#git-installation-and-configuration)
4. [Repository Cloning](#repository-cloning)
5. [Python Environment Setup](#python-environment-setup)
6. [Arduino IDE Setup](#arduino-ide-setup)
7. [Hardware Setup](#hardware-setup)
8. [Running a Session](#running-a-session)
9. [Troubleshooting](#troubleshooting)

---

## System Overview

The REACHER system is composed of four coordinated repositories:

### Repository Architecture

1. **reacher** (Core Python Package)
   - Core server for session management, event logging, and real-time control
   - Python-based framework
   - Provides the backend functionality and interface components
   - GitHub: `https://github.com/Otis-Lab-MUSC/reacher`

2. **labrynth** (Frontend Application)
   - Pre-built modifiable frontend application
   - Browser-based interface for running experiments
   - Manages multiple simultaneous sessions
   - GitHub: `https://github.com/Otis-Lab-MUSC/labrynth`

3. **Firmware** (Arduino Firmware)
   - Low-level Arduino sketches for hardware control
   - Controls solenoids, pumps, sensors, levers, and cues
   - Multiple paradigms: Fixed Ratio (FR), Progressive Ratio (PR), Variable Interval (VI), Omission, Pavlovian
   - Firmware source now ships inside the reacher package (folded into `reacher/firmware/`); the standalone repo is archived: `https://github.com/Otis-Lab-MUSC/reacher-firmware`

---

## Prerequisites

### System Requirements

| Component | Minimum | Recommended | High-Performance |
|-----------|---------|-------------|------------------|
| **CPU** | Quad-core (e.g., Intel i3) | 6-8 core (e.g., Intel i5/i7, AMD Ryzen 5) | 12-core+ (e.g., AMD Ryzen 9, Intel i9) |
| **RAM** | 8 GB | 16 GB | 32 GB+ |
| **Storage** | 256 GB SSD | 512 GB SSD | 1 TB NVMe SSD+ |
| **OS** | Windows 10 (64-bit), Linux, macOS | Ubuntu/Debian Linux, Windows 11, macOS | Linux with optimized kernels |
| **GPU** | Integrated graphics | Mid-range (NVIDIA GTX 1660) | High-end (NVIDIA RTX 3080+) |

### Required Software

- **Git**: Version control system
- **Python**: 3.10 or higher (3.10-3.13 recommended)
- **Arduino IDE** (optional): Only needed to build firmware from source. The app ships pre-compiled firmware and flashes it via the Firmware Upload card.
- **Web Browser**: For Labrynth interface

### Hardware Requirements

- Arduino-compatible microcontroller (Arduino Mega 2560 default/current; Arduino UNO supported as legacy)
- USB-A to USB-B cable
- Experimental rig components (levers, pumps, cues, etc.)
- Optional: Raspberry Pi for distributed setups

---

## Git Installation and Configuration

### Windows

#### Option 1: Git for Windows (Recommended)

1. Download Git installer:
   - Visit: https://git-scm.com/download/win
   - Download the 64-bit installer

2. Run the installer:
   ```powershell
   # If downloaded to Downloads folder, run:
   cd ~\Downloads
   .\Git-<version>-64-bit.exe
   ```

3. Installation options (recommended settings):
   - Editor: Use Visual Studio Code or your preferred editor
   - PATH: "Git from the command line and also from 3rd-party software"
   - HTTPS: "Use the OpenSSL library"
   - Line endings: "Checkout Windows-style, commit Unix-style"
   - Terminal: "Use Windows' default console window" or "Use MinTTY"
   - Extras: Enable "Git Credential Manager"

4. Verify installation:
   ```powershell
   git --version
   ```

#### Option 2: Via Chocolatey

```powershell
# Install Chocolatey package manager first (if not installed)
Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

# Install Git
choco install git -y

# Refresh environment variables
refreshenv

# Verify
git --version
```

### macOS

#### Option 1: Via Homebrew (Recommended)

```bash
# Install Homebrew if not already installed
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install Git
brew install git

# Verify
git --version
```

#### Option 2: Via Xcode Command Line Tools

```bash
# Install Xcode Command Line Tools (includes Git)
xcode-select --install

# Verify
git --version
```

### Linux (Ubuntu/Debian)

```bash
# Update package index
sudo apt update

# Install Git
sudo apt install git -y

# Verify installation
git --version
```

### Git Configuration

After installing Git, configure your identity (required for commits):

```bash
# Set your name and email
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Set default branch name to 'main' (optional but recommended)
git config --global init.defaultBranch main

# Enable credential caching to avoid repeated password prompts
# On Windows (using Git Credential Manager):
git config --global credential.helper manager

# On macOS (using Keychain):
git config --global credential.helper osxkeychain

# On Linux (cache for 1 hour):
git config --global credential.helper 'cache --timeout=3600'

# Verify configuration
git config --list
```

---

## Repository Cloning

### Choose Your Installation Directory

First, create a directory for the project:

**Windows (PowerShell):**
```powershell
# Create project directory
New-Item -Path "$HOME\Projects\REACHER" -ItemType Directory -Force
cd "$HOME\Projects\REACHER"
```

**macOS/Linux (Bash):**
```bash
# Create project directory
mkdir -p ~/Projects/REACHER
cd ~/Projects/REACHER
```

### Clone All Repositories

You have two options for cloning: HTTPS (simpler) or SSH (more secure, requires SSH key setup).

#### Option 1: HTTPS (Simpler, No SSH Key Required)

**Windows (PowerShell):**
```powershell
# Clone all repositories
git clone https://github.com/Otis-Lab-MUSC/doncheck-et-al-nature-protocols-2025.git
git clone https://github.com/Otis-Lab-MUSC/reacher.git
git clone https://github.com/Otis-Lab-MUSC/labrynth.git
# Firmware source now ships inside the reacher package (reacher/firmware/); the standalone
# reacher-firmware repository is archived and no longer needs cloning:
# https://github.com/Otis-Lab-MUSC/reacher-firmware

# Verify all repositories are present
ls
```

**macOS/Linux (Bash):**
```bash
# Clone all repositories
git clone https://github.com/Otis-Lab-MUSC/doncheck-et-al-nature-protocols-2025.git
git clone https://github.com/Otis-Lab-MUSC/reacher.git
git clone https://github.com/Otis-Lab-MUSC/labrynth.git
# Firmware source now ships inside the reacher package (reacher/firmware/); the standalone
# reacher-firmware repository is archived and no longer needs cloning:
# https://github.com/Otis-Lab-MUSC/reacher-firmware

# Verify all repositories are present
ls -la
```

#### Option 2: SSH (Recommended for Contributors)

First, set up SSH keys (if not already done):

**Generate SSH Key:**
```bash
# Generate a new SSH key
ssh-keygen -t ed25519 -C "your.email@example.com"

# When prompted, press Enter to accept default location
# Enter a passphrase (optional but recommended)

# Start ssh-agent
# On Windows (PowerShell):
Start-Service ssh-agent

# On macOS/Linux:
eval "$(ssh-agent -s)"

# Add your SSH key
# On Windows (PowerShell):
ssh-add $HOME\.ssh\id_ed25519

# On macOS/Linux:
ssh-add ~/.ssh/id_ed25519

# Copy public key to clipboard
# On Windows (PowerShell):
Get-Content $HOME\.ssh\id_ed25519.pub | Set-Clipboard

# On macOS:
pbcopy < ~/.ssh/id_ed25519.pub

# On Linux:
cat ~/.ssh/id_ed25519.pub | xclip -selection clipboard
# Or manually copy the output of: cat ~/.ssh/id_ed25519.pub
```

Then add the key to GitHub:
1. Go to GitHub.com → Settings → SSH and GPG keys
2. Click "New SSH key"
3. Paste your public key and save

**Clone with SSH:**
```bash
# Clone all repositories using SSH
git clone git@github.com:Otis-Lab-MUSC/doncheck-et-al-nature-protocols-2025.git
git clone git@github.com:Otis-Lab-MUSC/reacher.git
git clone git@github.com:Otis-Lab-MUSC/labrynth.git
# reacher-firmware is archived (firmware now lives in reacher/firmware/); cloning it is no longer necessary
# https://github.com/Otis-Lab-MUSC/reacher-firmware
```

---

## Python Environment Setup

### Install Python

#### Windows

1. **Download Python:**
   - Visit: https://www.python.org/downloads/
   - Download Python 3.11 (or latest 3.x version)

2. **Run installer:**
   - ☑️ Check "Add Python to PATH"
   - Click "Install Now"

3. **Verify installation:**
   ```powershell
   python --version
   pip --version
   ```

#### macOS

```bash
# Install Python via Homebrew
brew install python@3.11

# Verify installation
python3 --version
pip3 --version

# Create symlinks (optional, for convenience)
ln -s $(which python3) /usr/local/bin/python
ln -s $(which pip3) /usr/local/bin/pip
```

#### Linux (Ubuntu/Debian)

```bash
# Update package index
sudo apt update

# Install Python and pip
sudo apt install python3.11 python3.11-venv python3-pip -y

# Verify installation
python3 --version
pip3 --version
```

### Setup Virtual Environments

Create separate virtual environments for each Python repository:

#### For REACHER (Core Package)

**Windows (PowerShell):**
```powershell
# Navigate to reacher directory
cd "$HOME\Projects\REACHER\reacher"

# Create virtual environment
python -m venv venv

# Activate virtual environment
.\venv\Scripts\Activate.ps1

# Upgrade pip
python -m pip install --upgrade pip

# Install the reacher package in development mode (reacher uses pyproject.toml; there is no requirements.txt)
pip install -e .

# Deactivate when done
deactivate
```

**macOS/Linux (Bash):**
```bash
# Navigate to reacher directory
cd ~/Projects/REACHER/reacher

# Create virtual environment
python3 -m venv venv

# Activate virtual environment
source venv/bin/activate

# Upgrade pip
python -m pip install --upgrade pip

# Install the reacher package in development mode (reacher uses pyproject.toml; there is no requirements.txt)
pip install -e .

# Deactivate when done
deactivate
```

#### For Labrynth (Frontend Application)

**Windows (PowerShell):**
```powershell
# Navigate to labrynth directory
cd "$HOME\Projects\REACHER\labrynth"

# Create virtual environment
python -m venv venv

# Activate virtual environment
.\venv\Scripts\Activate.ps1

# Upgrade pip
python -m pip install --upgrade pip

# Install labrynth in development mode (uses pyproject.toml; pulls in reacher2p)
pip install -e .

# IMPORTANT: Install the reacher package
# Option 1: Install from the local reacher directory
pip install -e ..\reacher

# Option 2: Install from PyPI (the distribution name is reacher2p; the import stays `reacher`)
# pip install reacher2p

# Deactivate when done
deactivate
```

**macOS/Linux (Bash):**
```bash
# Navigate to labrynth directory
cd ~/Projects/REACHER/labrynth

# Create virtual environment
python3 -m venv venv

# Activate virtual environment
source venv/bin/activate

# Upgrade pip
python -m pip install --upgrade pip

# Install labrynth in development mode (uses pyproject.toml; pulls in reacher2p)
pip install -e .

# IMPORTANT: Install the reacher package
# Option 1: Install from the local reacher directory
pip install -e ../reacher

# Option 2: Install from PyPI (the distribution name is reacher2p; the import stays `reacher`)
# pip install reacher2p

# Deactivate when done
deactivate
```

### Verifying Python Setup

Test that everything is installed correctly:

**Windows (PowerShell):**
```powershell
# Activate labrynth environment
cd "$HOME\Projects\REACHER\labrynth"
.\venv\Scripts\Activate.ps1

# Test imports
python -c "import reacher; print(reacher.__version__)"
python -c "import fastapi; print('FastAPI imported successfully')"
python -c "import serial; print('PySerial imported successfully')"

deactivate
```

**macOS/Linux (Bash):**
```bash
# Activate labrynth environment
cd ~/Projects/REACHER/labrynth
source venv/bin/activate

# Test imports
python -c "import reacher; print(reacher.__version__)"
python -c "import fastapi; print('FastAPI imported successfully')"
python -c "import serial; print('PySerial imported successfully')"

deactivate
```

---

## Arduino IDE Setup

> **The Arduino IDE is optional.** The standard workflow flashes pre-compiled firmware from the app's "Firmware Upload" card (select board, select paradigm, upload). You only need the Arduino IDE to build firmware from source. The firmware still uses the ArduinoJson library when compiled from source.

### Install Arduino IDE

#### Windows

1. **Download Arduino IDE 2.x:**
   - Visit: https://www.arduino.cc/en/software
   - Download "Windows Win 10 and newer, 64 bits" installer

2. **Run installer and follow prompts**

3. **Verify installation:**
   - Launch Arduino IDE
   - Check Help → About Arduino to verify version

#### macOS

```bash
# Option 1: Via Homebrew Cask
brew install --cask arduino-ide

# Option 2: Download DMG from https://www.arduino.cc/en/software
# Then drag Arduino IDE to Applications folder
```

#### Linux (Ubuntu/Debian)

```bash
# Download AppImage
cd ~/Downloads
wget https://downloads.arduino.cc/arduino-ide/arduino-ide_latest_Linux_64bit.AppImage

# Make executable
chmod +x arduino-ide_*.AppImage

# Run Arduino IDE
./arduino-ide_*.AppImage

# Optional: Create desktop entry
mkdir -p ~/.local/share/applications
cat > ~/.local/share/applications/arduino-ide.desktop << EOF
[Desktop Entry]
Name=Arduino IDE
Exec=$HOME/Downloads/arduino-ide_latest_Linux_64bit.AppImage
Icon=arduino-ide
Type=Application
Categories=Development;
EOF
```

### Install Required Arduino Libraries

The firmware requires the following Arduino libraries:

1. **Launch Arduino IDE**

2. **Install ArduinoJson library:**
   - Go to: Tools → Manage Libraries (or Ctrl+Shift+I)
   - Search for "ArduinoJson"
   - Install "ArduinoJson by Benoit Blanchon" (version 6.x or later)

3. **Verify built-in libraries:**
   - `Arduino.h` - Built-in
   - `SoftwareSerial.h` - Built-in

### Configure Arduino for Your Board

1. **Connect your Arduino via USB**

2. **Select your board:**
   - Tools → Board → Arduino AVR Boards → Arduino Mega 2560 (default/current; Arduino Uno is supported as legacy)

3. **Select the COM port:**
   - Tools → Port → Select your Arduino's port
   - Windows: COM3, COM4, etc.
   - macOS: /dev/cu.usbmodem* or /dev/cu.usbserial*
   - Linux: /dev/ttyACM0, /dev/ttyUSB0, etc.

### Upload Firmware to Arduino

#### Standard workflow: Firmware Upload card (recommended)

The app ships pre-compiled firmware and flashes it via avrdude — the Arduino IDE is not required.

1. In the dashboard's **Device Setup** panel, find the **Firmware Upload** card.
2. Select the board from the board dropdown (**UNO** or **MEGA**).
3. Select a paradigm from the paradigm dropdown ("Select paradigm...").
4. Click the upload button and watch the progress indicator.

#### Optional: build from source in the Arduino IDE

Firmware source lives inside the reacher package at `reacher/firmware/`. Paradigm sketch folders are:
   - `fr` - Fixed Ratio (most commonly used)
   - `pr` - Progressive Ratio
   - `vi` - Variable Interval
   - `omission` - Omission contingency
   - `pavlovian` - Pavlovian

1. **Open the sketch:**

**Windows (PowerShell):**
```powershell
# Navigate to firmware directory (inside the reacher package)
cd "$HOME\Projects\REACHER\reacher\firmware"

# For Fixed Ratio paradigm (recommended to start):
# Open: fr\fr.ino in Arduino IDE
```

**macOS/Linux (Bash):**
```bash
# Navigate to firmware directory (inside the reacher package)
cd ~/Projects/REACHER/reacher/firmware

# For Fixed Ratio paradigm (recommended to start):
# Open: fr/fr.ino in Arduino IDE
```

2. **In Arduino IDE:**
   - File → Open → Navigate to the `.ino` file
   - Verify/Compile: Click the checkmark button (or Sketch → Verify/Compile)
   - Upload: Click the arrow button (or Sketch → Upload)
   - Wait for "Done uploading" message

3. **Verify upload:**
   - Open Serial Monitor: Tools → Serial Monitor
   - Set baud rate to 115200
   - You should see initialization messages from the Arduino

### Linux-Specific: USB Permissions

On Linux, you may need to add your user to the `dialout` group:

```bash
# Add user to dialout group
sudo usermod -a -G dialout $USER

# Log out and log back in, or run:
newgrp dialout

# Verify group membership
groups
```

---

## Hardware Setup

### Pin Configuration (Standard for All Firmware)

The REACHER firmware uses the following pin configuration:

| Pin | Function | Type | Notes |
|-----|----------|------|-------|
| 2 | Microscope timestamp trigger | Input | INT0, fixed (not remappable) |
| 3 | Cue speaker | PWM Output | Must be PWM-capable |
| 4 | Pump | Digital Output | Controls infusion pump |
| 5 | Lick circuit | Digital Input | INPUT_PULLUP; detects licking behavior |
| 6 | Laser | Digital Output | For optogenetic stimulation |
| 7 | Cue 2 | PWM Output | Second cue speaker |
| 8 | Pump 2 | Digital Output | Second infusion pump |
| 9 | Microscope trigger | Digital Output | Triggers imaging system |
| 10 | Right-hand lever | Digital Input | INPUT_PULLUP; active/inactive lever |
| 11 | SLM timestamp | Input | PCINT, remappable (8-13) |
| 13 | Left-hand lever | Digital Input | INPUT_PULLUP; active/inactive lever |

All pins are runtime-remappable **except pin 2** (INT0, fixed).

### Wiring Guidelines

1. **Connect peripherals to Arduino according to pin configuration**
2. **Use appropriate power supplies for high-current devices (pumps, lasers)**
3. **Implement proper isolation for inputs (optocouplers recommended)**
4. **Use pull-down resistors on input pins for stable readings**
5. **Connect Arduino to computer via USB**

### Testing Hardware

Arm and test devices from the dashboard's **Hardware Controls** (in the Session Configuration panel). The dashboard sends numeric `CommandCode` values over the serial link as newline-delimited JSON — the firmware no longer accepts the old string commands typed into the Arduino Serial Monitor.

1. Connect to the Arduino from the **Device Setup** panel.
2. In **Session Configuration → Hardware Controls**, arm a device (e.g. arming the pump sends `PUMP_ARM` = 401; arming the right-hand lever sends `LEVER_RH_ARM` = 1001).
3. Use the per-device **Test** buttons to fire a device (e.g. the pump Test sends `PUMP_TEST` = 403, the cue Test sends `CUE_TEST` = 303). The dashboard sends the corresponding numeric command automatically.

---

## Running a Session

### Quick Start: Pre-built Application (Easiest)

If you prefer not to run from source code, download the pre-built application:

Release assets are version-stamped, so download the one matching the latest
release from the [releases page](https://github.com/Otis-Lab-MUSC/labrynth/releases/latest)
(substitute the current `<version>`, e.g. `3.0.0-beta.6`).

**Windows:**
```powershell
# Download labrynth-<version>-windows-x64.exe (Inno Setup installer) from:
#   https://github.com/Otis-Lab-MUSC/labrynth/releases/latest
# then run the installer:
.\labrynth-<version>-windows-x64.exe
```

**macOS (Apple Silicon):**
```bash
# Download labrynth-<version>-macos-arm64.dmg from:
#   https://github.com/Otis-Lab-MUSC/labrynth/releases/latest
# Open the .dmg, drag Labrynth.app to /Applications, then launch:
open /Applications/Labrynth.app
```

**Linux (Ubuntu/Debian):**
```bash
# Download labrynth_<version>_amd64.deb from:
#   https://github.com/Otis-Lab-MUSC/labrynth/releases/latest
# then install:
sudo dpkg -i labrynth_<version>_amd64.deb
# Or use the portable AppImage (labrynth-<version>-linux-x64.AppImage):
#   chmod +x labrynth-<version>-linux-x64.AppImage && ./labrynth-<version>-linux-x64.AppImage
```

The bundled app opens your browser to `http://localhost:6229/` and runs a system-tray icon; close it from the tray to quit.

### Running from Source Code

#### Method 1: Run the backend (Recommended)

The reacher backend (FastAPI + Uvicorn) serves the React app same-origin on port 6229.

**Windows (PowerShell):**
```powershell
# From the labrynth directory (so REACHER_STATIC_DIR resolves)
cd "$HOME\Projects\REACHER\labrynth"
.\venv\Scripts\Activate.ps1

# Launch the backend (also available as the `reacher` entry point)
python -m reacher

# Then open http://localhost:6229/ in your browser
```

**macOS/Linux (Bash):**
```bash
# From the labrynth directory (so REACHER_STATIC_DIR resolves)
cd ~/Projects/REACHER/labrynth
source venv/bin/activate

# Launch the backend (also available as the `reacher` entry point)
python -m reacher

# Then open http://localhost:6229/ in your browser
```

#### Method 2: Frontend dev server (for frontend development)

For iterating on the React frontend, run the Vite dev server alongside the backend. It serves on `:5173` and proxies `/api` and `/ws` to the backend on `:6229`.

```bash
# Backend in one terminal (see Method 1), then:
cd ~/Projects/REACHER/labrynth/web
npm run dev
```

### Using the Labrynth Interface

Once the application is running, the dashboard has three sidebar panels — **Device Setup**, **Session Configuration**, and **Session**. Follow these steps:

#### Step 1: Connect to the Microcontroller (Device Setup panel)

1. Open the **Device Setup** panel.
2. In the **COM Port** section, choose your Arduino's port from the dropdown ("Select port...").
   - Windows: COM3, COM4, etc.
   - macOS: /dev/cu.usbmodem*
   - Linux: /dev/ttyACM0, /dev/ttyUSB0
3. Enter a session name in the session-name field (placeholder: *What would you like to name this session? (e.g., BOX_1)*).
4. The panel shows the connected **Port**, **Board**, and **Paradigm**. (To pair a remote machine instead, use **Add Machine** and enter its URL and the pairing code, e.g. `000-000`.)

#### Step 2: Configure the Experiment (Session Configuration panel)

1. Open the **Session Configuration** panel.
2. **File Configuration:** set **Filename:** (e.g., "mouse_01_session_1") and **Destination:** (the export folder).
3. **Session Preset** ("Select a session preset...") — for FR self-administration, choose one of: **SA High** (Both, 3600 s, infusion 10, stop delay 10 s), **SA Mid** (Both/3600/20/10), **SA Low** (Both/3600/40/10), or **SA Extinction** (Time/3600/30 infusions/10 s). Pavlovian presets (**Pavlovian - Acquisition**, **Pavlovian - Reversal**) are also available.
4. **Device Preset** ("Select a preset...") applies a hardware configuration.
5. **Hardware Controls** — arm/configure devices, grouped under **System Controls**, **Input Devices**, **Output Devices**, and **Two-Photon Devices**. Device labels include RH Lever, LH Lever, Cue 1, Cue 2, Pump 1, Pump 2, Laser, Lick Circuit, Microscope, and SLM. Use the per-device **Test** buttons to fire a device. Default device parameters: cue 8000 Hz / 1600 ms, pump 2000 ms, laser 40 Hz / 5000 ms, timeout 20000 ms (20 s), FR ratio 1.

#### Step 3: Run the Experiment (Session panel)

1. Open the **Session** panel.
2. Click **Start** ("Start a new session"); a pre-start summary modal lets you review settings before the session begins.
3. **During the session:**
   - A live event timeline shows all behavioral events with timestamps.
   - Use **Split segment** to mark a new segment, or **Restart session** to start over.
4. Click **Stop** ("Stop the session (requires confirmation)") to end the session.

#### Step 4: Export Data

1. After stopping the session, export the data from the **Session** panel.
2. The export is a **ZIP archive** containing `behavior_events.csv`, `frame_timestamps.csv`, `slm_timestamps.csv`, `arduino_config.json`, `event_log.jsonl`, `metadata.json`, and an optional `notes.txt`.
3. Default export location: your configured **Destination**, otherwise `~/Downloads`. (Live logs are written to `~/REACHER/LOG/<YYYY-MM-DD_HH-MM-SS>/`.)

### Data Output Format

The system exports a **ZIP archive** containing CSV and JSON files:

- `behavior_events.csv` — columns: `device, event, start_timestamp, end_timestamp, start_frame_index, end_frame_index` (segmented sessions produce `behavior_events_NNN.csv`)
- `frame_timestamps.csv` — columns: `frame_index, timestamp_ms`
- `slm_timestamps.csv` — columns: `event_index, timestamp_ms`
- `arduino_config.json` — firmware info + hardware settings
- `event_log.jsonl` — raw event stream
- `metadata.json` — session metadata
- `notes.txt` — optional session notes

**Behavioral Data (example):**
```csv
device,event,start_timestamp,end_timestamp,start_frame_index,end_frame_index
RH Lever,ACTIVE_PRESS,1523,1623,45,48
Pump 1,INFUSION,1650,3650,49,109
RH Lever,TIMEOUT_PRESS,4200,4300,125,128
```

**Frame Data (example - if using imaging):**
```csv
frame_index,timestamp_ms
1,1000
2,1033
3,1066
```

### Multiple Sessions

Labrynth supports running multiple sessions simultaneously:

1. Add machines from the **Device Setup** panel (local COM ports, or remote machines via **Add Machine**)
2. Each machine can connect to a different Arduino on a different COM port
3. Configure and run each session independently
4. Each session is named via the session-name field for easy identification

---

## Troubleshooting

### Git Issues

**Problem: "git: command not found"**
- **Solution:** Git is not installed or not in PATH
  - Reinstall Git and ensure "Add to PATH" is checked
  - Restart your terminal/PowerShell after installation

**Problem: "Permission denied (publickey)" when cloning via SSH**
- **Solution:** SSH key not configured
  - Use HTTPS cloning method instead
  - Or set up SSH key following the SSH setup instructions above

**Problem: "fatal: not a git repository"**
- **Solution:** You're not in a Git repository directory
  - Navigate to the correct directory with `cd`

### Python/Virtual Environment Issues

**Problem: "python: command not found"**
- **Windows:** Use `python` (not `python3`)
- **macOS/Linux:** Use `python3` explicitly, or create alias
- Verify Python is installed: Check PATH environment variable

**Problem: "cannot activate virtual environment"**
- **Windows PowerShell:** You may need to enable script execution:
  ```powershell
  Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
  ```
- **Windows:** Use `.\venv\Scripts\Activate.ps1` (not `activate`)
- **macOS/Linux:** Use `source venv/bin/activate` (not `.\venv\`)

**Problem: "No module named 'reacher'"**
- **Solution:** The reacher package is not installed in your labrynth environment
  ```bash
  # Activate labrynth venv first, then:
  pip install -e ../reacher
  ```

**Problem: Package installation fails with "error: externally-managed-environment"**
- **Solution (Linux):** This is a Debian/Ubuntu safeguard. Use virtual environments:
  ```bash
  # Always create and activate venv before installing packages
  python3 -m venv venv
  source venv/bin/activate
  pip install -e .
  ```

### Arduino/Firmware Issues

**Problem: Arduino IDE cannot find COM port**
- **Windows:** Install Arduino drivers from arduino.cc
- **macOS:** Grant permission in System Preferences → Security & Privacy
- **Linux:** Add user to `dialout` group (see Linux-Specific USB Permissions)
- Try unplugging and replugging Arduino
- Check Device Manager (Windows) or `ls /dev/tty*` (macOS/Linux)

**Problem: Upload fails with "avrdude: stk500_recv(): programmer is not responding"**
- **Solutions:**
  - Wrong board selected: Verify Tools → Board matches your Arduino
  - Wrong COM port: Verify Tools → Port shows your Arduino
  - USB cable issue: Try a different cable (must support data, not just power)
  - Reset Arduino: Press reset button before uploading

**Problem: "SoftwareSerial.h: No such file or directory"**
- **Solution:** Built-in library not found (rare)
  - Reinstall Arduino IDE
  - Verify board package is installed: Tools → Board → Boards Manager

**Problem: "ArduinoJson.h: No such file or directory"**
- **Solution:** Library not installed
  - Tools → Manage Libraries → Search "ArduinoJson" → Install

### Labrynth Application Issues

**Problem: "Address already in use" or port 6229 error**
- **Solution:** Another instance is running or port is occupied
  ```powershell
  # Windows: Find and kill process on port 6229
  netstat -ano | findstr :6229
  taskkill /PID <PID> /F
  ```
  ```bash
  # macOS/Linux: Find and kill process
  lsof -ti:6229 | xargs kill -9
  ```

**Problem: Browser doesn't open automatically**
- **Solution:** Manually navigate to `http://localhost:6229/`

**Problem: "Cannot connect to microcontroller"**
- **Solutions:**
  1. Verify Arduino is connected and COM port is correct
  2. Check that firmware is uploaded and running (open Serial Monitor at 115200 baud to verify)
  3. Ensure no other program is using the serial port (close Serial Monitor, other apps)
  4. Re-select the port from the COM Port dropdown in the Device Setup panel
  5. Restart the Arduino (unplug/replug USB)

**Problem: No data appearing in the event timeline or event log**
- **Solutions:**
  1. Verify hardware is properly connected to Arduino pins
  2. Check that devices are armed in **Session Configuration → Hardware Controls**
  3. Confirm events appear when you interact with hardware
  4. Verify the session is started in the **Session** panel
  5. Check that lever is set as "active" if expecting rewards

**Problem: Session crashes or stops responding**
- **Solutions:**
  1. Check Python console/terminal for error messages
  2. Verify your Python version is 3.10 or higher
  3. Ensure all dependencies are installed: `pip list`
  4. Try creating a fresh virtual environment
  5. Check system resources (CPU, RAM) - close other applications

### Data Export Issues

**Problem: "Cannot find data file" or export fails**
- **Solutions:**
  1. Verify you stopped the session before exporting
  2. Check file permissions in save directory
  3. Ensure destination directory exists and is writable
  4. Check the default export location: your configured Destination, otherwise `~/Downloads` (live logs are written to `~/REACHER/LOG/<timestamp>/`)

**Problem: Data file is empty or incomplete**
- **Solutions:**
  1. Verify session was actually started (not just armed)
  2. Check that events occurred during the session
  3. Ensure session was stopped properly (not forced closed)

### Platform-Specific Issues

#### Windows Specific

**Problem: "WindowsPath object not recognized"**
- Use raw strings or forward slashes in paths:
  ```python
  # Good:
  r"C:\Users\username\folder"
  "C:/Users/username/folder"
  
  # Bad:
  "C:\Users\username\folder"  # Backslashes interpreted as escape characters
  ```

**Problem: Long path names cause issues**
- Enable long paths in Windows:
  ```powershell
  # Run as Administrator
  New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1 -PropertyType DWORD -Force
  ```

#### macOS Specific

**Problem: "Operation not permitted" errors**
- Grant terminal/IDE full disk access:
  - System Preferences → Security & Privacy → Privacy → Full Disk Access
  - Add Terminal or your IDE

#### Linux Specific

**Problem: USB device permissions denied**
- Add user to dialout group (requires logout):
  ```bash
  sudo usermod -a -G dialout $USER
  ```

**Problem: ModuleNotFoundError after pip install**
- Ensure you're using the venv's Python:
  ```bash
  which python  # Should point to venv/bin/python
  ```

### Getting Additional Help

If you encounter issues not covered here:

1. **Check Documentation:**
   - Individual repository READMEs
   - Nature Protocols paper: Doncheck et al. (2025)
   - GitHub Issues: https://github.com/Otis-Lab-MUSC/

2. **Community Support:**
   - Open an issue on the relevant GitHub repository
   - Include:
     - Operating system and version
     - Python version
     - Complete error message
     - Steps to reproduce

3. **Contact:**
   - Author: Joshua Boquiren (thejoshbq@proton.me)
   - Lab: Otis Lab (http://www.otis-lab.org)

---

## Quick Reference Commands

### Git Commands
```bash
# Check status
git status

# Pull latest changes
git pull origin main

# Check current branch
git branch

# View commit history
git log --oneline -10
```

### Python Virtual Environment
```powershell
# Windows PowerShell
.\venv\Scripts\Activate.ps1  # Activate
deactivate                    # Deactivate
pip list                      # List installed packages
```

```bash
# macOS/Linux
source venv/bin/activate      # Activate
deactivate                    # Deactivate
pip list                      # List installed packages
```

### Running Labrynth
```bash
# From the labrynth directory, with venv activated
python -m reacher        # backend serves the React app on http://localhost:6229/
# For frontend dev: cd web && npm run dev   (Vite on :5173, proxies to :6229)
```

### Hardware Commands (sent automatically by the dashboard)

Devices are armed and tested from the dashboard's Hardware Controls. The dashboard/CLI
sends numeric `CommandCode` values over the serial link as newline-delimited JSON; the
firmware no longer accepts the old string commands. Examples:
```
CUE_ARM     = 301    CUE_TEST    = 303
PUMP_ARM    = 401    PUMP_TEST   = 403
LASER_ARM   = 601    LASER_TEST  = 603
LEVER_RH_ARM = 1001  LEVER_LH_ARM = 1301
SESSION_END  = 100
```

---

## Appendix: System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    REACHER System Architecture              │
└─────────────────────────────────────────────────────────────┘

    ┌──────────────────┐
    │   User Browser   │ ←──→ http://localhost:6229/
    └────────┬─────────┘
             │
             ↓
    ┌──────────────────┐
    │  Labrynth (UI)   │ ←──→ React frontend, session management
    └────────┬─────────┘
             │
             ↓
    ┌──────────────────┐
    │ REACHER (Python) │ ←──→ FastAPI + Uvicorn backend, serial communication
    └────────┬─────────┘
             │
             ↓ Serial (USB)
    ┌──────────────────┐
    │ Arduino + Firmware│ ←──→ Hardware control, event logging
    └────────┬─────────┘
             │
             ↓
    ┌──────────────────┐
    │  Physical Rig    │ ←──→ Levers, pumps, cues, sensors
    └──────────────────┘
```

---

## Citation

If using these resources, please cite:

> Doncheck, E.M. et al. Drug self-administration in head-fixed mice. *Nat. Protoc.* (2026). https://doi.org/PLACEHOLDER

---

*Last updated: January 2026 — reacher 3.0.0-beta.5 / labrynth 3.0.0-beta.6 / firmware 3.0.0-beta.5*
