# Drug Self-Administration in Head-fixed Mice

*This repository aggregates all code, hardware designs, and supporting resources for the Nature Protocols paper: "Drug self-administration in head-fixed mice" by Elizabeth M. Doncheck et al. (2026). For the full protocol, see the [published article](https://doi.org/10.1038/s41596-025-XXXXX).*

## Overview

This meta-repository is a centralized hub for reproducible implementation of the head-restrained intravenous and oral self-administration protocol in mice. It links to modular components of the **REACHER** open-source behavioral software stack (firmware, backend, application, hardware models), tracking the **REACHER-Suite v2 series** on the `develop` branch of each component repository.

For step-by-step usage, see the protocol guides in [`docs/`](docs/):

- [`PROTOCOL.md`](docs/PROTOCOL.md) — abridged top-to-bottom walkthrough (install → flash → wire → run → export)
- [`SETUP_AND_USAGE.md`](docs/SETUP_AND_USAGE.md) — long-form installation, hardware setup, and session workflow
- [`DEVELOPMENT.md`](docs/DEVELOPMENT.md) — architecture, development environment, and modification guide
- [`TROUBLESHOOTING.md`](docs/TROUBLESHOOTING.md) — diagnostic flowcharts and per-device testing

## Resources

| Repository | Description |
|-----------------|-------------|
| [`reacher-firmware`](https://github.com/Otis-Lab-MUSC/reacher-firmware) | Arduino C++ firmware for the rig. Five paradigms (FR, PR, VI, Omission, Pavlovian) sharing the `REACHERDevices` library; JSON serial protocol at 115200 baud. |
| [`reacher`](https://github.com/Otis-Lab-MUSC/reacher) | FastAPI + WebSocket backend with multi-threaded serial kernel. Dual-role deployment (primary GUI host or headless peripheral). Session management with mid-session SPLIT/RESTART, port locking, mDNS discovery (with `REACHER_BROKER_URL` unicast fallback), firmware uploader, event logging, and a systemd install script for headless Raspberry Pis. Listens on port `6229`. |
| [`labrynth`](https://github.com/Otis-Lab-MUSC/labrynth) | Desktop application shell. Bundles the React 19 dashboard, terminal CLI (`reacher-cli`), the firmware submodule, and a 6-stage PyInstaller build pipeline that produces platform installers (`labrynth-{VERSION}-{platform}.{ext}`) for Windows, macOS, and Linux. Includes a Machines panel for pairing and proxying remote peripherals (e.g., Raspberry Pis) over a 6-digit pairing code. |
| [`reacher-hardware-models`](https://github.com/Otis-Lab-MUSC/reacher-hardware-models) | STL files and machining specs for the head-fixed rig: lever assemblies, syringe-pump components, ethernet circuit boxes, and the head-fixation station. |

### See also

These two analysis packages, developed by the same lab, consume the data format produced by REACHER and are the standard downstream path for peri-event analysis:

- [`pynapse`](https://github.com/Otis-Lab-MUSC/pynapse) — neural data engine for aligning calcium-imaging signals with REACHER behavioral events and extracting peri-event tensors.
- `axplorer` — peri-event analysis dashboard built on Pynapse (PETH, response classification, figure export). Currently in development at the Otis Lab; not yet publicly released.

Both are outside the scope of this protocol guide; see their respective READMEs for installation and usage.

---

## Citation

If using these resources, please cite:

> Doncheck, E.M. et al. Drug self-administration in head-fixed mice. *Nat. Protoc.* (2026). https://doi.org/PLACEHOLDER

---

*This repository tracks REACHER-Suite v2 (develop). Last updated: April 2026.*
