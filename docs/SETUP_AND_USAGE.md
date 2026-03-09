# REACHER System - Installation and Usage Guide

This guide provides comprehensive instructions for setting up and using the REACHER (Rodent Experiment Application Controls and Handling Ecosystem for Research) system. The system consists of three coordinated repositories that work together to provide a complete drug self-administration behavioral control platform for head-fixed mice.

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

The REACHER system is composed of three coordinated repositories:

### Repository Architecture

1. **reacher** (Core Python Package)
   - FastAPI REST API + WebSocket server for session management, serial communication, event logging, and real-time hardware control
   - Python 3.10+
   - GitHub: `https://github.com/Otis-Lab-MUSC/reacher`

2. **labrynth** (Frontend Application)
   - React 19 + TypeScript + Vite frontend application with terminal CLI
   - Includes reacher-firmware as a git submodule
   - Build pipeline for standalone executables via PyInstaller
   - GitHub: `https://github.com/Otis-Lab-MUSC/labrynth`

3. **reacher-firmware** (Arduino Firmware)
   - Low-level Arduino sketches for hardware control
   - Controls solenoids, pumps, sensors, levers, and cues
   - 5 paradigms: Fixed Ratio (FR), Progressive Ratio (PR), Variable Interval (VI), Omission, Pavlovian
   - REACHERDevices shared library (v2.0.0)
   - Also included as a submodule of labrynth
   - GitHub: `https://github.com/Otis-Lab-MUSC/reacher-firmware`

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
- **Python**: 3.10 or higher (3.10–3.13 supported, 3.13 recommended)
- **Node.js**: 18+ with npm (for frontend development; not needed if using a pre-built executable)
- **Arduino IDE**: For uploading firmware to microcontrollers
- **arduino-cli** (optional): Command-line alternative to Arduino IDE
- **Web Browser**: For REACHER interface

### Hardware Requirements

- Arduino-compatible microcontroller (e.g., Arduino UNO)
- USB-A to USB-B cable
- Experimental rig components (levers, pumps, primary cue speaker, etc.)
- Optional: secondary cue speaker, secondary pump (for Pavlovian and advanced paradigms)
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
# Clone repositories
git clone https://github.com/Otis-Lab-MUSC/doncheck-et-al-nature-protocols-2025.git
git clone https://github.com/Otis-Lab-MUSC/reacher.git
git clone https://github.com/Otis-Lab-MUSC/labrynth.git

# Initialize firmware submodule within labrynth
cd labrynth
git submodule update --init --recursive
cd ..

# Verify all repositories are present
ls
```

**macOS/Linux (Bash):**
```bash
# Clone repositories
git clone https://github.com/Otis-Lab-MUSC/doncheck-et-al-nature-protocols-2025.git
git clone https://github.com/Otis-Lab-MUSC/reacher.git
git clone https://github.com/Otis-Lab-MUSC/labrynth.git

# Initialize firmware submodule within labrynth
cd labrynth
git submodule update --init --recursive
cd ..

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
# Clone repositories using SSH
git clone git@github.com:Otis-Lab-MUSC/doncheck-et-al-nature-protocols-2025.git
git clone git@github.com:Otis-Lab-MUSC/reacher.git
git clone git@github.com:Otis-Lab-MUSC/labrynth.git

# Initialize firmware submodule within labrynth
cd labrynth
git submodule update --init --recursive
cd ..
```

> **Note:** The firmware is included as a submodule in `labrynth/firmware/`. Clone `reacher-firmware` separately only if you need standalone firmware development outside of the labrynth build pipeline.

---

## Python Environment Setup

### Install Python

#### Windows

1. **Download Python:**
   - Visit: https://www.python.org/downloads/
   - Download Python 3.13 (or latest 3.x version, 3.10+ required)

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
brew install python@3.13

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
sudo apt install python3.13 python3.13-venv python3-pip -y

# Verify installation
python3 --version
pip3 --version
```

### Install Node.js (for Frontend Development)

Node.js is required for developing or building the React frontend. It is **not** needed if you only use the pre-built standalone executable.

#### Windows

```powershell
# Option 1: Download installer from https://nodejs.org/ (LTS recommended)

# Option 2: Via Chocolatey
choco install nodejs -y
refreshenv

# Verify
node --version
npm --version
```

#### macOS

```bash
# Via Homebrew
brew install node

# Verify
node --version
npm --version
```

