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











