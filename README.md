# OpenLane
This repository contains all the information needed to run RTL to GDSII using openlane flow. In addition to that, it also contain procedures on how to create a custom LEF file and plugging it into an openlane flow.

# Table of Contents
- [Overview of openlane flow.](#overview-of-openlane-flow)
- [Introduction to basic PnR flow.](#introduction-to-basic-pnr-flow)
- [How to build and invoke openlane?](#how-to-build-and-invoke-openlane)
- [Introduction to LEF file.](#introduction-to-lef-file)
  - [Create port definition and set attributes port class and port use for a layout.](#create-port-definition-and-set-attributes-port-class-and-port-use-for-a-layout)
  - [Define LEF properties.](#define-lef-properties)
  - [Extract a standard format LEF file for a sample layout.](#extract-a-standard-format-lef-file-for-a-sample-layout)
  
 # Overview of openlane flow
OpenLANE is a completely automated RTL to GDSII flow which has imbided in it many opensource components, viz., OpenROAD, Yosys, ABC, Magic etc and custom methodology scripts for design exploration and optimization. Openlane is built around Skywater 130nm process node and is capable of performing full ASIC implementation steps from RTL all the way down to GDSII. 
The flow-chart below gives a better picture of openlane flow as a whole (**Image Source:** [efabless/openlane](https://github.com/efabless/openlane/blob/master/doc/openlane.flow.1.png)

![alt text](https://github.com/njose939/OpenLane/blob/master/Images/openlane.flow.1.png?raw=true)
  