#### Linux (Ubuntu/Debian)

```bash
sudo apt install nodejs npm -y

# Verify
node --version
npm --version
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

# Install reacher in development mode (uses pyproject.toml)
pip install -e .

# Optional: install dev dependencies (pytest, ruff, httpx)
# pip install -e ".[dev]"

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

# Install reacher in development mode (uses pyproject.toml)
pip install -e .

# Optional: install dev dependencies (pytest, ruff, httpx)
# pip install -e ".[dev]"

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

# Install the reacher backend package
pip install -e ..\reacher

# Install labrynth with CLI extras (prompt_toolkit, httpx, websockets)
pip install -e ".[cli]"

# Install frontend dependencies
cd web
npm ci
cd ..

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

# Install the reacher backend package
pip install -e ../reacher

# Install labrynth with CLI extras (prompt_toolkit, httpx, websockets)
pip install -e ".[cli]"

# Install frontend dependencies
cd web && npm ci && cd ..

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
python -c "import reacher; print('REACHER version:', reacher.__version__ if hasattr(reacher, '__version__') else 'imported successfully')"
python -c "import fastapi; print('FastAPI version:', fastapi.__version__)"
python -c "import serial; print('PySerial imported successfully')"

deactivate
```

**macOS/Linux (Bash):**
```bash
# Activate labrynth environment
cd ~/Projects/REACHER/labrynth
source venv/bin/activate

# Test imports
python -c "import reacher; print('REACHER version:', reacher.__version__ if hasattr(reacher, '__version__') else 'imported successfully')"
python -c "import fastapi; print('FastAPI version:', fastapi.__version__)"
python -c "import serial; print('PySerial imported successfully')"

deactivate
```

---

## Arduino IDE Setup

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

### Required Arduino Libraries

The firmware uses the **REACHERDevices** shared library, which is included locally in the repository at `libraries/REACHERDevices/`. No external Library Manager installs are needed.

**For Arduino IDE users:** Symlink or copy the library folder to your Arduino libraries directory:

```bash
# macOS/Linux
ln -s ~/Projects/REACHER/labrynth/firmware/libraries/REACHERDevices ~/Arduino/libraries/REACHERDevices

# Windows (PowerShell, run as administrator)
New-Item -ItemType SymbolicLink -Path "$HOME\Documents\Arduino\libraries\REACHERDevices" -Target "$HOME\Projects\REACHER\labrynth\firmware\libraries\REACHERDevices"
```

**For arduino-cli users:** Use the `--libraries` flag when compiling (see below).

### Configure Arduino for Your Board

1. **Connect your Arduino via USB**

2. **Select your board:**
   - Tools → Board → Arduino AVR Boards → Arduino Uno (or your specific board)

3. **Select the COM port:**
   - Tools → Port → Select your Arduino's port
   - Windows: COM3, COM4, etc.
   - macOS: /dev/cu.usbmodem* or /dev/cu.usbserial*
   - Linux: /dev/ttyACM0, /dev/ttyUSB0, etc.

### Upload Firmware to Arduino

#### Method 1: Arduino IDE

1. **Choose a paradigm from the firmware repository:**
   - `fr/fr.ino` — Fixed Ratio (most commonly used)
   - `pr/pr.ino` — Progressive Ratio
   - `vi/vi.ino` — Variable Interval
   - `omission/omission.ino` — Omission contingency
   - `pavlovian/pavlovian.ino` — Pavlovian classical conditioning

2. **Open the sketch:**

**Windows (PowerShell):**
```powershell
# Navigate to firmware directory (submodule in labrynth)
cd "$HOME\Projects\REACHER\labrynth\firmware"

# For Fixed Ratio paradigm (recommended to start):
# Open: fr\fr.ino in Arduino IDE
```

**macOS/Linux (Bash):**
```bash
# Navigate to firmware directory (submodule in labrynth)
cd ~/Projects/REACHER/labrynth/firmware

# For Fixed Ratio paradigm (recommended to start):
# Open: fr/fr.ino in Arduino IDE
```

3. **In Arduino IDE:**
   - File → Open → Navigate to the `.ino` file
   - Verify/Compile: Click the checkmark button (or Sketch → Verify/Compile)
   - Upload: Click the arrow button (or Sketch → Upload)
   - Wait for "Done uploading" message

