# Digital-VLSI-SoC-OpenLane-Sky130

## 📘 Project Introduction
This repository showcases the complete physical design flow using the OpenLANE toolchain with the Sky130 open-source PDK. Executed as part of Digital VLSI SoC Design and Planning Workshop by VLSI System Design Corporation, the project offers a hands-on experience in chip design, starting from RTL to the final GDSII.

OpenLANE is a powerful, open-source RTL-to-GDSII automation framework that integrates tools like Yosys, OpenROAD, Magic, Netgen, Fault, OpenPhySyn, and more — all working cohesively to produce clean, manufacturable layouts without manual intervention. The flow is optimized specifically for the Skywater 130nm PDK, enabling the development of hard macros and full chips efficiently.

## Physical Design using OpenLANE & Sky130 PDK


## 📁 Repository Structure

## 🔗 Table of Contents

- [Introduction](#-introduction)
- [Overall Design Flow](#overall-design-flow)
- [OpenLANE Flow](#openlane-flow)
  - [Synthesis](#1-synthesis)
    - [Synthesis Strategies](#11-synthesis-strategies)
    - [Design Exploration Utility](#12-deign-exploration-utility)
    - [Design For Test - DFT Insertion](#13-design-for-test---dft-insertion)
  - [2. Floor Planning and Power Planning](#2-floor-planning-and-power-planning)
  - [3. Placement](#3-placement)
  - [4. Clock Tree Synthesis](#4-clock-tree-synthesis)
  - [5. Fake Antenna and Diode Swapping](#5-fake-antenna-and-diode-swapping)
  - [6. Routing](#6-routing)
  - [7. RC Extraction](#7-rc-extraction)
  - [8. Static Timing Analysis (STA)](#8-sta)
  - [9. Sign-off Steps](#9-sign-off-steps)
  - [10. GDSII Extraction](#10-gdsii-extraction)
- [OpenLane Installation and Environment Setup](#openlane-installation-and-environment-setup)
- [OpenLane Directory Structure](#openlane-directory-structure)
- [Working with OpenLane](#working-with-openlane)
  - [Start OpenLane](#start-openlane)
  - [Design Preparation](#design-preparation)
  - [Configuration Priority](#configuration-priority)
- [Synthesis](#synthesis)
  - [Key Concepts](#key-concepts)
  - [Utilization Factor](#utilisation-factor)
  - [Aspect Ratio](#aspect-ratio)
- [Floorplanning](#floorplanning)
  - [Pre-Placed Cells](#pre-placed-cells)
  - [Decoupling Capacitors](#decoupling-capacitors-to-the-pre-placed-cells)
  - [Power Planning](#power-planning)
  - [Pin Placement](#pin-placement)
  - [Floorplanning - OpenLane](#floorplanning---openlane)
- [Placement](#placement)
- [Cell Design Flow](#cell-design-flow)
  - [SPICE Deck Creation](#spice-deck-creation)
  - [Simulation in ngspice](#simulation-in-ngspce)
  - [VTC](#vtc)
  - [VTC with 2.5× PMOS Width](#vtc-with-25-x-w-25-times-channel-width-of-pmos)
  - [Transient Simulation](#transient-simulation)
- [Custom Design of SKY130 Standard Cell](#custom-design-of-sky130-standard-cell)
  - [SPICE Characterization](#spice-characterisation)
  - [LEF Extraction](#lef-extraction)
- [Integration with OpenLANE](#synthesis-floorplanning-with-custom-standard-cell)
- [Static Timing Analysis](#static-timing-analysis)
  - [Pre-CTS Timing Analysis in OpenROAD](#pre-cts-timing-analysis-in-openroad)
- [CTS](#cts)
- [PDN](#pdn)
- [Routing](#routing)
- [GDSII](#gdsii)
- [Acknowledgements](#acknowledgements)
- [References](#references)
---

## ⚙️ Tools & Environment

* **PDK:** Skywater130 open-source
* **OpenLANE version:** v2023.02 or newer
* **Design:** PicoRV32a (small RISC-V core)
* **Platform:** Ubuntu 20.04 (recommended)

---

## 🛠️ Setup Instructions

1. Clone the repo and OpenLANE environment:

```bash
git clone https://github.com/vaishnav-openlane/PicoRV32-Sky130.git
cd PicoRV32-Sky130
```

2. Install OpenLANE (refer: [https://github.com/The-OpenROAD-Project/OpenLane](https://github.com/The-OpenROAD-Project/OpenLane))
3. Launch the OpenLANE interactive shell:

```bash
cd openlane
make mount
```

4. Run synthesis and flow:

```tcl
run_synthesis
run_floorplan
run_placement
run_cts
run_routing
run_drc
run_lvs
run_antenna_check
```

---

## 📷 Screenshots

| Step | Description         | Screenshot             |
| ---- | ------------------- | ---------------------- |
| D1   | Initial Synthesis   | ![d1](images/day1.png) |
| D2   | Placement & Routing | ![d2](images/day2.png) |

(Feel free to update screenshots later.)

---

## 📌 TODO / Future Work

* [ ] Integrate power grid analysis
* [ ] Perform IR drop estimation
* [ ] Tapeout readiness checks

---

## 📄 License

This project is released under the [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0).

---

## 🙏 Acknowledgments

* VSD Corp for conducting the workshop
* OpenROAD, Skywater, and Efabless communities
* Inspiration: [shariethernet's repo](https://github.com/shariethernet/Physical-Design-with-OpenLANE-using-SKY130-PDK)

---

## 👤 Author

**M Vaishnav**
Final Year B.E., Electronics and Communication Engineering
# PicoRV32 SoC Physical Design using OpenLANE & Sky130 PDK

## 📘 Project Introduction

This repository presents a hands-on implementation of the **PicoRV32a SoC**, fully synthesized and laid out using the **automated OpenLANE RTL-to-GDSII flow** and the **Skywater 130 nm PDK**. Completed within the *VSD “Advanced Physical Design using OpenLANE/Sky130”* workshop, this project demonstrates a complete backend chip-design journey—from Verilog code to tape-out-ready layout.

**OpenLANE** is a fully open-source EDA toolchain (Apache‑2.0) that orchestrates a series of tools—**Yosys**, **OpenROAD**, **Magic**, **Netgen**, **Fault**, **OpenPhySyn**, **SPEF-Extractor**, and custom scripts—to generate clean, manufacturable GDSII layouts with minimal manual intervention.

### 🚀 Key Highlights:

* Integrated **custom-designed standard cell libraries** with Sky130 PDK.
* Resolved **setup and hold slack violations** via timing optimization.
* Performed **DRC, LVS, and antenna rule checks**.
* Demonstrated **fully automated RTL-to-GDSII flow** with OpenLANE.

---

## 📁 Repository Structure

```
├── src/                    # RTL and Verilog files
├── openlane/              # OpenLANE run configurations
│   └── picorv32a/
├── macros/                # Custom standard cell library files
├── results/               # Final GDS, DEF, netlist, reports
│   ├── gds/
│   ├── def/
│   └── logs/
├── images/                # Screenshots of layout, waveforms, etc.
├── scripts/               # Helper and automation scripts
└── README.md              # This file
```

---

## ⚙️ Tools & Environment

* **PDK:** Skywater130 open-source
* **OpenLANE version:** v2023.02 or newer
* **Design:** PicoRV32a (small RISC-V core)
* **Platform:** Ubuntu 20.04 (recommended)

---

## 🛠️ Setup Instructions

1. Clone the repo and OpenLANE environment:

```bash
git clone https://github.com/vaishnav-openlane/PicoRV32-Sky130.git
cd PicoRV32-Sky130
```

2. Install OpenLANE (refer: [https://github.com/The-OpenROAD-Project/OpenLane](https://github.com/The-OpenROAD-Project/OpenLane))
3. Launch the OpenLANE interactive shell:

```bash
cd openlane
make mount
```

4. Run synthesis and flow:

```tcl
run_synthesis
run_floorplan
run_placement
run_cts
run_routing
run_drc
run_lvs
run_antenna_check
```

---

## 📷 Screenshots

| Step | Description         | Screenshot             |
| ---- | ------------------- | ---------------------- |
| D1   | Initial Synthesis   | ![d1](images/day1.png) |
| D2   | Placement & Routing | ![d2](images/day2.png) |

(Feel free to update screenshots later.)

---

## 📌 TODO / Future Work

* [ ] Integrate power grid analysis
* [ ] Perform IR drop estimation
* [ ] Tapeout readiness checks

---

## 📄 License

This project is released under the [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0).

---

## 🙏 Acknowledgments

* VSD Corp for conducting the workshop
* OpenROAD, Skywater, and Efabless communities
* Inspiration: [shariethernet's repo](https://github.com/shariethernet/Physical-Design-with-OpenLANE-using-SKY130-PDK)

---

## 👤 Author

**M Vaishnav**
Final Year B.E., Electronics and Communication Engineering
