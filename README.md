# Drug Self-Administration in Head-fixed Mice

*This repository aggregates all code, hardware designs, and supporting resources for the Nature Protocols paper: "Drug self-administration in head-fixed mice" by Elizabeth M. Doncheck et al. (2026). For the full protocol, see the [published article](https://doi.org/10.1038/s41596-026-01406-1).*

## Overview

This meta-repository serves as a centralized hub for reproducible implementation of the head-restrained intravenous and oral self-administration protocol in mice. It links to modular components of the REACHER open-source behavioral software stack, custom hardware designs (including 3D-printable parts via Tinkercad), and optimized surgical protocols for catheter implantation.

For detailed step-by-step instructions, refer to the paper's protocol sections on equipment construction, software setup, surgery, and behavioral training.

## Resources

| Repository | Description |
|-----------------|-------------|
| [`reacher-firmware`](https://github.com/otis-lab-musc/reacher-firmware) | ⚠️ **Archived.** Firmware source is now maintained inside the [`reacher`](https://github.com/Otis-Lab-MUSC/reacher) package at `firmware/`. This repo is read-only. |
| [`reacher`](https://github.com/otis-lab-musc/reacher) | Core server for session management, event logging, and real-time control of self-administration trials. Supports Python-based extensions for custom rewards. |
| [`labrynth`](https://github.com/otis-lab-musc/labrynth) | Pre-built modifiable application. |
| [`reacher-hardware-models`](https://github.com/otis-lab-musc/reacher-hardware-models) | 3D models for various hardware prints. |

For comprehensive installation instructions or customization guides, please refer to the `docs` folder in this repository.

---

## Citation

If using these resources, please cite:

> Doncheck, E.M. et al. Drug self-administration in head-fixed mice. *Nat. Protoc.* (2026). https://doi.org/10.1038/s41596-026-01406-1

---

*Last updated: June 2026*