4. **Verify upload:**
   - Open Serial Monitor: Tools → Serial Monitor
   - Set baud rate to 115200
   - You should see initialization messages from the Arduino

#### Method 2: arduino-cli (Command Line)

```bash
# Install Arduino AVR core
arduino-cli core install arduino:avr

# Compile all 5 paradigms to hex files
cd ~/Projects/REACHER/labrynth/firmware
bash compile.sh                     # compiles all 5 → hex/

# Upload a specific paradigm (e.g., FR)
arduino-cli upload -p /dev/ttyACM0 --fqbn arduino:avr:uno --input-dir hex/uno/fr
```

#### Method 3: REACHER Web Interface (Easiest)

Once the REACHER application is running, firmware can be uploaded directly from the web interface. Select a paradigm and COM port, and the application handles compilation and upload automatically using bundled avrdude.

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

The REACHER firmware uses the following pin configuration (from `Pins.h`):

| Pin | Function | Type | Notes |
|-----|----------|------|-------|
| 2 | Frame timestamp trigger | Input (INT0) | Microscope frame ISR |
| 3 | Primary cue speaker | PWM Output | Tone generation |
| 4 | Primary pump | Digital Output | Syringe pump relay |
| 5 | Lick circuit | Digital Input (PULLUP) | Lick detection |
| 6 | Laser | PWM Output | Optogenetic stimulation |
| 7 | Secondary cue speaker | PWM Output | Secondary tone output |
| 8 | Secondary pump | Digital Output | Secondary syringe pump relay |
| 9 | Microscope trigger | Digital Output | 50 ms trigger pulse |
| 10 | Right-hand lever | Digital Input (PULLUP) | Active/inactive lever |
| 12 | Left-hand lever | Digital Input (PULLUP) | Active/inactive lever |

### Wiring Guidelines

1. **Connect peripherals to Arduino according to pin configuration**
2. **Use appropriate power supplies for high-current devices (pumps, lasers)**
3. **Implement proper isolation for inputs (optocouplers recommended)**
4. **Use pull-up resistors on input pins for stable readings** (firmware enables internal pull-ups)
5. **Connect Arduino to computer via USB**

### Testing Hardware

Use the REACHER web interface or terminal CLI to test hardware connectivity — this is easier and safer than raw serial commands.

To test manually via Arduino IDE Serial Monitor (115200 baud):

1. Upload firmware and open Serial Monitor
2. Send `102` — Should respond with device identification (JSON)
3. The firmware uses numeric command codes for all operations:
   - `102` — Identify device (`*IDN?`)
   - `1001` — Arm right-hand lever
   - `401` — Arm pump
   - `301` — Arm cue speaker
   - `103` — Run test chain

> **Tip:** The web interface and terminal CLI provide a user-friendly way to arm devices and run tests without memorizing command codes.

---

## Running a Session

### Method 1: Standalone Executable (Easiest)

The standalone executable bundles the Python backend, React frontend, firmware hex files, and avrdude — no installation required.

**Linux:**
```bash
# Run the executable
./dist/REACHER/REACHER

# Or from the extracted directory:
cd REACHER
./REACHER
```

**Windows:**
```powershell
# Run the installer
.\REACHER-2.0.0-windows-x64.exe

# Or run the portable executable:
.\dist\REACHER\REACHER.exe
```

**macOS:**
```bash
# Open the application
open REACHER.app
```

The application starts the backend server on `http://localhost:6229` and automatically opens a browser window.

### Method 2: Terminal CLI

The terminal CLI provides a menu-driven interface using prompt_toolkit:

```bash
# Activate labrynth environment
cd ~/Projects/REACHER/labrynth
source venv/bin/activate

# Run the CLI (auto-starts backend if not running)
reacher-cli

# Or:
python -m cli
```

### Method 3: Development Mode (Two Terminals)

For active development, run the backend and frontend separately:

**Terminal 1 — Backend (FastAPI on port 6229):**
```bash
cd ~/Projects/REACHER/reacher
source venv/bin/activate
python -m reacher
```

**Terminal 2 — Frontend (Vite dev server on port 5173):**
```bash
cd ~/Projects/REACHER/labrynth/web
npm run dev
```

Open `http://localhost:5173` in your browser. The Vite dev server proxies API requests to the backend on port 6229.

### Using the REACHER Interface

Once the application is running, follow these steps:

#### Step 1: Session Panel — Create and Connect

