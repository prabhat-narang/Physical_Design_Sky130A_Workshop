# Physical_Design_Sky130A_Workshop


## Introduction to OpenLANE
Helpful links:
- [OpenLANE Github repository](https://github.com/efabless/openlane)
- [Fossi Dial Up Tim Ansell](https://www.youtube.com/watch?v=EczW2IWdnOM&list=PLUg3wIOWD8yoZCg9XpFSgEgljx6MSdm9L&index=2)
- [Fossi Dial Up Mohamed Shalan](https://www.youtube.com/watch?v=Vhyv0eq_mLU&list=PLUg3wIOWD8yoZCg9XpFSgEgljx6MSdm9L&index=3)

### Directory Structure

### Design Preparation Step

### Reviewing files
After successfull execution of design preparation step, a 'runs' folder appears in the project directory. Inside 'runs', a folder with date of execution as name is created. In this folder, all the folders except the 'temp' folder is empty after the previous step. The following files/folders are present in this folder:
- tmp: It has merged.lef file. It is made by merging .tlef files and .lef files. It has layers and via information as well as cell level information. This folder has temporary files as the name suggests.
- results: Resulting outputs are stored here after each stage (Synthesis, Placement, Floorplan, etc)
- reports: Repoorts generated at each stage is stored here.
- logs: Contains logs for each stage.
- config.tcl: This file shows all default parameters taken by run. Also contains location of tlef, lef, lib and other important files. The parameters in this file are updated on the go. So after a stage, if we decide to change a parameter of that step, the changes will be displayed in this file, and can be used for debugging.
- cmds.log: Contains log of commands that are executed.
![Screenshot of Reviewing Files]()

### Running Synthesis Step
To run the sythesis step, which runs Yosys and ABC, run command 
```
run_synthesis
```
This step usually take 5-6 minutes to execute.
![Screenshot of Synthesis step]()

After the command has executed, Yosys + ABC as well as STA has been done. The final synthesis statistic report is displayed. 
![Screenshot of Synthesis Statistics]()

To find the flop ratio,
```
                 No. of D flip flop
Flop Ratio =    ____________________   
                 Total no. of cells
```              
```
No. of D Flip Flops = [PDK]_sc_[variant]__dfxtp_2 = 1613
Total no. of cells = 14876
Flop Ratio = 1613/14876 = 0.1084
```
![Screenshot of result synthesis]()


- The synthesised netlist os generated in the `/results/synthesis` folder as described before. 
- In the '/reports/synthesis' folder, 
    - we can find `1-yosys_4.stat.rpt` statistics report as displayed in terminal after syntheis command is executed. 
    - we can find `2-opensta_main.timing.rpt` OpenSTA timing report file.

![Screenshot of opensta timing report]() 

## Floorplanning
### 1. Defining width and height of core and die
- Core: Section of the chip where the fundamental logic of the design is placed.
- Die: Die encapsulates the core, and is a small semiconductor material specimen on which the fundamental circuit is fabricated.
![Screenshot of core_vs_die](core_vs_die)

Let's say we have a basic netlist. We can find the area of each individual standard cell component present in the netlist. We can leave the wires for now. By approximately adjoining he cells in apropriate places, we can find the area occupied by the netlist. 
- We can find how efficient the placement of layout is with a metric, **Utilisation Factor**. 
- We can find if the netlist is square shaped or in rectangle form by **Aspect Ratio**.

```
                      Area occupied by the netlist
Utilisation Factor = ______________________________
                         Total Area of the core
Aspect Ratio = Height / Width         
```
Example, where netlist area = 2x2 = 4 unit and core are = 4x4 = 16 unit:
![Screenshot of example of Utilisation factor and Aspect ratio](uf_ar_example)

### 2. Defining location of Pre-placed Cells
If we have a huge combinational logic as a part of a design, that translates to say 100k gate netlist, we can take that out of the main netlist. 
- We can simplify the netlist by dividing it into smaller circuits.
- We can convert each netlist into blocks with individual inputs and outputs.  
- These can be used as black blocks and reused wherever needed.

![Screenshot of Combination logic to blocks](cl_to_blocks)
![Screenshot of Blocks to Black Box](blocks_to_blackbox)

The black box defined above then can be seperated into modules with their own inputs and outputs. Each of these modules will be implemented seperately and only once. They are usually called IPs or modules. An **IP** or **module** is individual circuit with it's own sets of input and ouputs that does a specific task. It can be incorporated in larger design to simplify the design process or used where the same circuit is repeated number of times in a design.
![Screenshot of Black Box to IP](blackbox_to_ip)

Some of the readily available IPs include Memory, Clock Gating Cell, Comparators, Mux, DAC etc.

- Arrangement of these IPs or modules is called **Floorplanning**. 
- The location of these IPs or modules is fixed on a floorplan by the user before automated PnR is run, hence they are called **Pre-placed cells**. 
- Once the location of these IPs is fixed, the automated PnR tool does not change their location but places rest of the logic cells in the design onto the chip.

If we have 3 IPs, Block 1, 2, and 3, we can place them on the floorplan depending on the physical I/O pins and the block's I/O pins as shown below. 
![Screenshot of preplaced cells fixed](location_preplacedcell)

### 3. Decoupling Capacitors
In a switching circuit, all the elements such as a logic gate continously needs to switch their output from low-to-high or high-to-low. This element sees a capacitor at it's output as a load due to parasitic capacitance of the metal interconnects. During switching of output from low-to-high, current is drawn by the element from it's power supply's Vdd to charge the load capacitor at the output to the Vdd level of the circuit. Similarly, during switching from high-to-low, current is discharged from the output capacitor to the power supply's Vss pin so that output can be brought down to Vss level. This can be understood from following diagram.

![Screenshot of need for decoupling](need_for_decoupling)

Since the physical power supply pins are physically distant from modules or IPs that have these switching circuits, there are a series of physically long wires (wire bonds from physical pins and interconnects) that connect the Vdd and Vss of modules to chip's Vdd and Vss pins. These physical wires have considerable series inductance and resistance due to their small and limited dimensions. Thus, when during switching of the outputs of various logic elements from low-to-high, a in-rush current flows from Vdd to module's Vdd, and a in-rush current flows from module's Vss to Vss during switching from high-to-low of the outputs. This in-rush current causes momentary change in currents through the wires. This causes a voltage drop in the wires equal to
```
V_drop = i*R_dd + L_dd*(di/dt)
```
where R_dd is series resistance, L_dd is series inductance, and i is the current flowing through the wire. Same equation applies for R_ss and L_ss.

This voltage drop reduces the Vdd of module to Vdd' and increases the Vss from 0 to Vss'. So, the output capacitor of logic elements in the module is charged to Vdd' that is available to it and and discharged to Vss'. This output is an input for the next element. Thus, the voltage swing range has been reduced for the next element's input. This swing range must remain in the noise margin range as shown below for the circuit to work.

![Screenshot of noise margin](noise_margin)

A large capacitor must be place physically close to the IP or module which charges from the power supply, and supplies the required charge for the elements to charge their output capacitors during switching action without significantly dropping the supply voltage of the module. 
- The capacitor must be large so that there is no significant voltage change during discharging or charging of the capacitor from the in-rush currents from the module.
- The capacitor must be physically close so that there is no appreciable series resistance and inductance of the wires connecting capacitor to the module.

![Screenshot of decoupling](decoupling)
![Screenshot of decoupled IP](decoupled_ip)

### 4. Power Planning

Let's say we have multiple modules each with their decoupling capacitors coneected to them placed all over the chip. If a module is driving it's output from low-to-high or vice versa, which is input to another module, the Vdd and Vss each module sees must be same or whithin noise margin range with respect to each other for the circuit to function properly. If the power supply is being supplied from a single point, the distance from the power supply will be different for each module and due the the series resistance and inductance, may see different power supply voltages.
![Screenshot of single source of supply](single_source_supply)

Also, if we have a multi-bit bus in which a number of bits are being switched from 1 to 0, the load capacitance on each bit line must be discharged from Vdd to 0 through a ground tap point each capacitor connects to if there is single ground source. This will cause significant current to flow through ground tap point which will cause a potential drop to appear at the local ground tap point with respect to Vss of the chip. This is called **ground bounce**.
![Screenshot of Ground Bounce](ground_bounce)

Similarly, if a number of bits are being switched from 0 to 1, a significant current flows from Vdd tap point of the module causing a voltage drop.
![Screenshot of Voltage Drop](voltage_drop)

If these voltage drops and bounce are greater than the respective noise margins, erroneous interpretation of the bit values may take place. To mitigate the above issues, power supply is supplied from multiple direction on the chip, so that currents take the path of least impedance to or from the modules. The Vdd and Vss interconnects are placed in a grid fashion so that there are multiple Vdd as well as Vss tap point available to the bus lines as well as to the modules so that there is no significant current flowing from one single wire at a time. this is ccalled as **power planning**.
![Screenshot of Multi-Source Supply](multi_source_supply)
![Screenshot of power planning](power_planning)

### 5. Pin Placement
Since the chip needs to interact with outside world, the input and output signals of the designed chips are connected to pins that communicate physically with the circuit outside the chips. The pins are usually placed such that inputs are on the left, and outputs are on the right. But this is one of the practices, not hard and fast rule, inputs can be on upper side, outputs below, or both on left etc. The pins are placed so that input of modules that connect to these pins are close by. Similarly, output of the modules need to be close to output pins so that routing becomes easy.
- The pins are placed in the area between core and die.
- The size and width of pins depends on the current that needs to pass through them, so power pins, clock pins are wide, whereas I/O pins are thin.
![Screenshot of pin placement](pin_placement)

### 6. Logical Cell Placement Blockage
The region where pins are place, the region between core and die, is blocked so that the autorouter does not place cells here. This is called Logical Cell Placement Blockage.
![Screenshot of Placement Blockage](placement_blockage)

### Floorplan using OpenLANE
- In this stage, we set die area, core area, utilisation factor, aspect ratio, place the I/O cells, make power distribution network and macro placement.
- Standard cells are not placed in this stage, they are set in placement stage.

We can set the switches to customise the floorplan stage. The switches for floorplan can be found in `openlane/configurations/README.md` file under the floorplan section. 
![Screenshot of Floorplan Section of REAME](floorplanning_section_readme)

Some important switches are:
1. `FP_CORE_UTIL` : Core utilisation percentage.
2. `FP_ASPECT_RATIO` : Aspect ratio of netlist
3. `FP_CORE_MARGIN` : Margin in um, between core and die boundary
4. `FP_IO_HMETAL` : Metal layer on which horizontal IO pins are placed. (Actual layer is +1 of this number)
5. `FP_IO_VMETAL` : Metal layer on which vertical IO pins are placed. (Actual layer is +1 of this number)
6. `FP_ENDCAP_CELL` : Decoupling cap information
7. `FP_PDN_xxx` :  Power distribution network generation parameters (5 parameters)

- These switches are set in `openlane/configurations/floorplan.tcl` file as general design parameters. 
- For every project, the values can be customised in `config.tcl` files and PDK specific configuration file `[PDK]_sc_[variant]_config.tcl` in the project directory. 
- PDK specific file `[PDK]_sc_[variant]_config.tcl` has highest priority, then `config.tcl` file, then `floorplan.tcl` file.

To execute the floorplan stage
```
run_floorplan
```

A .def file is created in `results/floorplan` directory. 
![Screenshot of def file](def_file)

We can view the layout by open it in Magic with the command
```
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def &
```
Tech and merged lef files are provided to Magic for it to interpret the cells. `&` at the end of command frees up command line by pringing back the prompt.

![Screenshot of Magic View after floorplanning](magic_layout_planning)
![Screenshot of Pins and Decap in design](magic_pins_decap)

- `s` key selects the chip design or anything (like pads) under the cursor. `v` key zooms out to the whole layout.
- `z` key zooms in a block selected by left click and then right clicking.
- `what` command in tkcon window gives information about selected component.
- Standard cells are present in lower left corner as they are not placed yet.
- Zooming near the pads, we can see decoupling capacitors.

If we want to change the configurations on the go, for eg. IO placement mode,
```
set ::env(FP_IO_MODE) 2
run_floorplan
```


## Placement

### 1. Library Binding
For each component in netllist, like FFs, gates etc, a library has information about their physical dimension of the cell, various flavours of the same cell, and timing information among others. This library information is 'binded' or correlated to the each component in the netlist.

### 2. Placement
Without disturbing or moving the pre-placed cells, the cells corresponding to netlist components are placed on the core. Their placemment depends on distance from pins, distance between nearby components in netlist etc.
![Screenshot of placement](placement)

### 3. Optimising placement
- With the cells placed, wire path from pin to cell or cell to cell is approximated, and based on that, wire parameters like capacitance, resistance etc are calculated for simulation. 
- The signal integrity is tested by checking if the slew rate is within the acceptable range for the approximated wire.
- If the slew rate is not within acceptable range, it means wire lenth is too large for proper signal integrity.
- In this case, buffers (or repetaers) are placed in between the path where signal integrity was violated.
- In case of high frequency designs, the individual cells are placed right next to each other to minimise the delay caused by the wires. This is called **Abutment** (eg. yellow cells)
- In case criss crossing happens, ie, when we can see the paths of wires may intersect while routing, one of the solution is to route on different metal layers.

![Screenshot of Potimising Placement](placement_optimisation)

### Placement using OpenLANE
- Placement occurs in 2 stages, Global and Detailed. Global Placement done using **RePlAce**.
- Congestion aware placement and timing constrained placement. In this, congestion aware placement is done.
- In global placement, objective is to reduce wire length by reduction of parameter HPWL (Half Parameter Wire Length).

To run Placement stage, run the command
```
run_placement
```

Results are stored in `results/placement` folder in .def file. Results can be viewed in Magic by

```
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def &
```

![Screenshot of final placement](placement_final)
![Screenshot of zoomed placement](placement_zoom)

## Cell Design Flow
A library consists of cells (like buuders, FFs etc) with different parameters like size, drive strength, Vth etc. The cell is designed with the following flow.
1. Inputs: Input to the cell design flow is PDKs. PDKs defiine DRC and LVS rules (in tech file), SPICE models, library and user-defined specs. Library defined specs include cell height, cell width, drive strength. User defined spec include supply voltage, metal layers, pin locations etc. 
2. Design Steps: This step includes-
  -  Circuit design step: Functionality of the cell is implemented using nmos and pmos, and the circuit parameters are defined based on the requirements. For eq, W/L ratio of nmos and pmos are desfined based on current requirement, switching point etc. Outpput of this step is CDL.
  -  Layout design step: Based on Euler's path and stick diagram. From the circuit design above, nmos and pmos network graphs are extracted. Then, the Euler's path is obtained. After that, a stick diagram is drawn. From stick diagram, proper layout diagram is drawn by adhering to the rules and specs from the Inputs stage. This diagram is used to make the actual layout in the EDA tool.
  -  Characterisation: Flow- Read in the SPICE model from PDK. Read extracted SPICE Netlist. Recognise beahviour of buffer. Read sub-circuit of buffer. Power supply is applied to the circuit. Necessary stimulus is given as input to the circuit. Output load capacitance is defined. Simulation parameters are defined. All these parameters as configuration file is provided to **GUNA** software and output is generated as varrious .lib files.
3. Outputs: Outputs include:
  - CDL (Circuit Descriptin Language): Output of Circuit design step.
  - GDSII, LEF, extracted spice netlist (.cir): GDSII is layout file, LEF stores dimensions of cell, .cir file stored extracted parasitic R,L,C values from the layout. These are output of Layout step.
  - Timing, noise, power .libs function: Output of characterisation step.

![Screenshot of Cell Design Flow](cell_design_flow)

### Timing Characterisation
**Timing Threshold Definitions**
1. `slew_low_rise_thr`: low point on rising waveform to calculate slew.
2. `slew_high_rise_thr`: high point on rising waveform to calculate slew.
3. `slew_low_fall_thr`: low point on falling waveform to calculate slew.
4. `slew_high_fall_thr`: high point on falling waveform to calculate slew.
5. `in_rise_thr`: threshold point on input rising waveform
6. `in_fall_thr`: threshold point on input falling waveform
7. `out_rise_thr`: threshold point on output rising waveform
8. `out_fall_thr`: threshold point on output falling waveform

To calculate propogation delay between input and output of buffer, we use the time differnce between `out_rise_thr` and `in_rise_thr` point on waveform or vice-versa.
```
Tpd = T(out_*_thr) - T(in_*_thr)
```
There can be error in finding the delay value and delay values can come out as negative in the following case.
- Thereshold value is not correctly set.
- The circuit is faulty and/or there is a huge slew in rising/falling waveform.

To calculate slew
 ```
 Tslew = T(slew_high_*_thr) - T(slew_low_*_thr)
 ```
## Designing a library cell
We create a SPICE deck for a circuit. A SPICE deck has:
- Model
- Netlist
- Component Values
- Indentify 'Nodes'
- Name 'Nodes'
- Simulation Information
- library files

**Switching Threshold**
- Inverter is very robust circuit. With varying W/L ratios of pmos and nmos, the DC transfer characteristics waveform only shifts but the shape remains intact.
- One of the parameter to quantify robustness of inverter if **Threshold Voltage (Vm)**
- Point where `Vin = Vout`
- `IdsP = -IdsN` at this point
- Both transistors in saturation region

### 16 Mask CMOS Process 
1. Selecting a substrate: Usually P-type, high resitivity, substrate doping level which is less than 'well' doping level, 100 orientation.
2. Creating active regions for transistors
3. N-well and P-well formation: In P-well, nmos transistor is made, and vice versa.
4. Gate formation
5. Lightly Doped Drain (LDD) formation: 2 reasons for this, 1) Hot electron effect, 2) Short channel effect. Both associated with reduction of size. N+ means high concentration, N- means low concentration.
6. Source and Drain formation: Channeling effect is when vector velocity of ions matches with silicon structure, ions may go deep in P substrate without being blocked by Si atoms above. Screen oxide is used to randomize the ion directions and cause deposition.
7. Contacts and local interconnects formation
8. Higher level metal formation 

![Screenshot of 16 Mask Process](st_mask_process)

### Layout of Inverter in Magic
The inverter repository is cloned to local machine in openlane directory.
```
git clone https://github.com/nickson-jose/vsdstdcelldesign.git
```

The `pdks/sky130A/libs.tech/magic/sky130A.tech` file is copied to the folder created above.
```
cp sky130A.tech ...
```

To open the layout of inverter, Magic is called from `vsdstdcelldesign` folder and .mag file is opened.
```
magic -T sky130A.tech sky130_inv.mag &
```

To verify the design, 
- we can check the connections by hovering the cursor, and pressing 's' key two or three times to select all the connected pins and wires.
- The intersection of polysilicon layer with n diffusuion is where nmos is formed and vice versa. This can be verified by selecting the intersection region and passing `what` command in tkcon.

*LEF files are abstract picture of the cell using only the metal layers. Useful for placement and protection of IP.*

In the Magic Tool, we can check for DRC errors by clicking on DRC > DRC Find Next Error. The error can be viewed in tkcon window after the error site has been selected by Magic. 

To extract the SPICE netlist from the layout, in the tkcon window, execute command
```
extract all
```
 This creates a .ext file in the folder. To extract the parasitic resistance and capacitance value,
```
ext2spice cthresh 0 rthresh0
```
No new file will be created. To get the final .spice file, execute
```
ext2spice
```

![Screenshot of MAGIC inverter layout](magic_inv_layout)





