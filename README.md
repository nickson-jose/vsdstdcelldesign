# OpenLane
This repository contains all the information needed to run RTL to GDSII using openlane flow. In addition to that, it also contain procedures on how to create a custom LEF file and plugging it into an openlane flow.

# Table of Contents
- [Overview of openlane flow.](#overview-of-openlane-flow)
- [Introduction to basic Physical Design flow.](#introduction-to-basic-physical-design-flow)
- [How to build and invoke openlane?](#how-to-build-and-invoke-openlane?)
- [Introduction to LEF.](#introduction-to-lef)
  - [Create port definition and set attributes port class and port use for a layout.](#create-port-definition-and-set-attributes-port-class-and-port-use-for-a-layout)
  - [Define LEF properties.](#define-lef-properties)
  - [Extract a standard format LEF file for a sample layout.](#extract-a-standard-format-lef-file-for-a-sample-layout)
  
 # Overview of openlane flow
OpenLANE is a completely automated RTL to GDSII flow which has imbided in it many opensource components, viz., OpenROAD, Yosys, ABC, Magic etc and custom methodology scripts for design exploration and optimization. Openlane is built around Skywater 130nm process node and is capable of performing full ASIC implementation steps from RTL all the way down to GDSII. 
The flow-chart below gives a better picture of openlane flow as a whole (**Image Source:** [efabless/openlane](https://github.com/efabless/openlane/blob/master/doc/openlane.flow.1.png))

![alt text](https://github.com/njose939/OpenLane/blob/master/Images/openlane.flow.1.png?raw=true)
  
# Introduction to basic Physical Design flow
Place and Route (PnR) is the core of any physical design flow and OpenLANE integrates into it several key open source tools which perform each of the respective stages of PnR.
Below are the stages and the respective tools (in ()) that are called by openlane for the functionalities as described:
- Synthesis
  - Generating gate-level netlist (yosys).
  - Performing cell mapping (abc).
  - Performing pre-layout STA (OpenSTA).
- Floorplanning
  - Defining the core area for the macro as well as the cell sites and the tracks (init_fp).
  - Placing the macro input and output ports (ioplacer).
  - Generating the power distribution network (pdn).
- Placement
  - Performing global placement (RePLace).
  - Perfroming detailed placement to legalize the globally placed components (OpenDP).
- Clock Tree Synthesis (CTS)
  - Synthesizing the clock tree (TritonCTS).
- Routing
  - Performing global routing to generate a guide file for the detailed router (FastRoute).
  - Performing detailed routing (TritonRoute)
- GDSII Generation
  - Streaming out the final GDSII layout file from the routed def (Magic).
  
# How to build and invoke openlane?

Detailed description on how to build and invoke openlane is given in this [link](https://github.com/njose939/openlane_build_script).

#  Introduction to LEF
LEF (Library Exchange Format) is of two types:
- Cell LEF - It's an abstract view of the cell and only gives information about PR boundary, pin position and metal layer information of the cell.
- Technology LEF - It contains information about available metal layer, via information, DRCs of particular technology used by placer and router etc..
The below diagram highlights the difference between a layout and a LEF:

![alt text](https://github.com/njose939/OpenLane/blob/master/Images/layout_vs_LEF.JPG?raw=true)

## Create port definition and set attributes port class and port use for a layout

For LEF files, a cell that contains ports is written as a macro cell, and the ports are the declared PINs of the macro. Our objective is to extract LEF from a given layout (here of a simple CMOS inverter) in standard format. Defining port and setting correct class and use attributes to each port is the first step. 
The easiest way to define a port is through Magic Layout window and following are the steps:
- In Magic Layout window, first source the .mag file for the design (here inverter). Then **Edit >> Text** which opens up a dialogue box.

![alt text](https://github.com/njose939/OpenLane/blob/master/Images/portA.JPG?raw=true)

- For each layer (to be turned into port), make a box on that particular layer and input a label name along with a sticky label of the layer name with which the port needs to be associated. Ensure the Port enable checkbox is checked and default checkbox is unchecked as shown in the figure:

![alt text](https://github.com/njose939/OpenLane/blob/master/Images/portY.JPG?raw=true)

In the above two figures, port A (input port) and port Y (output port) are taken from locali (local interconnect) layer. Also, the number in the textarea near enable checkbox defines the order in which the ports will be written in LEF file (0 being the first).

- For power and ground layers, the definition could be same or different than the signal layer. Here, ground and power connectivity are taken from metal1 (Notice the sticky label)

| VPWR                                                                                        |   VGND        |
| --------------------------------------------------------------------------------------------| ------------- |
| ![alt text](https://github.com/njose939/OpenLane/blob/master/Images/portVPWR.JPG?raw=true)  | ![alt text](https://github.com/njose939/OpenLane/blob/master/Images/portVGND.JPG?raw=true) |

Post port definition, the next step is setting **port class** and **port use** attributes. These attributes define the direction as well as usage of each port and are set in tkcon window (after selecting each port on layout window. a keyboard shortcut would be repeatedly pressing `s` till that port gets highlighed) as:

![alt text](https://github.com/njose939/OpenLane/blob/master/Images/port_class_use.JPG?raw=true) 

## Define LEF properties

Certain properties needs to be set before writing the LEF. These values are fetched by placer and router to determine, for instance, site where a cell needs to be placed. Macro cell properties common to the LEF/DEF definition but that have no corresponding database interpretation in magic are retained using the cell **property** method in magic. There are specific property names associated with the LEF format. Once the properties are set, `lef write` command writes the LEF file with the same nomenclature as that of the layout (.mag) file.

![alt text](https://github.com/njose939/OpenLane/blob/master/Images/final_LEF_write.JPG?raw=true) 

