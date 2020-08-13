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

![alt text](https://github.com/njose939/OpenLane/blob/master/Images/layout_vs_LEF.jpg?raw=true)
