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
    - [Designing a Library Cell](#Designing-a-Library-Cell)
    - [steps for simulation in ngspice](#Follow-these-steps-for-simulation-in-ngspice)
    - [SPICE Switching Threshold and Propagation Delay](#SPICE-Switching-Threshold-and-Propagation-Delay)
    - [16-Mask CMOS Process](#16-Mask-CMOS-Process)
  - [Lab](#Lab-3)
    - [Spice extraction of inverter in magic](#Spice-extraction-of-inverter-in-magic)
    - [Post-Layout Spice simulation (ngspice)](#Post-Layout-Spice-simulation-(ngspice))
    - [Slew rate and Propagation delay](#Slew-rate-and-Propagation-delay)
    - [Fix Tech File DRC via Magic](#Fix-Tech-File-DRC-via-Magic)
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

#### Designing a Library Cell 

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
 
 <img width="848" height="602" alt="Screenshot 2025-07-26 125422" src="https://github.com/user-attachments/assets/f647f49c-e4af-4a20-9eaf-dfa6f4d9f203" />

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

*Vout vs Vin Plot :*

<img width="848" height="656" alt="Screenshot 2025-07-26 125706" src="https://github.com/user-attachments/assets/4e34ffd1-4c48-4ab0-b0fd-cba749e4df5a" />

  
<img width="848" height="648" alt="Screenshot 2025-07-26 125749" src="https://github.com/user-attachments/assets/a225d17d-3e87-4ea4-a6d9-4e5caf95988f" />


- From the above we can see that the switching threshold of the latter is exactly midway with reference to Vdd and is slightly shifted to the left with the former

- At the Switching threshold pmos and nmos drain add up to zero. Using this condition and the Drain Current equation we can fix a value for W/L to obtain the required switching voltage

- To obtain a symmetric DC plot, you can scale the aspect ratio of PMOS by 2.5 times

#### SPICE Switching Threshold and Propagation Delay 

- Switching threshold = Vin is equal to Vout. This the point where both PMOS and NMOS is in saturation or kind of turned on, and leakage current is high. 
                      If PMOS is thicker than NMOS, the CMOS will have higher switching threshold (1.2V vs 1V) while threshold will be lower when NMOS becomes thicker.

- Propagation delay = rise or fall delay


```the line intersection is the switching threshold```

<img width="848" height="541" alt="Screenshot 2025-07-26 135748" src="https://github.com/user-attachments/assets/d3abddf5-db6a-448f-952e-b885b8568468" />


```transient analysis is used for finding propagation delay :```

<img width="200" height="216" alt="Screenshot 2025-07-26 135659" src="https://github.com/user-attachments/assets/7ab2839f-d176-49b6-b1b5-30f36c04c5e2" />

```
Vin in 0 0 pulse 0 2.5 0 10p 10p 1n 2n 
*** Simulation Command ***
.op
.tran 10p 4n
```
<img width="848" height="53" alt="Screenshot 2025-07-26 135650" src="https://github.com/user-attachments/assets/957730e3-5601-4bbc-a6d9-46503961e79a" />

`starts at 0V`
`ends at 2.5V`
`starts at time 0`
`rise time of 10ps`
`fall time of 10ps`
`pulse-width of 1ns`
`period of 2ns`

**SPICE transient analysis :**

<img width="848" height="889" alt="187056370-18949899-a158-4307-96d9-d5c06bbeed66" src="https://github.com/user-attachments/assets/754e0757-5b33-494c-95ae-2a287599dc6d" />

#### 16-Mask CMOS Process 
*inverter*

1. Substrate Selection
   - P-type Si wafer (5–50 Ω·cm, ⟨100⟩ orientation).

   - Key: Substrate doping < well doping.

2. Active Region Isolation (LOCOS)
   - Mask 1: Photoresist patterning to protect transistor regions.

   - Layers:

     - Si₃N₄ (80nm): Blocks SiO₂ growth.

     - Field Oxide (LOCOS): Grows in unprotected areas (~1µm).

   - Result: Isolated active regions for NMOS/PMOS.

3. Well Formation
   - N-well (PMOS):

     - Mask 2: Protects NMOS region.

     - Phosphorus implant @400 keV.

   - P-well (NMOS):

     - Mask 3: Protects PMOS region.

     - Boron implant @200 keV.

     - Drive-in diffusion to deepen wells.

4. Gate Formation
   - Threshold Voltage Control:

     - Mask 4/5: Boron (NMOS) & Arsenic (PMOS) implants.

     - Gate Oxide: Etch/regrow 10nm SiO₂ (high quality).

     - Mask 6: Poly-Si gate patterning.

5. Lightly Doped Drain (LDD)
   - Purpose: Prevent hot-electron/short-channel effects.

   - Mask 7: N⁻ implant (Phosphorus) for NMOS.

   - Mask 8: P⁻ implant (Boron) for PMOS.

   - Sidewall spacers: SiO₂ deposition + anisotropic etch.

6. Source/Drain Formation
   - Mask 9: N⁺ implant (Arsenic) for NMOS.

   - Mask 10: P⁺ implant (Boron) for PMOS.

   - Screen oxide prevents channeling.

7. Contacts & Local Interconnects
   - TiN/TiSi₂ Formation:

     - Ti sputtering → RTA (600–700°C) → TiSi₂ (gates) + TiN (routing).

   - Mask 11: Etch TiN for first-layer contacts.

8. Metallization (Al/W)
   - Planarization: PSG deposition + CMP.

   - Contact Holes:

     - Mask 12/14: Via etching.

     - Mask 13/15: Aluminum/W deposition.

   - Mask 16: Top-layer contact/pad opening.

*Key Insights :*
- PMOS Width > NMOS (2–3x) for current balancing.

- LDD reduces leakage; Sidewall spacers protect LDD during S/D implants.

- TiSi₂ lowers gate resistance; TiN aids local routing.

- 16 masks cover wells, implants, gates, contacts, and metals.

Final Result : 

<img width="848" height="400" alt="Screenshot 2025-07-26 174207" src="https://github.com/user-attachments/assets/fa5ac8b5-675a-4151-a230-072c64d6137e" />


## Lab 3 

#### Spice extraction of inverter in magic 

- Clone vsdstdcelldesign. Copy the techfile ```sky130A.tech``` from ```pdks/sky130A/libs.tech/magic/``` to directory of the cloned repo. 
  <img width="848" height="52" alt="Screenshot 2025-07-26 174607" src="https://github.com/user-attachments/assets/b185ee77-7baf-4d9d-a14e-8f3ce328d139" />

- View the mag file using magic ```magic -T sky130A.tech sky130_inv.mag &```
  <img width="848" height="57" alt="Screenshot 2025-07-26 174719" src="https://github.com/user-attachments/assets/144921cd-5e35-4c8b-9918-89460b008333" />
  
- Make an extract file ```.ext``` by typing extract all in the tkon terminal of magic. 

- Extract the ```.spice``` file from this ext file by typing ```ext2spice cthresh 0 rthresh 0``` then ```ext2spice``` in the tkon terminal.

  <img width="848" height="260" alt="Screenshot 2025-07-26 174858" src="https://github.com/user-attachments/assets/76fb85c3-e007-4032-9925-8800b238c3e0" />
  
#### Post-Layout Spice simulation (ngspice)

- Open the spice file by typing ```ngspice sky130A_inv.spice``` and Generate a graph using plot y vs time a
  
  <img width="848" height="875" alt="Screenshot 2025-07-26 181256" src="https://github.com/user-attachments/assets/6b11a52d-e7dd-4626-882e-e8d24f331810" />

  <img width="848" height="487" alt="Screenshot 2025-07-26 181156" src="https://github.com/user-attachments/assets/fa7d1c08-da3b-4118-83f7-42ff78559790" />

  
#### Slew rate and Propagation delay 

- Rise Transition ```output transition time from 20%(0.66V) to 80%(2.64V)``` : 0.03581ns
  ```(2.156 - 2.12019)```
  <img width="848" height="78" alt="Screenshot 2025-07-26 213748" src="https://github.com/user-attachments/assets/02b52e5c-fff7-4283-8258-e47d702eabac" />

- Fall Transition ```output transition time from 80%(2.64V) to 20%(0.66V)``` : 0.5969ns
  <img width="848" height="77" alt="Screenshot 2025-07-26 214030" src="https://github.com/user-attachments/assets/b9a32895-a01c-4740-9f63-48f32acf497e" />

- Rise delay ```delay between 50%(1.65V) of input to 50%(1.65V) of output``` : 0.0303ns
  <img width="848" height="72" alt="Screenshot 2025-07-26 214545" src="https://github.com/user-attachments/assets/158e2eaf-4f69-4fbd-82bb-5b2c5814962d" />

- Fall Delay ```delay between 50%(1.65V) of input to 50%(1.65V) of output``` : 0.0483ns

#### Fix Tech File DRC via Magic


All technology-specific information comes from a technology file. This file includes such information as layer types used, electrical connectivity between types,     design rules, rules for mask generation, and rules for extracting netlists for circuit simulation - [DRC rules for SKY130nm PDK](https://skywater-pdk.readthedocs.io/en/main/rules/periphery.html#rules-periphery--page-root)

Clone custom inverter standard cell design :

```
# Change directory to openlane
cd Desktop/work/tools/openlane_working_dir/openlane

# Clone the repository with custom inverter design
git clone https://github.com/nickson-jose/vsdstdcelldesign

# Change into repository directory
cd vsdstdcelldesign
 
# Copy magic tech file to the repo directory for easy access
cp /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech .

# Check contents whether everything is present
ls

# Command to open custom inverter layout in magic
magic -T sky130A.tech sky130_inv.mag &
```

- Inside the drc_tests/ are the .mag layout files and the sky130A.tech.
- Open magic with ```poly.mag``` as input: magic poly.mag from file --> open
- Focus on Incorrect poly.9.the spacing between polyresistor with poly or diff/tap must at least be 0.480um. Using command box in console, we can see that   the distance violated but there is no DRC violations shown. Our goal is to fix the tech file to include that DRC.
<img width="848" height="63" alt="Screenshot 2025-07-26 232040" src="https://github.com/user-attachments/assets/102cbe43-321e-451d-8e0e-82f87013bd23" />


<img width="848" height="472" alt="Screenshot 2025-07-26 221828" src="https://github.com/user-attachments/assets/48b3df34-f795-4d5b-9c47-09462b66b709" />
<img width="848" height="532" alt="Screenshot 2025-07-26 223639" src="https://github.com/user-attachments/assets/d9732a2e-a38e-49a6-9374-ab0a0183dd06" />
<img width="400" height="137" alt="Screenshot 2025-07-26 232204" src="https://github.com/user-attachments/assets/d39c2834-fb34-45d8-8f2f-93e4ffb25187" />

The current sky130A.tech file only includes spacing rules for:

- n-poly resistor to n-diffusion

- p-poly resistor to p-diffusion

We are now adding two new critical spacing rules (highlighted in green):

- Left rule: Spacing between n-poly resistor and regular poly (non-resistor)

- Right rule: Spacing between p-poly resistor and regular poly (non-resistor)

These additions ensure proper isolation between resistor and non-resistor poly layers in the design

<img width="400" height="193" alt="Screenshot 2025-07-26 223705" src="https://github.com/user-attachments/assets/db2ff8f4-d82f-454b-bf15-88568d65e412" />
<img width="400" height="151" alt="Screenshot 2025-07-26 223718" src="https://github.com/user-attachments/assets/e36aa647-54e9-460a-bd63-e89fcb35c495" />


To apply the new poly resistor spacing rules:

Run ```tech load sky130A.tech``` to update the rules.

Use ```drc check``` – violations appear as white dots on poly layers.

Inspect each violation with ```drc find```

New rules cover:

n-poly ↔ poly spacing

p-poly ↔ poly spacing

<img width="400" height="87" alt="Screenshot 2025-07-26 223505" src="https://github.com/user-attachments/assets/a6ec936c-e03e-440c-9729-59cd987577ab" />



  


















