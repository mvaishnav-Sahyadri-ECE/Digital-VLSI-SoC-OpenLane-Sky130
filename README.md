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

**Introduction to RISC-V** : 

- RISC-V stands for Reduced Instruction Set Computing – Version 5 and is designed with simplicity and modularity in mind.

- Unlike proprietary ISAs like ARM or x86, RISC-V is open, meaning anyone can implement or modify it freely.

- The architecture consists of a base integer set and optional extensions (e.g., for multiplication, atomic operations, floating point).

- RISC-V is ideal for education, research, and industrial design due to its transparency and scalability.

**Simplified RTL to GDSII Flow** : 

*Synthesis*

The Register Transfer Level (RTL) code, typically written in Verilog or VHDL, is translated into a gate-level netlist. This netlist is composed of logic gates and components from a pre-defined standard cell library. These cells have fixed sizes and electrical characteristics, provided by the Process Design Kit (PDK). During synthesis, optimizations such as constant propagation, logic simplification, and technology mapping are performed to ensure an area-efficient and logically equivalent design.

*Floor Planning and Power Planning*

This step involves defining the physical layout of the chip, including the die area, core area, margins, and reserved regions for IP blocks or macros. Floorplanning also includes decisions on I/O pin placement and defining power domains.
In power planning, a Power Distribution Network (PDN) is created to deliver clean power across the chip. The power rails and straps are usually placed on upper metal layers since they are thicker and have lower resistance, which helps reduce IR drop and ensures reliable power delivery.

*Placement*

Placement determines the exact physical location of standard cells within the defined floorplan. This process occurs in two phases:

- Global Placement: Estimates optimal positions to reduce wirelength and congestion, but may not obey all design rules.

- Detailed Placement: Adjusts the global result to ensure legal placement while minimizing additional wirelength or timing penalties.

*Clock Tree Synthesis (CTS)*

A clock tree is built to distribute the clock signal to all sequential elements (flip-flops) in the design. Since all flip-flops must receive the clock signal simultaneously to avoid timing violations like skew and jitter, structures such as H-trees or X-trees are used. CTS also buffers the clock signal and ensures balanced timing paths across different regions.

*Routing*

Routing connects the logically associated pins (nets) across the placed cells using horizontal and vertical metal layers defined in the PDK.

- Global Routing identifies approximate routing paths.

- Detailed Routing determines exact wiring with specific tracks, vias, and metal layers.
 
- The Sky130 PDK provides six metal layers, each with defined widths, spacings, pitches, and via rules to guide the router for legal and             manufacturable connections.

*Verification Before Sign-off*

Before the final design is ready for fabrication, several verification steps are required:

- DRC (Design Rule Check): Ensures the physical layout complies with all foundry-imposed rules like minimum spacing, width, enclosure, etc.

- LVS (Layout Versus Schematic): Compares the layout netlist against the schematic/netlist from the synthesis phase to confirm logical equivalence.

- Timing Analysis: Static Timing Analysis (STA) verifies that all setup and hold time constraints are satisfied across all paths and corners         (process, voltage, temperature).
The final Result is [GDSII file format.](https://anysilicon.com/semipedia/gdsii/)

RTL to GDSII flow : [Openlane](https://efabless.com/openlane)

**OpenLane Flow** : 

<img width="800" height="=500" alt="182759711-6b9352ec-7652-4589-af31-53a409eb2830" src="https://github.com/user-attachments/assets/f3cdde11-73e1-496e-9486-3d02f62ff65d"/>


OpenLane is an automated RTL to GDSII flow that utilizes various components such as OpenROAD, Yosys, Magic, Netgen, and custom scripts for design exploration and optimization.

The flow performs all ASIC implementation steps from RTL down to GDSII. OpenLane Interactive Mode Commands include a variety of functions such as setting the current netlist, running logic verification, setting the current def file, preparing lef files, preparing a liberty file, generating an exclude list file, sourcing configurations, specifying a path to save config_in.tcl, preparing a run, specifying a design folder, overwriting an existing run, specifying a path to save the run, specifying a name for a specific run, creating a tcl configuration file for a design, setting the verilog source code file, specifying the design's configuration file, setting a verbose output level, generating a padframe, saving views of a given run, changing save paths for various file types, labeling pins of a macro def, generating a verilog netlist from a def file, creating an obstruction, setting tracks on a layer, extracting core dimensions, running SPEF extraction and Static Timing Analysis, running antenna checks, saving environment variables, running OpenSTA timing analysis, checking for unmapped cells, checking for assign statements, and checking if the LEF was properly read.

- *Antenna Rules Violation* : long wire segments will act as antennna and will accumulate charges, this might damage the connected transistor gates. Solution is to either use bridging or antenna diode insertion to leak away the charges.  

Inside a specific design folder contains a config.tcl which overrides the default settings on OpenLANE. 


<img width="848" height="126" alt="Screenshot 2025-07-24 142941" src="https://github.com/user-attachments/assets/a61e7285-ca8d-45ef-adc8-2522e4e1103e" />



These configurations are specific to a design (e.g. clock period, clock port, verilog files...). 
The priority order for the OpenLANE settings:

1. sky130_xxxxx_config.tcl 
2. config.tcl

**Lab :**

- *running OpenLANE :*

  - Type docker to Invoke openLane tool (inside``` openlane/```)(```/Desktop/work/tools/openlane_working_dir/openlane```)

  - ```./flow.tcl -interactive``` = run script for automating the whole RTL to GDSII flow but in step by step ```-interactive mode```

  - ```package require openlane 0.9```
 
  <img width="723" height="250" alt="Screenshot 2025-07-24 120932" src="https://github.com/user-attachments/assets/42c4e8a2-9a39-4ac9-9a62-2be252e3fb0f" />
- *Design Setup Stage :*

  ```prep -design picorv32a``` = Setup the filesystem where the OpenLANE tools can dump the outputs. This also creates a ```run/``` folder inside    the specific design directory which contains the command ```log```files, ```results```, and the ```reports``` dump by each tools. These folders    will be empty for now except  for lef files generated by this design setup stage. This merged the cell LEF files ```.lef``` and technology LEF     files ```.tlef ```generating ```merged.nom.lef```   inside ```run/tmp/```
  
