# VSDMemSOC
VSDMemSOC Implementation flow:: RTL2GDSII
### VSDMemSoC
VSDMemSoC is an SoC that includes a RISC-V based processor called RVMYTH and an external 1kB SRAM instruction memory (Imem), separating the instruction memory from the processor core.
### Introduction to VSDMemSoC
The purpose of designing such an SOC to is to demonstrate the concept of separating the main core and memories of processor. This would help to make the RISC-V more modular and portable. Here, the instruction memory is separated.

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
  
# List of commands used in the openlane flow

```
cd OpenLane/ 
make mount 
{ If Error occurs use the below commands in OpenLane directory:
sudo chown $USER /var/run/docker.sock 
PYTHON_BIN=python3 make mount
}

./flow.tcl -interactive
package require openlane 0.9
prep -design picorv32a
run_synthesis
run_floorplan
run_placement
run_cts
run_routing
run_magic
run_magic_spice_export
run_magic_drc
run_netgen
run_magic_antenna_check
```

### Synthesis Exploration
At the begining a synthesis strategy exploration can be done using OpenLane command ```./flow.tcl -design vsdmemsoc synth_explore```
The ouput gives area estimation for different stratagies.

**Image of synthesis stratagies table**

<img width="542" alt="Screenshot 2024-03-01 152814" src="https://github.com/Pa1mantri/VSDMemSOC/assets/114488271/37b6e651-de37-43a8-809b-ceef16068177">

After running synthesis, Pre-layout STA report and chip Area information is available.

**Image of synthesis STA, Power**

<img width="885" alt="Screenshot 2024-03-01 153026" src="https://github.com/Pa1mantri/VSDMemSOC/assets/114488271/ca025198-20e0-4341-8060-b6cb2d20984e">

<img width="897" alt="Screenshot 2024-03-01 153002" src="https://github.com/Pa1mantri/VSDMemSOC/assets/114488271/8a693790-92ce-4ddb-bfb4-ffc606972fc1">

**yosys report**

<img width="926" alt="Screenshot 2024-03-01 144324" src="https://github.com/Pa1mantri/VSDMemSOC/assets/114488271/757a6e74-6b2b-4add-a0b7-69b2bfb86fb6">


### Next step is the floorplan

.def file is created. Can be viewed using magic tool using the command ```magic -d XR -T /home/pa1mantri/.volare/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.nom.lef def read vsdmemsoc.def &``` tech and lef information is also provided to the magic tool.

### Placement

.def file is created. Global and detailed placements happens during this step.

.def file opened through magic tool using tech and lef files.

<img width="719" alt="Screenshot 2024-03-01 151047" src="https://github.com/Pa1mantri/VSDMemSOC/assets/114488271/05d0e366-140d-4bb3-a011-a8619002652a">

### CTS

Up-until now, an ideal clock is considered. Now real clock gets into the picture.

**Clock Tree for the entire design**

<img width="727" alt="Screenshot 2024-03-01 142542" src="https://github.com/Pa1mantri/VSDMemSOC/assets/114488271/ab3cbb77-4173-4920-bc97-4908c177aad9">

**Power Report and Clock skew report**

<img width="898" alt="Screenshot 2024-03-01 151350" src="https://github.com/Pa1mantri/VSDMemSOC/assets/114488271/3e6ec9e6-db7e-430d-b5ea-4908606f877a">

<img width="899" alt="Screenshot 2024-03-01 151401" src="https://github.com/Pa1mantri/VSDMemSOC/assets/114488271/6084de85-81ba-4d98-9db0-e565fcf482bb">

# Routing

**Routing Congestion view from OpenRoad**

<img width="717" alt="Screenshot 2024-03-01 142643" src="https://github.com/Pa1mantri/VSDMemSOC/assets/114488271/d7400188-724e-4aca-97f0-e5f3379c7171">

**Power and Clock skew report**

<img width="897" alt="Screenshot 2024-03-01 151745" src="https://github.com/Pa1mantri/VSDMemSOC/assets/114488271/33cf0a4e-e33c-4c2d-9990-ba4e0d413275">

<img width="896" alt="Screenshot 2024-03-01 151756" src="https://github.com/Pa1mantri/VSDMemSOC/assets/114488271/d0bf5520-fb19-4040-930c-711f78126442">


## GDSII

GDS Stands for Graphic Design Standard. This is the file that is sent to the foundry and is called as "tape-out".

**GDS file from OpenRoad**

<img width="716" alt="Screenshot 2024-03-01 142512" src="https://github.com/Pa1mantri/VSDMemSOC/assets/114488271/01ad1c75-ada3-4f15-a058-492b15e471c6">

**Flow completed with zero violations**

<img width="916" alt="Screenshot 2024-03-01 123857" src="https://github.com/Pa1mantri/VSDMemSOC/assets/114488271/b7736a32-8b79-4d63-8b01-313e5b3957b8">

**Total Summary file generated at the end of the flow with all the details**

<img width="923" alt="Screenshot 2024-03-01 144509" src="https://github.com/Pa1mantri/VSDMemSOC/assets/114488271/e51d3c8a-94da-4746-9a18-81a39a8cc27e">

<img width="856" alt="Screenshot 2024-03-01 144832" src="https://github.com/Pa1mantri/VSDMemSOC/assets/114488271/29c9e1f8-8c78-4086-b99f-31bfdcd4f4d7">






