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
To run the sythesis step, which runs Yosys and ABC, run command `run_synthesis`. This step usually take 5-6 minutes to execute.
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


- The synthesised netlist os generated in the '/results/synthesis' folder as described before. 
- In the '/reports/synthesis' folder, 
    - we can find '1-yosys_4.stat.rpt' statistics report as displayed in terminal after syntheis command is executed. 
    - we can find '2-opensta_main.timing.rpt' OpenSTA timing report file.

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

![Screenshot of decoupled IP](decoupled_ip)

### 4. Power Planning









