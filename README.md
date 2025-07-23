 # Digital-VLSI-SoC-OpenLane-Sky130

## 📘 Project Introduction
This repository showcases the complete physical design flow using the OpenLANE toolchain with the Sky130 open-source PDK. Executed as part of Digital VLSI SoC Design and Planning Workshop by VLSI System Design Corporation, the project offers a hands-on experience in chip design, starting from RTL to the final GDSII.

OpenLANE is a powerful, open-source RTL-to-GDSII automation framework that integrates tools like Yosys, OpenROAD, Magic, Netgen, Fault, OpenPhySyn, and more — all working cohesively to produce clean, manufacturable layouts without manual intervention. The flow is optimized specifically for the Skywater 130nm PDK, enabling the development of hard macros and full chips efficiently.

## Physical Design using OpenLANE & Sky130 PDK


## 📁 Repository Structure

- [Introduction](#introduction)
- [Overall Design Flow](#overall-design-flow)
- [DAY 1](#DAY-1)
  - [Synthesis](#synthesis)
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

# DAY-1
## Inception-of-Open-source-EDA,OpenLane-and-Sky130-PDK

we explore the fundamental physical elements of an integrated circuit (IC) : 

**Package, Chip, Pads, Core, Die, and IPs** : 
- The QFN-48 (Quad Flat No-lead) package is a surface-mounted IC package with 48 pins used to connect the chip to the outside world.

- Inside the package is the die, the actual silicon chip where transistors are fabricated.

- The core is the functional part of the die where the logic (e.g., processor, memory) is implemented.

- Surrounding the core are pads, which act as electrical contact points for interfacing signals from the core to the pins on the package.
  
- The chip as a whole includes the die, package, and connections.

The core of the chip will contain two types of blocks:

**Foundry IP Blocks** (e.g. ADC, DAC, PLL, and SRAM) = blocks which requires some amount of intelligent techniques to build which can only be designed by foundries.

**Macro blocks** (e.g. RISC-V SOC and SPI) = pure digital logic blocks compared to IP's which might require some analog parts.

<img width="1457" height="853" alt="182751377-2810d388-21b0-4df1-b1d4-c72176d80d28" src="https://github.com/user-attachments/assets/d6c9f443-dc08-4587-972a-c46c9b927dad" />