1. In the **Session** panel, create a new session
2. Select the COM port for your Arduino from the dropdown
   - Windows: COM3, COM4, etc.
   - macOS: /dev/cu.usbmodem*
   - Linux: /dev/ttyACM0, /dev/ttyUSB0
3. Select a paradigm (FR, PR, VI, Omission, Pavlovian)
4. Optionally upload firmware directly from the interface

#### Step 2: Program Panel — Configure Experiment

1. Go to **Program** panel
2. **Apply a preset** or manually configure parameters:
   - Set paradigm-specific parameters (ratio, interval, step size, etc.)
   - Set session limits (time, infusions, trials)
3. **Start / Stop / Pause** the experiment from this panel

#### Step 3: Hardware Panel — Configure Devices

1. Navigate to **Hardware** panel
2. **Arm/Disarm Devices:**
   - Toggle components on/off (levers, pump, cue, laser, lick circuit, microscope)
   - Set which lever is "active" (delivers reward)
3. **Set Parameters:**
   - **Cue:** Frequency (Hz), duration (ms)
   - **Pump:** Duration (ms)
   - **Laser:** Duration (ms), frequency (Hz), mode (contingent/independent)
   - **Levers:** Timeout, ratio
4. **Test chains** to verify hardware connectivity

#### Step 4: Monitor Panel — Run Experiment

1. Switch to **Monitor** panel
2. **During the session:**
   - Live statistics display (presses, infusions, licks, trials)
   - Event timeline shows all behavioral events in real-time
3. **Control options:**
   - **Pause:** Temporarily pause the session (can resume)
   - **Stop:** End the session (cannot resume)

#### Step 5: Data Panel — Export Data

1. After stopping the session, go to **Data** panel
2. Set the filename and destination directory
3. Add optional notes
4. Click **Export** to save as a ZIP archive

### Data Output Format

The system generates a ZIP archive with the following structure:

```
{filename}.zip
├── behavior_events.csv     # Behavioral event log
├── frame_timestamps.csv    # Microscope frame timing
├── arduino_config.json     # Firmware info + hardware settings
├── metadata.json           # Session metadata and event counts
└── notes.txt               # Optional session notes
```

**Behavioral Events (`behavior_events.csv`):**
```csv
device,event,start_timestamp,end_timestamp,start_frame_index,end_frame_index
RH_LEVER,ACTIVE_PRESS,1523,1623,45,48
PUMP,INFUSION,1650,3650,49,110
RH_LEVER,TIMEOUT_PRESS,4200,4300,126,129
```

**Frame Timestamps (`frame_timestamps.csv`):**
```csv
frame_index,timestamp_ms
1,1000
2,1033
3,1066
```

**Default destination:** `~/Downloads/` (fallback: `~/REACHER/DATA/`)

### Multiple Sessions

REACHER supports running multiple sessions simultaneously:

1. Create additional sessions in the Session panel
2. Each session connects to a different Arduino on a different COM port
3. Configure and run each session independently

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
- Ensure Python 3.10 or higher is installed

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

### Node.js/npm Issues

