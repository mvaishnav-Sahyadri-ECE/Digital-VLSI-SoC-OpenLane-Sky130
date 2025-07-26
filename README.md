## Physical Design using OpenLANE & Sky130 PDK

## 📘 Project Introduction
This repository showcases the complete physical design flow using the OpenLANE toolchain with the Sky130 open-source PDK. Executed as part of Digital VLSI SoC Design and Planning Workshop by VLSI System Design Corporation, the project offers a hands-on experience in chip design, starting from RTL to the final GDSII.

OpenLANE is a powerful, open-source RTL-to-GDSII automation framework that integrates tools like Yosys, OpenROAD, Magic, Netgen, Fault, OpenPhySyn, and more — all working cohesively to produce clean, manufacturable layouts without manual intervention. The flow is optimized specifically for the Skywater 130nm PDK, enabling the development of hard macros and full chips efficiently.



## 📁 Repository Structure

- [DAY 1](#DAY-1)
  - [Theory](#DAY-1)
    - [Introduction to RISC-V](#Introduction-to-RISC-V)
    - [Simplified RTL to GDSII Flow](#Simplified-RTL-to-GDSII-Flow)
    - [OpenLane Flow](#OpenLane-Flow)
  - [Lab](#Lab-1)
    - [Synthesis](#Lab-1)
    - [Estimation of Flip Flop Ratio](#Estimation-of-Flip-Flop-Ratio)
    - [Slack](#Slack)
- [DAY 2](#Day-2)
  - [Theory](#DAY-2)
    - [Floorplan](#Floorplan)
    - [Placement](#Placement)
  - [Lab](#Lab-2)
    - [Floorplan](#Floor-Plan)
    - [Placement](#Place-Ment)
    - [Characterization](#Library-Characterization)
    - [Estimation of area of the die](#Estimation-of-area-of-the-die)
- [Day 3](#Day-3)
  - [Theory](#DAY-3)
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


# DAY-1
# Inception-of-Open-source-EDA,OpenLane-and-Sky130-PDK

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

<img width="848" height="853" alt="182751377-2810d388-21b0-4df1-b1d4-c72176d80d28" src="https://github.com/user-attachments/assets/d6c9f443-dc08-4587-972a-c46c9b927dad" />

## Introduction to RISC-V

- RISC-V stands for Reduced Instruction Set Computing – Version 5 and is designed with simplicity and modularity in mind.

- Unlike proprietary ISAs like ARM or x86, RISC-V is open, meaning anyone can implement or modify it freely.

- The architecture consists of a base integer set and optional extensions (e.g., for multiplication, atomic operations, floating point).

- RISC-V is ideal for education, research, and industrial design due to its transparency and scalability.

## Simplified RTL to GDSII Flow 

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

## OpenLane Flow

<img width="848" height="=500" alt="182759711-6b9352ec-7652-4589-af31-53a409eb2830" src="https://github.com/user-attachments/assets/f3cdde11-73e1-496e-9486-3d02f62ff65d"/>


OpenLane is an automated RTL to GDSII flow that utilizes various components such as OpenROAD, Yosys, Magic, Netgen, and custom scripts for design exploration and optimization.

The flow performs all ASIC implementation steps from RTL down to GDSII. OpenLane Interactive Mode Commands include a variety of functions such as setting the current netlist, running logic verification, setting the current def file, preparing lef files, preparing a liberty file, generating an exclude list file, sourcing configurations, specifying a path to save config_in.tcl, preparing a run, specifying a design folder, overwriting an existing run, specifying a path to save the run, specifying a name for a specific run, creating a tcl configuration file for a design, setting the verilog source code file, specifying the design's configuration file, setting a verbose output level, generating a padframe, saving views of a given run, changing save paths for various file types, labeling pins of a macro def, generating a verilog netlist from a def file, creating an obstruction, setting tracks on a layer, extracting core dimensions, running SPEF extraction and Static Timing Analysis, running antenna checks, saving environment variables, running OpenSTA timing analysis, checking for unmapped cells, checking for assign statements, and checking if the LEF was properly read.

- *Antenna Rules Violation* : long wire segments will act as antennna and will accumulate charges, this might damage the connected transistor gates. Solution is to either use bridging or antenna diode insertion to leak away the charges.  

Inside a specific design folder contains a config.tcl which overrides the default settings on OpenLANE. 


<img width="848" height="126" alt="Screenshot 2025-07-24 142941" src="https://github.com/user-attachments/assets/a61e7285-ca8d-45ef-adc8-2522e4e1103e" />



These configurations are specific to a design (e.g. clock period, clock port, verilog files...). 
The priority order for the OpenLANE settings:

1. sky130_xxxxx_config.tcl 
2. config.tcl

## Lab 1 

#### Synthesis

- *Running OpenLANE :*

  - Type docker to Invoke openLane tool (inside``` openlane/```)(```/Desktop/work/tools/openlane_working_dir/openlane```)

  - ```./flow.tcl -interactive``` = run script for automating the whole RTL to GDSII flow but in step by step ```-interactive mode```

  - ```package require openlane 0.9```
 
  <img width="848" height="250" alt="Screenshot 2025-07-24 120932" src="https://github.com/user-attachments/assets/42c4e8a2-9a39-4ac9-9a62-2be252e3fb0f" />
- *Design Setup Stage :*

  ```prep -design picorv32a``` = Setup the filesystem where the OpenLANE tools can dump the outputs. This also creates a ```run/``` folder inside    the specific design directory which contains the command ```log```files, ```results```, and the ```reports``` dump by each tools. These folders    will be empty for now except  for lef files generated by this design setup stage. This merged the cell LEF files ```.lef``` and technology LEF     files ```.tlef ```generating ```merged.nom.lef```   inside ```run/tmp/```
  <img width="848" height="370" alt="Screenshot 2025-07-24 121111" src="https://github.com/user-attachments/assets/d9540d85-6019-4535-8d9a-02e1ee24831c" />


- *Running synthesis :*

  ```run_synthesis``` = Run yosys RTL synthesis, ABC scripts and OpenSTA.
  
  After running synthesis, inside the ```runs/[date]/results/synthesis``` is ```picorv32a_synthesis.v``` which is the mapping of the netlist to      standard cell    library using ABC. The                          ```runs/[date]/reports/synthesis``` will contain synthesis statistic reports and static timing analysis    reports.

  ```Results  and Reports : ```
  
  <img width="848" height="107" alt="Screenshot 2025-07-24 132349" src="https://github.com/user-attachments/assets/a0e2bb41-4784-4395-8c2a-7c3d3f53156a" />
  <img width="848" height="130" alt="Screenshot 2025-07-24 133023" src="https://github.com/user-attachments/assets/78de66bd-0091-43ba-9f40-47aad4db611c" />


#### Estimation of Flip Flop Ratio 

<img width="848" height="600" alt="Screenshot 2025-07-24 123628" src="https://github.com/user-attachments/assets/30b7898a-9a64-4365-9e04-f5027549eead" />

The flipflop ratio is (number of flip flops)/(total number of cells) is 1613/14876 = 0.10843. Or 10.843%

#### Slack 

refers to the difference between the required arrival time and the actual arrival time of a signal. If the signal arrives earlier than required, it's called positive slack, indicating the design meets timing. If it arrives later, it's negative slack, meaning a timing violation has occurred. Positive slack is desirable, while negative slack must be fixed to ensure reliable circuit operation.

In Order to fix negetive slack we change the clock period to ```55.00``` in ```sky130_xxxxx_config.tcl``` 

```before```

<img width="848" height="255" alt="Screenshot 2025-07-24 123323" src="https://github.com/user-attachments/assets/9fe5c38e-a80a-4875-9cfc-4d0b81cf11c3" />


```After```

<img width="848" height="255" alt="Screenshot 2025-07-24 131231" src="https://github.com/user-attachments/assets/ae72bbe7-8a79-40b1-80b4-6af999df93fb" />

# Day 2 
# Good Floorplan vs Bad Floorplan and Introduction to Library Cells

#### Floorplan

- Determine Core and Die Dimensions

  - The core is the central area where logic blocks are placed.

  - Width and height depend on standard cell dimensions in the netlist.

  - Utilization factor = (Area occupied by netlist) / (Total core area).

  - Typically ranges between 0.5 to 0.6 (remaining space is for routing and additional cells).

  - Aspect ratio = (Height / Width) of the core.

  - An aspect ratio of 1 produces a square core.
    <img width="848" height="367" alt="Screenshot 2025-07-24 223107" src="https://github.com/user-attachments/assets/92da74da-1a22-40b1-9b2e-09e5d11c03c7" />

- Preplaced Cells (Macros/IPs)

  - These are complex reusable blocks (e.g., memory, clock-gating cells, comparators).

  - Their placement is user-defined and fixed before automated placement & routing.

  - Automated tools cannot move preplaced cells after definition.
    <img width="848" height="367" alt="Screenshot 2025-07-24 223127" src="https://github.com/user-attachments/assets/675fe99d-0725-4d39-ab18-e5d19f703447" />

- Decoupling Capacitors

  - Placed near preplaced cells to stabilize voltage supply.
    
    <img width="848" height="603" alt="Untitled" src="https://github.com/user-attachments/assets/c04cf8d8-f3de-49f7-bda0-91d5892b8fe5" />


  - Reason: Long power supply wires cause voltage drop (IR drop) due to resistance/inductance.

  - Solution: Decoupling capacitors provide localized current during switching, keeping voltage within noise margin.

- Power Planning

  - Decoupling capacitors alone are insufficient for full-chip power stability.

  - Ground bounce: Excessive current sinking when many cells switch to '0'.

  - Voltage droop: Insufficient current sourcing when many cells switch to '1'.

  - Solution: Use a power mesh with multiple VDD/VSS taps across the chip for uniform current distribution.
    <img width="848" height="371" alt="Screenshot 2025-07-24 222913" src="https://github.com/user-attachments/assets/4c943466-20cc-4e48-96ca-6c0bcdba10ab" />

  

- Pin Placement

  - I/O ports are placed between the core and die boundary.

  - Placement depends on connected cell locations in the core.

  - Clock ports are thicker (low-resistance path) to ensure full-chip drive capability.

  - Logical Cell Placement Blockage

  - Ensures no cells are placed over die pin locations during automated placement/routing.

  - Defined as blockage regions to prevent overlap.
    <img width="848" height="355" alt="Screenshot 2025-07-24 222958" src="https://github.com/user-attachments/assets/aa9bb6ac-70b1-4a1e-a388-22b122db452f" />
    <img width="848" height="347" alt="Screenshot 2025-07-24 223040" src="https://github.com/user-attachments/assets/0f4ab5a9-2147-4725-8bc1-e3412078ac7b" />


#### Placement

- Netlist Binding
  - Netlist binding is the process of mapping the logical representation of a digital design onto standard cell shapes from a library. Each component in the netlist is mapped to a specific shape defined in the       library.

- Initial Placement Design
  In this phase, components from the netlist are placed within the chip's core area. Key considerations include:

  - Proximity to Pins: Components are strategically placed based on their distance from input and output pins to minimize signal delays.

  - Signal Optimization: Signals requiring rapid propagation, such as FF1 to FF2, are placed close together. Buffer cells may be added for signal integrity.

  - Wire-Length and Capacitance Estimation: Wire length and capacitance estimates guide placement optimization, factoring in signal delay, power consumption, and integrity.

- Final Placement Optimization
  - The final placement phase fine-tunes the component layout within the chip, optimizing for performance. It assumes an ideal clock and aims to minimize signal delays, conserve power, and meet design                constraints.

  - The next step in the Digital ASIC design flow after floorplanning is placement. The synthesized netlist has been mapped to standard cells and the floorplanning phase has determined the standard cells rows,       enabling placement.
  
  *OpenLane does placement in two stages:*

  *Global Placement - Optimized but not legal placement. Optimization works to reduce wirelength by reducing half parameter wirelength
  Detailed Placement - Legalizes placement of cells into standard cell rows while adhering to global placement*



## Lab 2 

#### Floor Plan

- Run Floorplan : ```run floor_plan```

- To view our floorplan in Magic we need :

  1. Magic technology file (sky130A.tech)
  2. Def file of floorplan
  3. Merged LEF file

  head over to the following directory to view the results of floorplan using Magic :

  ```cd /Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/<date>/results/floorplan```

  To invoke magic use the command :

  ```magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def &```
  
  <img width="848" height="37" alt="Screenshot 2025-07-24 134349" src="https://github.com/user-attachments/assets/40cb2aba-9fc6-43e8-a196-1216bfb0aa48" />

  *Result* : 

  <img width="848" height="732" alt="Screenshot 2025-07-24 133352" src="https://github.com/user-attachments/assets/b7d0b4e7-4386-4b35-90fe-e37f92b2afa1" />

  To center the view, press "s" to select whole die then press "v" to center the view. Point the cursor to a cell then press "s" to select it, zoom into it by pressing 'z". Type "what" in tkcon to display          information of selected object. These objects might be IO pin, decap cell, or well taps as shown below.

  <img width="848" height="907" alt="Screenshot 2025-07-24 133602" src="https://github.com/user-attachments/assets/c546ebab-bfb0-4631-9967-62575694aa4a" />
  
  if we zoom, we can see that some of the micro, IO pad, and tap-cells have been placed appropriately.

  <img width="848" height="668" alt="Screenshot 2025-07-24 133654" src="https://github.com/user-attachments/assets/770e1f71-f57d-44e4-b710-31d5bbab4fac" />
  
  To get information about the selected object press ```s``` and type ```what``` in console - same as in the above image

#### Place Ment 

- To do a placement in OpenLane : ```run_placement```

- Viewing Placement in Magic :

  ```magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def &```

  <img width="848" height="42" alt="Screenshot 2025-07-24 134409" src="https://github.com/user-attachments/assets/35373e18-0d02-4666-9629-b5e8f4adcdc3" />

  *results* :

  <img width="848" height="728" alt="Screenshot 2025-07-24 134153" src="https://github.com/user-attachments/assets/10d3ff16-2da9-4f14-a763-c834a3e04b00" />

  <img width="848" height="545" alt="Screenshot 2025-07-24 134259" src="https://github.com/user-attachments/assets/dc7bd71a-8625-4cd5-abad-92c53610c52d" />
  
  If we zoom, we the core with all the standard cells placed in between power can ground rail


#### Library Characterization

- Purpose:
  
  - Provides standard cells, macros, decaps for EDA tools.

  - Cells come in different sizes, Vth, drive strengths.

- Inputs (From Foundry PDK):
  
  ✔ DRC/LVS rules (manufacturability)
  
  ✔ SPICE models (transistor behavior)
  
  ✔ User specs (cell height, width, VDD, metal layers)

- Cell Design Flow
  
  - Circuit Design → CDL netlist.

  - Transistor Sizing → Meet speed/power needs.

  - Layout → Stick diagram + Euler’s path (Magic tool).

  - Outputs:

    GDSII (layout)

    LEF (dimensions, pins)

    Extracted SPICE (parasitics)

- Characterization (GUNA Tool):
  
  - Timing:

    Slew (20%-80% rise/fall time).

    Prop Delay (50% input → 50% output).

    Power (leakage, dynamic).

    Noise (crosstalk).

- Key Notes:
  
    ⚠ Negative delay? → Fix threshold (use 50%).
  
   📌 Bigger cells = stronger drive but slower (higher Vth).

  
  <img width="848" height="603" alt="dadda" src="https://github.com/user-attachments/assets/a288d9cd-e1cc-4df1-bdf8-d5c77d97a7c4" />

#### Estimation of area of the die 
- In ```runs/<date>/results/floorplan/picorv32a.floorplan.def``` which is a design exchange format, containing the die area and positions.
  
  <img width="848" height="107" alt="Screenshot 2025-07-24 132349" src="https://github.com/user-attachments/assets/472471ea-1a17-4a3b-8d65-77c46b1f1ffb" />
  

  ```
  DESIGN picorv32a ;
  UNITS DISTANCE MICRONS 1000 ;
  DIEAREA ( 0 0 ) ( 660685 671405 ) ;
  ```

  <img width="848" height="592" alt="Screenshot 2025-07-24 132311" src="https://github.com/user-attachments/assets/2cdf5fe9-7670-4eb5-a1dd-8c9a8d98e253" />

  The die area here is in database units and 1 micron is equivalent to 1000 database units. Thus area of the die is (660685/1000)microns*(671405/1000)microns = 443587 microns squared.


# Day 3
# Design a Library Cell using Magic Layout and Ngspice Characterization

## Designing a Library Cell :

 **SPICE Deck for CMOS Inverter**  

  - **Netlist** describing component connectivity.  
  - **W/L** values:  
    - `0.375u/0.25u` = Width=375nm, Length=250nm *(PMOS typically 2-3x wider than NMOS)*.  
  - **Voltages**:  
    - Gate supply (`Vin`): 2.5V *(scaled with length)*.  
    - `Vdd`: 2.5V (PMOS source).  
    - `0`: Ground (NMOS source).  
  - **Capacitances**:  
    - Load caps: `10ff` (attached to nodes).  
  - **Nodes**:  
    - Label connections (e.g., `Vin`, `Vdd`, `0`) for SPICE simulation.
      
 **Description** : 
 
 <img width="1387" height="602" alt="Screenshot 2025-07-26 125422" src="https://github.com/user-attachments/assets/f647f49c-e4af-4a20-9eaf-dfa6f4d9f203" />

 ```
 Syntax for the PMOS and NMOS descriptiom:
 [component name] [drain] [gate] [source] [substrate] [transistor type] W=[width] L=[length]
 
 All components are described based on nodes and its values
 .op is the start of SPICE simulation operation where Vin will be sweep from 0 to 2.5 with 0.5 steps
 tsmc_025um_model.mod is the model file containing the technological parameters for the 0.25um NMOS and PMOS The steps to simulate in SPICE:
 ```

  **Key Points**:  
  - PMOS wider → balances current with NMOS.  
  - Node naming essential for simulation.  
  - Example: `M1` (PMOS) and `M2` (NMOS) form inverter.  

  
  
#### Follow these steps for simulation in ngspice :
 
  1. Source the circuit file in Ngspice using source <file_name>.cir.
  2. Execute the simulations using the run command.
  3. Use setplot to prepare for plotting.
  4. For DC analysis (as indicated in the .cir file), use dc1 to prepare the DC plot.
  5. Display available vectors using display.
  6. Plot specific vectors, e.g., plot vout vs vin, to visualize the circuit behavior.



  






