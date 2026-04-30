# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Documentation-only meta-repository for the Nature Protocols paper:

> Doncheck, E.M. et al. Drug self-administration in head-fixed mice. *Nat. Protoc.* (2026).

There is no source code here — no build, test, or lint. The paper's implementations live in four external repos under the `otis-lab-musc` GitHub org and (locally) as siblings under `~/Otis-Lab/Projects/REACHER-Suite/`. This repo is the citation hub that ties them together for readers of the published article.

## What's in `docs/`

| File | Covers |
|---|---|
| `PROTOCOL.md` | Abridged step-by-step path: Arduino IDE → REACHER install → firmware → app → session setup |
| `SETUP_AND_USAGE.md` | Long-form installation: Git/Python/Node, Arduino IDE, hardware, running sessions |
| `DEVELOPMENT.md` | Three-layer architecture (firmware → backend → frontend), serial protocol (JSON @ 115200 baud), paradigms (FR, PR, VI, Omission, Pavlovian), modification guide |
| `TROUBLESHOOTING.md` | Diagnostic tree across hardware, install, Arduino, sessions, CLI, data export, per-platform issues |

## Where the real code lives

| External repo (in README links) | Purpose | Local path |
|---|---|---|
| `otis-lab-musc/reacher-firmware` | Arduino firmware | `~/Otis-Lab/Projects/REACHER-Suite/reacher-firmware/` |
| `otis-lab-musc/reacher` | Python FastAPI backend | `~/Otis-Lab/Projects/REACHER-Suite/reacher/` |
| `otis-lab-musc/labrynth` | React frontend + CLI | `~/Otis-Lab/Projects/REACHER-Suite/labrynth/` |
| `otis-lab-musc/reacher-hardware-models` | 3D-printable enclosures | `~/Otis-Lab/Projects/REACHER-Suite/reacher-hardware-models/` |

Whenever a doc mentions a command, port, pin, or version, the source of truth is in those checkouts — not in this repo.

## Editing the docs

- Cross-check any technical claim (port number, pin assignment, command, version, paradigm name) against the corresponding REACHER-Suite checkout's `develop` branch before editing. Drift between this hub and the actual code is the failure mode the v2 refresh was correcting.
- The four docs overlap by design — `PROTOCOL.md` is the abridged path; the others are the long-form companions. A change to a flag, port, or paradigm in one usually needs mirroring across all four.
- Plain text only. All four docs are emoji-free; don't reintroduce emoji (`PROTOCOL.md` previously used 📝 / 💡 / ❗ admonition icons — those were stripped during the v2 rewrite).
- Current terminology: backend port is `6229` (not `7007`); the component is "REACHER backend / FastAPI/Uvicorn" (not "The Labrynth / Panel server"). Application name is "Labrynth" (no leading "The"); the system as a whole is "REACHER" or "REACHER-Suite". **Pin docs to "REACHER-Suite v2 (develop)"** — the v2.x umbrella covers the develop-branch state of each component (`reacher` 2.0.1, `labrynth` 2.1.x-dev, firmware v2.0.0). The umbrella is intentional: example output values that cite a specific version (e.g., `__version__ = "2.0.1"`) should match the develop value, but high-level statements about the system should use "v2" rather than a specific minor.
- Output paths: `~/REACHER/LOG/{YYYY-MM-DD_HH-MM-SS}/` is the per-session backend log folder (always written every session, segmented as `behavior_events_001.csv`, `behavior_events_002.csv`, ... when the user calls `/sessions/{id}/split` or `/sessions/{id}/restart`); the user-initiated ZIP export goes to whatever path the user types (placeholder `~/REACHER/DATA/`, fallback `~/Downloads/` if blank). Keep these distinct in the docs — the v1.0.1-era write-ups conflated them.
- Installer naming convention (Labrynth `develop`): `labrynth-{VERSION}-windows-x64.exe`, `labrynth-{VERSION}-macos-arm64.dmg`, `labrynth_{VERSION}_amd64.deb`, `labrynth-{VERSION}-linux-x64.tar.gz`, `labrynth-{VERSION}-linux-x64.AppImage`. `{VERSION}` is the git tag with the leading `v` stripped. The old `REACHER-X.Y.Z-*` placeholder names are wrong.
- Develop-only features to keep in mind when editing (these are absent from `main` and must not be described as if they live elsewhere): multi-machine pairing UI in `MachinePanel.tsx`, mDNS discovery and `_reacher._tcp.local.` Zeroconf service, peripheral-vs-primary detection by `STATIC_DIR` presence, 6-digit pairing codes, `/api/proxy/{deviceId}/...` request routing, `REACHER_BROKER_URL` unicast fallback, session SPLIT/RESTART endpoints, numbered `behavior_events_NNN.csv` outputs, `REACHER_WS_PING_INTERVAL` and `REACHER_WS_PING_TIMEOUT` env vars, and the `reacher` PyPI install plus `scripts/install.sh` systemd unit for headless Raspberry Pis. The `LASER_SET_TRACE` (cmd 673) command was removed on develop — don't reintroduce it.
