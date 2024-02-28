# VSDMemSOC
VSDMemSOC Implementation flow:: RTL2GDSII
### VSDMemSoC
VSDMemSoC is an SoC that includes a RISC-V based processor called RVMYTH and an external 1kB SRAM instruction memory (Imem), separating the instruction memory from the processor core.
### Introduction to VSDMemSoC
The purpose of designing such an SOC to is to demonstrate the concept of separating the main core and memories of processor. This would help to make the RISC-V more modular and portable. Here, the instruction memory is separated.
*1.png*
### RVMYTH
RVMYTH core is a simple RISCV-based CPU, introduced in a workshop by RedwoodEDA and VSD.Core is developed using TL-Verilog for faster development.
### SRAM
An SRAM is a type of random-access memory (RAM) that uses latching circuitry (flip-flop) to store each bit. It is a volatile memory where data is lost when power is removed. It is typically used for the cache and internal registers of a CPU as it very fast but expensive. The SRAM here is a 1 kB 6-transistor type with a 8-bit address bus and 32-bit data bus.
### RVMYTH Modelling
RISC-V is developed using Transactional verilog. So, we use sandpiper tool to convert the RVMYTH core written in TL-verilog to verilog.
### SRAM Modelling
Here we used one of the SRAM's present in the PDK provided by the skywater sky130 technology. Here the sram used has 32-bit word size and 8-bit address space which results in 256 memory locations(words).
### OpenLane Flow
# Running OpenLane
Go to the OpenLane folder and run ```make mount```. OpenLane uses several tools. The scripsts and tools used during the atomic commands are found in ```<OpenLane_dir>/scripts```. Also OpenRoad related scripts are found in ```<OpenLane_dir>/scripts/openroad```.
# Commands to create and Run the design
Command used for creating the project for the first time ```./flow.tcl -design vsdmemsoc init_design_config -add_to_designs```
For running the design in interactive mode ```./flow.tcl -interactive```
For prepping the design ```prep -design vsdmemsoc```
Since we are using a SRAM Macro from the PDK, we need to add LEF files of that SRAM before we start the openlane flow. The following two commands are used to add the SRAM lef files to the design flow
```
set lefs [glob $::env(DESIGN_DIR)/src/macros/*.lef]
add_lefs -src $lefs
```
*2.png*

- Synthesis
  - Command run_synthesis
  - Generating gate-level netlist (yosys). - Performing cell mapping (abc).
  - Performing pre-layout STA (OpenSTA).
  - Usefull info for design stage: Flip-flop ratio, chip area, timing performance
  
- Floorplanning
  - Command run_floorplan
  - Defining the core area for the macro as well as the cell sites and the tracks (init_fp).
  - Placing the macro input and output ports (ioplacer).
  - Generating the power distribution network (pdn).
    - /results/ *.def , contains the design Core Area.

- Placement
  - run_placement - Performing global placement (RePLace).
  - Perfroming detailed placement to legalize the globally placed components (OpenDP).

- Clock Tree Synthesis (CTS)
  - Synthesizing the clock tree (TritonCTS).
  
- Routing
 - Performing global routing to generate a guide file for the detailed router (FastRoute).
 - Performing detailed routing (TritonRoute)
  
- GDSII Generation
  - Streaming out the final GDSII layout file from the routed def (Magic).