**Problem: "node: command not found" or "npm: command not found"**
- Install Node.js 18+ (see [Install Node.js](#install-nodejs-for-frontend-development))
- Restart your terminal after installation

**Problem: `npm ci` fails with errors**
- Delete `node_modules/` and `package-lock.json`, then run `npm install`:
  ```bash
  cd ~/Projects/REACHER/labrynth/web
  rm -rf node_modules package-lock.json
  npm install
  ```
- Ensure Node.js version is 18 or higher: `node --version`

**Problem: React frontend shows blank page**
- Check the browser console (F12) for errors
- Verify the backend is running on port 6229
- In development mode, ensure Vite dev server is running (`npm run dev` in `web/`)
- Clear browser cache and reload

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

**Problem: "REACHERDevices.h: No such file or directory"**
- **Solution:** The shared library is not linked
  - Symlink or copy `libraries/REACHERDevices/` to your Arduino libraries directory (see [Required Arduino Libraries](#required-arduino-libraries))
  - For arduino-cli, use the `--libraries libraries/` flag

**Problem: `compile.sh` fails**
- Ensure `arduino-cli` is installed and the `arduino:avr` core is installed:
  ```bash
  arduino-cli core install arduino:avr
  ```
- Run from the firmware directory: `cd ~/Projects/REACHER/labrynth/firmware && bash compile.sh`

**Problem: Firmware upload from web interface fails**
- Verify the Arduino is connected and the correct COM port is selected
- Ensure no other program is using the serial port (close Serial Monitor)
- Try uploading via Arduino IDE or arduino-cli as a fallback

### Application Issues

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
- **Solution:** Manually navigate to `http://localhost:6229`

**Problem: WebSocket connection fails**
- Verify the backend is running on port 6229
- Check browser console for WebSocket errors
- Ensure no firewall or proxy is blocking WebSocket connections
- Try refreshing the page

**Problem: "Cannot connect to microcontroller"**
- **Solutions:**
  1. Verify Arduino is connected and COM port is correct
  2. Check that firmware is uploaded and running (open Serial Monitor at 115200 baud to verify)
  3. Ensure no other program is using the serial port (close Serial Monitor, other apps)
  4. Restart the Arduino (unplug/replug USB)

**Problem: No data appearing in monitor or event timeline**
- **Solutions:**
  1. Verify hardware is properly connected to Arduino pins
  2. Check that devices are "armed" in Hardware panel
  3. Verify the session is started in Program panel
  4. Check that lever is set as "active" if expecting rewards

**Problem: Session crashes or stops responding**
- **Solutions:**
  1. Check the terminal/console for error messages
  2. Verify your Python version is 3.10 or higher
  3. Ensure all dependencies are installed: `pip list`
  4. Try creating a fresh virtual environment
  5. Check system resources (CPU, RAM) — close other applications

### Data Export Issues

**Problem: "Cannot find data file" or export fails**
- **Solutions:**
  1. Verify you stopped the session before exporting
  2. Check file permissions in save directory
  3. Ensure destination directory exists and is writable
  4. Check default location: `~/Downloads/` or fallback `~/REACHER/DATA/`

**Problem: Data file is empty or incomplete**
- **Solutions:**
  1. Verify session was actually started (not just armed)
  2. Check that events occurred during the session
  3. Ensure session was stopped properly (not force-closed)

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

### Running REACHER
```bash
# Standalone executable
./dist/REACHER/REACHER

# Terminal CLI (with labrynth venv activated)
reacher-cli

# Backend only
python -m reacher

# Frontend dev server
cd web && npm run dev
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

### Serial Command Codes (for testing via Serial Monitor at 115200 baud)
```
102    — Identify device (*IDN?)
1001   — Arm right-hand lever
1301   — Arm left-hand lever
401    — Arm pump
301    — Arm cue speaker
601    — Arm laser
901    — Arm microscope
103    — Run test chain
101    — Start program
100    — Stop program
105    — Pause/resume program
```

---

## Appendix: System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                    REACHER v2.0.0 System Architecture               │
└─────────────────────────────────────────────────────────────────────┘

    ┌──────────────────────┐      ┌──────────────────────┐
    │   Browser (React 19) │      │   Terminal CLI        │
    │   http://localhost:   │      │   (prompt_toolkit)    │
    │     5173 (dev)       │      │   reacher-cli         │
    │     6229 (prod)      │      │                       │
    └──────────┬───────────┘      └──────────┬────────────┘
               │ REST + WebSocket             │ HTTP + WebSocket
               └──────────┬──────────────────-┘
                          │
                          ↓
               ┌──────────────────────┐
               │  FastAPI Backend     │ ←──→ Port 6229
               │  (reacher)           │      REST API + WebSocket
               │  Session management  │      CORS, static files
               │  Event logging       │      API key auth
               └──────────┬───────────┘
                          │
                          ↓ Serial JSON (115200 baud)
               ┌──────────────────────┐
               │  Arduino (ATmega328P)│ ←──→ REACHERDevices library
               │  Firmware v2.0.0     │      5 paradigms:
               │                      │      FR, PR, VI, Omission,
               │                      │      Pavlovian
               └──────────┬───────────┘
                          │
                          ↓
               ┌──────────────────────┐
               │  Physical Rig        │ ←──→ Levers, pumps (×2),
               │                      │      cue speakers (×2),
               │                      │      laser, lick circuit,
               │                      │      microscope
               └──────────────────────┘
```

---

## Citation

If using these resources, please cite:

> Doncheck, E.M. et al. Drug self-administration in head-fixed mice. *Nat. Protoc.* (2026). https://doi.org/PLACEHOLDER

---

*Last updated: March 2026*
