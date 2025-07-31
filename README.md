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
      
- [Day 3](#DAY-3)
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
      
- [DAY 4](#DAY-4)
  - [Theory](#DAY-4)
    - [Delay Table](#Delay-Table)
    - [Timing Analysis (using Ideal Clocks)](#Timing-Analysis-(using-Ideal-Clocks))
    - [Clock Tree Synthesis Stage](#Clock-Tree-Synthesis-Stage)
  - [Lab](#Lab-4)
    
- [DAY 5](#DAY-5)
  - [Lab](#Lab-5)
  


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
  
  <img width="848" height="900" alt="Screenshot 2025-07-24 132349" src="https://github.com/user-attachments/assets/a0e2bb41-4784-4395-8c2a-7c3d3f53156a" />
  <img width="848" height="900" alt="Screenshot 2025-07-24 133023" src="https://github.com/user-attachments/assets/78de66bd-0091-43ba-9f40-47aad4db611c" />


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
  
  <img width="848" height="400" alt="Screenshot 2025-07-24 134349" src="https://github.com/user-attachments/assets/40cb2aba-9fc6-43e8-a196-1216bfb0aa48" />

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

  <img width="848" height="400" alt="Screenshot 2025-07-24 134409" src="https://github.com/user-attachments/assets/35373e18-0d02-4666-9629-b5e8f4adcdc3" />

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
  
  <img width="848" height="400" alt="Screenshot 2025-07-24 132349" src="https://github.com/user-attachments/assets/472471ea-1a17-4a3b-8d65-77c46b1f1ffb" />


  ```
  DESIGN picorv32a ;
  UNITS DISTANCE MICRONS 1000 ;
  DIEAREA ( 0 0 ) ( 660685 671405 ) ;
  ```

  <img width="848" height="700" alt="Screenshot 2025-07-24 132311" src="https://github.com/user-attachments/assets/2cdf5fe9-7670-4eb5-a1dd-8c9a8d98e253" />

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
  <img width="848" height="168" alt="Screenshot 2025-07-30 125827" src="https://github.com/user-attachments/assets/73cde73d-b8ce-40b6-b346-73d279bccbc6" />

- View the mag file using magic ```magic -T sky130A.tech sky130_inv.mag &```
  <img width="848" height="57" alt="Screenshot 2025-07-26 174719" src="https://github.com/user-attachments/assets/144921cd-5e35-4c8b-9918-89460b008333" />
  
- Make an extract file ```.ext``` by typing extract all in the tkon terminal of magic. 

- Extract the ```.spice``` file from this ext file by typing ```ext2spice cthresh 0 rthresh 0``` then ```ext2spice``` in the tkon terminal.

  <img width="848" height="260" alt="Screenshot 2025-07-26 174858" src="https://github.com/user-attachments/assets/76fb85c3-e007-4032-9925-8800b238c3e0" />

  <img width="848" height="721" alt="Screenshot 2025-07-26 172928" src="https://github.com/user-attachments/assets/9d946e89-dce5-46dd-be89-e8f93b824935" />

  
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

# DAY 4 
# Pre-layout Timing Analysis and Importance of Good Clock Tree 

## Theory

#### Delay Table

To minimize clock skew between endpoints of a clock tree (preventing signal arrival at different times):

- Equal capacitive load: Buffers at the same level must drive identical capacitive loads to ensure uniform timing delay (latency).

- Uniform buffer sizing: Buffers on the same level must be the same size, as differing sizes lead to varying W/L ratios, resistances, and RC constants—resulting in    inconsistent delays.

To minimize clock skew in a clock tree distribution network, buffers at the same hierarchical level must maintain identical capacitive loads and uniform buffer sizes to ensure consistent propagation delays across all branches. While different levels may employ varying buffer sizes and drive different capacitive loads, the symmetric matching of these characteristics within each level guarantees balanced cumulative delays along all clock paths, effectively eliminating skew. The timing behavior of these buffer cells is characterized through delay tables in liberty files, where the primary delay component - output slew - is determined by both the output capacitive load and input transition time. This input transition time itself depends on the preceding buffer's output characteristics and its own transition delay properties, creating a cascaded timing relationship that must be carefully modeled for accurate skew prediction and control.

<img width="1166" height="635" alt="Screenshot 2025-07-30 223725" src="https://github.com/user-attachments/assets/15e6a690-8f95-41c5-8027-a0e04d836538" />

#### Timing Analysis (using Ideal Clocks)

During pre-layout static timing analysis (STA), the verification is performed using ideal clock models that do not yet account for the physical implementation effects. At this stage, the analysis excludes both clock buffer delays and net delays caused by RC parasitics, as the complete routing information is not yet available. Instead, wire delay estimates are derived from the process design kit (PDK) library wire models, which provide approximate interconnect characteristics based on technology parameters. This idealized approach enables early timing verification before physical implementation, but requires subsequent post-layout STA to validate timing with actual buffer insertion and extracted parasitic RC values from the final layout.

#### Clock Tree Synthesis Stage

When constructing a clock tree, three critical parameters must be addressed: (1) Clock skew minimization requires balanced clock tree structures that ensure equal wire lengths and matching delays to all endpoints, guaranteeing synchronous signal arrival. (2) Clock signal integrity demands careful slew rate control through specialized clock buffers that maintain symmetrical rise/fall times, compensating for RC degradation along interconnect routes. (3) Crosstalk prevention necessitates strategic shielding of clock nets, typically achieved by routing power (VDD) or ground lines adjacent to clock signals to break parasitic coupling capacitances with neighboring aggressor nets; this shielding methodology may also be applied to other timing-critical signal paths. Together, these techniques ensure robust clock distribution with minimal timing variation, proper signal quality, and reduced noise coupling throughout the clock network.

<img width="676" height="457" alt="Screenshot 2025-07-30 224543" src="https://github.com/user-attachments/assets/765ab7fb-20ea-40a2-a724-032cb203e7ad" />


## Lab 4

- Tracks.info used in routing stage (`/desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/openlane/sky130_fd_sc_hd`)

  <img width="848" height="496" alt="Screenshot 2025-07-30 000535" src="https://github.com/user-attachments/assets/5b0f8ca6-d1d9-4205-afaa-3d4e0c39272e" />

  <img width="848" height="490" alt="Screenshot 2025-07-30 000549" src="https://github.com/user-attachments/assets/bbf1a286-b4cd-44bf-a572-249b2c87fe0e" />

  <img width="848" height="800" alt="Screenshot 2025-07-30 113158" src="https://github.com/user-attachments/assets/b6e985f8-a29b-477c-806a-c1da7bda2ab1" />

  
- Type the Command in tkcon window to set grid as tracks of locali layer
   ```grid 0.46um 0.34um 0.23um 0.17um```

  <img width="848" height="848" alt="Screenshot 2025-07-28 121258" src="https://github.com/user-attachments/assets/cc10810e-5776-436d-a05d-44b4b84273b9" />

  <img width="848" height="608" alt="Screenshot 2025-07-27 011229" src="https://github.com/user-attachments/assets/5c5bc380-b802-41c4-b1a4-0f610579fcf9" />

- Two Things to verify : Pins lies on intersections and cell width is 3. We can make use of grids to identify cell width.

- Extracting `LEF` file

  - open the mag file 
    ```magic -T sky130A.tech sky130_inv.mag &```
  - save the inverter by your custom name save `sky130_vsdinv.mag`
    then type `lef write` in tkcon window. This will create files as shown below.
    
    <img width="848" height="146" alt="Screenshot 2025-07-27 015605" src="https://github.com/user-attachments/assets/ae0c769f-27b8-4342-84ce-398afc2b430d" />
    <img width="848" height="145" alt="Screenshot 2025-07-30 130455" src="https://github.com/user-attachments/assets/a75274fc-1505-4900-a4a6-2f0cd839281c" />
    

- Pluging-in Custom Inverter Cell into Openlane

  - Copy the LEF file `sky130_vsdinv.lef` and `sky130_fd_sc_hd__*` from `openlane/vsdstdcelldesign/libs` to `picorv32a/src` directory.
 
    <img width="848" height="292" alt="Screenshot 2025-07-27 112459" src="https://github.com/user-attachments/assets/cd356fb9-25aa-4665-b87b-1e17dc3b5659" />

    <img width="848" height="267" alt="Screenshot 2025-07-30 113419" src="https://github.com/user-attachments/assets/5022ddbe-00f2-4f35-babc-350cc474992e" />

 
  - add these commands into `config.tcl` in `Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a`
    ```
    set ::env(LIB_SYNTH) "$::env(OPENLANE_ROOT)/designs/picorv32a/scr/sly130_fd_sc_hd__typical.lib"
    set ::env(LIB_FASTEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/scr/sly130_fd_sc_hd__fast.lib"
    set ::env(LIB_SLOWEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/scr/sly130_fd_sc_hd__slow.lib"
    set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/scr/sly130_fd_sc_hd__typical.lib"

    set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]
    ```
    
    <img width="848" height="582" alt="Screenshot 2025-07-30 004441" src="https://github.com/user-attachments/assets/db94ac64-9f76-400c-83bc-161d92eb155d" />

  - Run docker and prepare the design picorv32a. You may make use of overwrite command : `prep -design picorv32a -tag <date> -overwrite`
 
    <img width="848" height="902" alt="Screenshot 2025-07-30 131638" src="https://github.com/user-attachments/assets/516d4ac8-968c-4285-9784-cfb003c45fed" />


  - setting lefs

    <img width="1697" height="892" alt="Screenshot 2025-07-30 132028" src="https://github.com/user-attachments/assets/647d1896-6a85-48f4-92e6-a4985aa4c76c" />

 
    ```
    set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
    add_lefs -src $lefs
    ```
    
  - running synthesis ```run_synthesis```
 
    <img width="848" height="481" alt="Screenshot 2025-07-28 130724" src="https://github.com/user-attachments/assets/bd6f4d92-62b0-427f-badd-2cc9968b03a4" />
 
    <img width="848" height="387" alt="Screenshot 2025-07-30 115338" src="https://github.com/user-attachments/assets/7a3485fb-6d89-40cc-84f6-17800e613640" />
 
    <img width="848" height="897" alt="Screenshot 2025-07-28 222817" src="https://github.com/user-attachments/assets/84f09731-1370-4d3c-b9c8-409273ba04f7" />
 
    we get to see --> chip area, wns (worst timing violation) and tns (total negative slack)

    merged.lef in tmp directory with our custom inverter as macro
    
    <img width="848" height="497" alt="Screenshot 2025-07-28 134549" src="https://github.com/user-attachments/assets/c044a361-b554-41d6-a30f-f4d218789c12" />


- we change some variable to get better timing improvement. This results change in chip area.so we'll change some variables because Those timing values aren't          ideal.

    ```
    echo $::env(SYNTH_STRATEGY)
    
    set ::env(SYNTH_STRATEGY) "DELAY 3"

    echo $::env(SYNTH_BUFFERING)
    // make sure its enabled i.e 1

    echo $::env(SYNTH_SIZING)
   
    set ::env(SYNTH_SIZING) 1

    echo $::env(SYNTH_DRIVING_CELL
    // check whether it's the proper cell or not

    run_synthesis
    
    ```

    <img width="848" height="921" alt="Screenshot 2025-07-28 223452" src="https://github.com/user-attachments/assets/354f77d9-484b-4fc9-82db-f98ba83e714f" />

    <img width="848" height="905" alt="Screenshot 2025-07-28 225050" src="https://github.com/user-attachments/assets/8fd2aa54-a14f-4a08-9c63-2e0da79a209a" />

    values area has increased and worst negative slack has become 0

- run floorplan

  <img width="1847" height="883" alt="Screenshot 2025-07-28 225444" src="https://github.com/user-attachments/assets/9c38515d-dcd4-43cb-93d8-db4e18e8d09e" />

  we get error

  <img width="1847" height="883" alt="Screenshot 2025-07-28 225444" src="https://github.com/user-attachments/assets/2e5e7292-a69f-4646-9e8a-3f35f27d626a" />

  run these commands instead :

  ```
  init_floorplan
  place_io
  tap_decap_or
  ```

- run placement and we obtain `DEF` file
  opening the file using magic by command --> `magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read        ../../tmp/merged.lef def read picorv32a.placement.def &`

  <img width="1832" height="245" alt="Screenshot 2025-07-30 124909" src="https://github.com/user-attachments/assets/cd4713c1-b98b-4fcd-b906-30b1b344b2a8" />

  <img width="1438" height="740" alt="Screenshot 2025-07-30 125132" src="https://github.com/user-attachments/assets/db8a2104-c693-4c8c-ae57-b77b073c9139" />

  <img width="1441" height="738" alt="Screenshot 2025-07-29 010515" src="https://github.com/user-attachments/assets/3d0ac85a-1153-45a3-aff5-97221c0888b9" />

  <img width="1505" height="782" alt="Screenshot 2025-07-29 010553" src="https://github.com/user-attachments/assets/9461a51c-4038-4e8f-ad4a-088adf6a8967" />

- Post-Synthesis timing analysis

  Since we are having improved timing of 0 wn. so what we will do is we will do timing analysis on Synthesis that has lot of violations
  - we gradually improve and reduce our slack by replacing cells with better one and to deliver improved delay.
  This is **timing ECO fixes**

  First we do is running the synthesis --> include newly added lef to openlane flow --> set ::env(SYNTH_SIZING) 1 --> set                ::env(SYNTH_MAX_FANOUT) 4 --> run_synthesis

  <img width="1848" height="902" alt="Screenshot 2025-07-31 191742" src="https://github.com/user-attachments/assets/3d42043d-4758-4577-ae4b-c14846e44257" />
  

  Later in `cd Desktop/work/tools/openlane_working_dir/openlane` we write command `sta pre_sta.conf`

  *sta pre_sta.conf*
  <img width="1838" height="901" alt="Screenshot 2025-07-31 211105" src="https://github.com/user-attachments/assets/ee021675-c15d-46ad-b587-43b0a22ccf3f" />

  *my_base.sdc*
  <img width="1832" height="891" alt="Screenshot 2025-07-31 211155" src="https://github.com/user-attachments/assets/2c6e2506-2d8f-4049-ad3a-d95f4fafeec8" />

  <img width="1433" height="756" alt="Screenshot 2025-07-31 191954" src="https://github.com/user-attachments/assets/0893743c-bb30-450a-9620-e1d8b62fa90b" />

  <img width="1817" height="908" alt="Screenshot 2025-07-31 192011" src="https://github.com/user-attachments/assets/b17625a1-252e-4413-befd-2b22a3379632" />

  Replacing some cells to reduce slack

  <img width="1855" height="437" alt="Screenshot 2025-07-31 202908" src="https://github.com/user-attachments/assets/73c52399-6aae-4cce-bd09-1e93e5eadfda" />

  report_net -connections _11672_ [we see the driver pins] 
  <img width="1857" height="910" alt="Screenshot 2025-07-31 203237" src="https://github.com/user-attachments/assets/5f8240b5-57a3-4f99-84f6-3ab736b991f4" />

  replace_cell _14510_ sky130_fd_sc_hd__or3_4 [we replace some cells with better suited ones for driving 4 fanouts]
  <img width="1847" height="921" alt="Screenshot 2025-07-31 203516" src="https://github.com/user-attachments/assets/affa5a5a-3594-4d79-9f75-0cf3bf84f6e7" />

  To check the report and see the changes in slack we use command : report_checks -fields {net cap slew input_pins} -digits 4

  Slack reduced

  <img width="1841" height="905" alt="Screenshot 2025-07-31 203603" src="https://github.com/user-attachments/assets/38e44c1a-927a-47ec-8eec-e10d63772ddc" />

  report_checks -from _29052_ -to _30440_ -through _14510_
  <img width="1842" height="912" alt="Screenshot 2025-07-31 204342" src="https://github.com/user-attachments/assets/747b907d-20f5-44fa-9665-31cadfd64629" />

  <img width="1852" height="916" alt="Screenshot 2025-07-31 204533" src="https://github.com/user-attachments/assets/5e3f3cc8-b68e-4948-909c-57ace47adec3" />

  report_net -connections _11668_
  
  replace_cell _14506_ sky130_fd_sc_hd__or4_4

  <img width="1855" height="645" alt="Screenshot 2025-07-31 205421" src="https://github.com/user-attachments/assets/bb689f1d-315f-476a-8b97-14916f3b0c5e" />

  Slack Reduced

  <img width="1853" height="880" alt="Screenshot 2025-07-31 205436" src="https://github.com/user-attachments/assets/6610dbe3-72fb-4f44-ac34-efcfef1c67b7" />

  Intially : -23.90
  
  now : -22.9860

  <img width="397" height="667" alt="Screenshot 2025-07-31 212541" src="https://github.com/user-attachments/assets/e52a36e4-8a81-4ed7-bdac-31ecddd4e35e" />

  - We have reduced around 0.914 ns of violation
 
  - iterative process

- Run CTS
  
  - Here we proceed with earlier 0 violation design. we want to proceed with the clean design to further stage.
  - run synthesis,floorplan,placement and then run cts (`run_cts`)

  <img width="1297" height="457" alt="Screenshot 2025-07-29 181220" src="https://github.com/user-attachments/assets/1cb63874-68e5-4fba-a84f-a27dcec9eb0b" />

  <img width="1847" height="272" alt="Screenshot 2025-07-29 181247" src="https://github.com/user-attachments/assets/7573ef66-934f-45e7-8193-366e2c699ff4" />
  

- Post-CTS Timing analysis : OpenROAD

  command : `openroad`

  <img width="1257" height="480" alt="Screenshot 2025-07-29 181627" src="https://github.com/user-attachments/assets/4123c22c-b05e-457c-a5f0-804acaad7fc8" />

  <img width="1841" height="620" alt="Screenshot 2025-07-29 181808" src="https://github.com/user-attachments/assets/7d40a9ad-701b-4e1f-8449-5c060cdf403c" />

  <img width="1847" height="908" alt="Screenshot 2025-07-29 181832" src="https://github.com/user-attachments/assets/2f8ed163-c532-491e-906b-cad0c448fc34" />

  <img width="1830" height="878" alt="Screenshot 2025-07-29 181916" src="https://github.com/user-attachments/assets/1e754cbe-4c97-479c-bcce-e9beb3ce3154" />

  removing 'sky130_fd_sc_hd__clkbuf_1 and running CTS again 

  <img width="1852" height="907" alt="Screenshot 2025-07-29 182340" src="https://github.com/user-attachments/assets/e4d7ed6d-9bc7-4d7a-975e-4a574397192c" />

  <img width="1518" height="655" alt="Screenshot 2025-07-30 222049" src="https://github.com/user-attachments/assets/3a085a95-c2b1-493c-8b05-0ca7bbe19744" />
  
  echo $::env(CTS_CLK_BUFFER_LIST) [Checking current value of CTS_CLK_BUFFER_LIST]

  <img width="1518" height="655" alt="Screenshot 2025-07-30 222049" src="https://github.com/user-attachments/assets/3a085a95-c2b1-493c-8b05-0ca7bbe19744" />

  running openROAD command again
  
  ```
  openroad

  read_lef /openLANE_flow/designs/picorv32a/runs/<date>/tmp/merged.lef

  read_def /openLANE_flow/designs/picorv32a/runs/<date>/results/cts/picorv32a.cts.def

  write_db pico_cts1.db

  Loading the created database in OpenROAD
  read_db pico_cts.db

  read_verilog /openLANE_flow/designs/picorv32a/runs/<date>/results/synthesis/picorv32a.synthesis_cts.v

  read_liberty $::env(LIB_SYNTH_COMPLETE)

  link_design picorv32a

  read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc

  set_propagated_clock [all_clocks]

  report_checks -path_delay min_max -fields {slew trans net cap input_pins} -format full_clock_expanded -digits 4

  report_clock_skew -hold

  report_clock_skew -setup
  ```
  
  <img width="1848" height="916" alt="Screenshot 2025-07-29 183123" src="https://github.com/user-attachments/assets/d270f744-9aec-439a-b537-3e5fb5a04a9a" />

  <img width="1337" height="740" alt="Screenshot 2025-07-30 220638" src="https://github.com/user-attachments/assets/f89677be-3220-45dc-8868-1bde680a25e1" />

  Checking clock skew for setup and hold
  <img width="1852" height="945" alt="Screenshot 2025-07-29 183410" src="https://github.com/user-attachments/assets/5b3b741f-fc9e-40d4-9720-3ff3ae5cf279" />



# DAY 5 
# Final steps for RTL2GDS using tritonRoute and openSTA

## Lab 5 

- Performing (PDN) Power Distribution Network

  - In docker open openlane andd perform prepare the design and include newly added `lef` to openlane flow.
  - set SYNTH_STRATEGY --> DELAY 3
  - set SYNTH_SIZING --> 1
  - run synthesis, floorplan, placement and finally cts
  - after cts is done we do power distribution network PDN
  - gen_pdn

  <img width="1845" height="898" alt="Screenshot 2025-07-29 183947" src="https://github.com/user-attachments/assets/0623ac22-6ec7-4ea5-9456-c67c1de65eab" />

  <img width="1852" height="906" alt="Screenshot 2025-07-29 184133" src="https://github.com/user-attachments/assets/944c3ba9-5daf-4bd0-8a20-96b47da0d1c6" />

  <img width="1810" height="887" alt="Screenshot 2025-07-29 184343" src="https://github.com/user-attachments/assets/23dc0a50-d4e4-4b89-8e1e-8fab538c8f30" />

  <img width="1847" height="902" alt="Screenshot 2025-07-29 184540" src="https://github.com/user-attachments/assets/f948ace1-3fe0-488a-9a35-a88aca1b8b36" />

  <img width="1856" height="898" alt="Screenshot 2025-07-29 184602" src="https://github.com/user-attachments/assets/4d1d219b-4dc2-4011-9633-dfe76a9c24be" />

  - In `<date>/tmp/floorplan/` directory we  have the pdn
    to open we use command : `magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef      def read 14-pdn.def &`

  <img width="1606" height="818" alt="Screenshot 2025-07-29 184757" src="https://github.com/user-attachments/assets/d80cc966-ecb3-4adc-a923-1d4fe65a3cc7" />

  <img width="1502" height="838" alt="Screenshot 2025-07-29 184832" src="https://github.com/user-attachments/assets/8979c7cf-74b0-4249-b8ba-4a8595d78207" />

  <img width="1506" height="837" alt="Screenshot 2025-07-29 184913" src="https://github.com/user-attachments/assets/9ecab539-9937-4f37-92ee-a3ead193d6ac" />

- Perfrom detailed routing using TritonRoute

  - run routing using command --> `run_routing`
 
    <img width="1852" height="903" alt="Screenshot 2025-07-29 185022" src="https://github.com/user-attachments/assets/1c33b709-93b7-4a08-81d0-c7b58e537e81" />

    <img width="1853" height="505" alt="Screenshot 2025-07-30 175231" src="https://github.com/user-attachments/assets/1a0038ba-6100-4473-b3db-b90d596c6e27" />

    <img width="1852" height="918" alt="routing" src="https://github.com/user-attachments/assets/26d7286a-5e80-4040-8960-0c9006bc4a71" />
  - go to `<date>/results/routing/` directory
 
    load the def file : `magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def      read picorv32a.def &`

    <img width="1841" height="912" alt="Screenshot 2025-07-29 190229" src="https://github.com/user-attachments/assets/2e960fa2-3caf-4d39-99e6-bfb2747c5a95" />

    <img width="1748" height="890" alt="Screenshot 2025-07-29 190359" src="https://github.com/user-attachments/assets/d3cc2791-c395-4fb3-bd54-f2905c7f5c99" />

    <img width="1742" height="895" alt="Screenshot 2025-07-29 190416" src="https://github.com/user-attachments/assets/c8b8a5fd-7a7c-4a43-bdc7-a9378b4e07b9" />

    <img width="1688" height="880" alt="Screenshot 2025-07-29 190436" src="https://github.com/user-attachments/assets/db974f00-d023-45df-acfe-24e865549b76" />

    fast route guide inside `<date>/tmp/routing`

    <img width="1853" height="915" alt="Screenshot 2025-07-29 194603" src="https://github.com/user-attachments/assets/c2643529-ff21-4734-a34a-dbd7a7ec5990" />

- Post-Route parasitic extraction using SPEF extractor
  
  for the extracting, we use [SPEF_EXTRACTOR](https://github.com/HanyMoussa/SPEF_EXTRACTOR)
  
  ```
  cd Desktop/work/tools/SPEF_EXTRACTOR
  
  python3 main.py /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/26-03_08-45/tmp/merged.lef                                     /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/26-03_08-45/results/routing/picorv32a.def
  ```
  
- Post-Route OpenSTA timing analysis

  <img width="1857" height="936" alt="Screenshot 2025-07-29 195516" src="https://github.com/user-attachments/assets/5d7777a5-7e44-4c99-a3c7-7ad856b9cfe1" />
  
  

**Picorv32a.def.png**


<img width="751" height="817" alt="Screenshot 2025-07-30 133612" src="https://github.com/user-attachments/assets/09ab689e-e161-4e35-a7cf-3023380250a4" />


**Picorv32a.gds.png**


<img width="1055" height="857" alt="Screenshot 2025-07-29 224044" src="https://github.com/user-attachments/assets/ace98a9a-4742-4409-8a97-660841e75a81" />



**GDS**

Stands for Graphic Design System. This is the file that is sent to the foundry and is called "tape-out".

In openLane use the command run_magic

The GDSII file is generated in the `results/magic` directory


**[DEF](https://teamvlsi.com/2020/08/def-file-in-vlsi-design-exchange.html)**

**[GDSII](https://en.wikipedia.org/wiki/GDSII)**


# Acknowledgements

[Kunal Ghosh](https://github.com/kunalg123) – Founder, VLSI System Design Corp.

[Nickson Jose](https://github.com/nickson-jose)– Developer & Contributor, Open Source Physical Design


     




    






  
  
  

  

  

    

 
    
    

    

    
 
    







  


















