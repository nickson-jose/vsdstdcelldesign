# Standard cell design and characterization using openlane flow
This repository contains all the information needed to build and run openlane flow, which has the capability to perform full ASIC implementation steps from RTL to GDSII. In addition, it also contains procedures on how to create a custom LEF file and plugging it into an openlane flow.

# Table of Contents
- [Introduction to Openlane flow.](#introduction-to-openlane-flow)
- [Overview of Physical Design flow.](#overview-of-physical-design-flow)
- [Build and invoke openlane.](#build-and-invoke-openlane)
- [Introduction to LEF.](#introduction-to-lef)
- [Standard cell design and characterization in openlane.](#standard-cell-design-and-characterization-in-openlane )
  - [Objective.](#objective)
  - [About PicoRV32.](#about-picorv32)
  - [Standard cell layout design in Magic.](#standard-cell-layout-design-in-magic)
  - [Create port definition.](#create-port-definition)
  - [Set `port class` and `port use` attributes for a layout.](#set-port-class-and-port-use-attributes-for-a-layout)
  - [Defining LEF properties and extracting LEF file.](#defining-lef-properties-and-extracting-lef-file)
  - [Plugging custom LEF to openlane flow.](#plugging-custom-lef-to-openlane-flow)
  - [Observations.](#observations)
  - [Challenges.](#challenges)
- [Future work.](#future-work)
- [Limitations.](#limitations)
- [Acknowledgements.](#acknowledgements)

# Introduction to Openlane flow
OpenLANE is a completely automated RTL to GDSII flow which embeds in it different opensource tools, namely, OpenROAD, Yosys, ABC, Magic etc., apart from many custom methodology scripts for design exploration and optimization. Openlane is built around Skywater 130nm process node and is capable of performing full ASIC implementation steps from RTL all the way down to GDSII. 
The flow-chart below gives a better picture of openlane flow as a whole (**Image Courtesy:** [efabless/openlane](https://github.com/efabless/openlane/blob/master/doc/openlane.flow.1.png))

![alt text](https://github.com/njose939/OpenLane/blob/master/Images/openlane.flow.1.png?raw=true)
  
# Overview of Physical Design flow
Place and Route (PnR) is the core of any ASIC implementation and Openlane flow integrates into it several key open source tools which perform each of the respective stages of PnR.
Below are the stages and the respective tools (in ( )) that are called by openlane for the functionalities as described:
- Synthesis
  - Generating gate-level netlist ([yosys](https://github.com/YosysHQ/yosys)).
  - Performing cell mapping ([abc](https://github.com/YosysHQ/yosys)).
  - Performing pre-layout STA ([OpenSTA](https://github.com/The-OpenROAD-Project/OpenSTA)).
- Floorplanning
  - Defining the core area for the macro as well as the cell sites and the tracks ([init_fp](https://github.com/The-OpenROAD-Project/OpenROAD/tree/master/src/init_fp)).
  - Placing the macro input and output ports ([ioplacer](https://github.com/The-OpenROAD-Project/ioPlacer/)).
  - Generating the power distribution network ([pdn](https://github.com/The-OpenROAD-Project/pdn/)).
- Placement
  - Performing global placement ([RePLace](https://github.com/The-OpenROAD-Project/RePlAce)).
  - Perfroming detailed placement to legalize the globally placed components ([OpenDP](https://github.com/The-OpenROAD-Project/OpenDP)).
- Clock Tree Synthesis (CTS)
  - Synthesizing the clock tree ([TritonCTS](https://github.com/The-OpenROAD-Project/OpenROAD/tree/master/src/TritonCTS)).
- Routing
  - Performing global routing to generate a guide file for the detailed router ([FastRoute](https://github.com/The-OpenROAD-Project/FastRoute/tree/openroad)).
  - Performing detailed routing ([TritonRoute](https://github.com/The-OpenROAD-Project/TritonRoute))
- GDSII Generation
  - Streaming out the final GDSII layout file from the routed def ([Magic](https://github.com/RTimothyEdwards/magic)).
  
# Build and Invoke openlane

Detailed description on how to build and invoke openlane is given in this [link](https://github.com/njose939/openlane_build_script).

#  Introduction to LEF
For a PnR tool to correctly place and route a block (a macro or a std. cell), it doesn't need to know entire layout information of the block; just the pin positions, PR boundary is sufficient. These minimal and abstracted information is provided to the tool by the Library Exchange Format (LEF) file. LEF file also serves the purpose of protecting intellectual property and is basically of two types:
- Cell LEF - It's an abstract view of the cell and only gives information about PR boundary, pin position and metal layer information of the cell.
- Technology LEF - It contains information about available metal layer, via information, DRCs of particular technology used by placer and router etc..
The below diagram highlights the difference between a layout and a LEF (**Image Courtesy:** Google):

![alt text](https://github.com/njose939/OpenLane/blob/master/Images/layout_vs_LEF.JPG?raw=true)

# Standard cell design and characterization in openlane

## Objective

The goal of the project is to design a single height standard cell and plug this custom cell into a more complex design and perform it's PnR in the openlane flow. The standard cell chosen is a basic CMOS inverter and the design into which it's plugged into is a pre-built [picorv32a](https://github.com/efabless/openlane/tree/master/designs/picorv32a) core. 

## About PicoRV32

PicoRV32 is a CPU core that implements the RISC-V RV32IMC Instruction Set. It can be configured as RV32I, RV32IC, RV32IM, or RV32IMC core; where the suffixes stand for:

- M - Multiply extension
- I - Base Integer Instructions
- C - Compressed Instructions

PicoRV32 is free and open hardware licensed under the ISC license. All features and data-sheet related to picoRV32 core can be obtained [here.](https://www.efabless.com/design_catalog/ip_block/92)

## Standard cell layout design in Magic

- The proposed inverter for the design is a single height standard cell, so the dimensions needs to be a multiple of the single height place site; which for sky130 node has a nomenclature of `unithd` with dimensions(in microns): 0.46 x 2.72 (width x height) for **sky130_fd_sc_hd** PDK variant. The magic tool is invoked with sky130 tech file as `magic -T sky130A.tech &` (the magic tech file (sky130A.tech) has also been included in this repo under `/libs` as reference).

- Thus, the first step in magic layout tool is to create a bounding box with a width of 1.38 (3 x width(unithd)) and height of 2.72. This can be done by using command `property FIXED_BBOX {0 0 138 272}` in magic tkcon window.

![alt text](https://github.com/njose939/OpenLane/blob/master/Images/initial_bbox.JPG?raw=true)

- This is followed by defining the ground and power segments (in metal 1), the respective contacts and finally the layout of the logic part. Same procedure can be followed for any standard cell layout.

![alt text](https://github.com/njose939/OpenLane/blob/master/Images/initial_metal2.jpg?raw=true)

### Note:
The layout (also included as the part of this repo, viz., `sky130_inv.mag`) can be viewed in magic layout window as `magic -T sky130A.tech sky130_inv.mag &` 

## Create port definition

Once the layout is ready, the next step is extracting LEF file for the cell. However, certain properties and definitions need to be set to the pins of the cell which aid the placer and router tool. For LEF files, a cell that contains ports is written as a macro cell, and the ports are the declared PINs of the macro. Our objective is to extract LEF from a given layout (here of a simple CMOS inverter) in standard format. Defining port and setting correct class and use attributes to each port is the first step. 
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

## Set port class and port use attributes for a layout

Post port definition, the next step is setting **port class** and **port use** attributes. The "class" and "use" properties of the port have no internal meaning to magic but are used by the LEF and DEF format read and write routines, and match the LEF/DEF CLASS and USE properties for macro cell pins. Valid classes are: default, input, output, tristate, bidirectional, inout, feedthrough, and feedthru. Valid uses are: default, analog, signal, digital, power, ground, and clock. These attributes are set in tkcon window (after selecting each port on layout window. A keyboard shortcut would be repeatedly pressing `s` till that port gets highlighed) as:

![alt text](https://github.com/njose939/OpenLane/blob/master/Images/port_class_use_1.JPG?raw=true) 

#### Additional:
You can delete or remove any port by first selecting the port (key `s`) and then executing below two commands in order (in tkcon window):
* `port remove`
* `label erase`

## Defining LEF properties and extracting LEF file

Certain properties needs to be set before writing the LEF. As mentioned before, these values are fetched by placer and router to determine, for instance, site where a cell needs to be placed. Macro cell properties common to the LEF/DEF definition but that have no corresponding database interpretation in magic are retained using the cell **property** method in magic. There are specific property names associated with the LEF format. Once the properties are set, `lef write` command writes the LEF file with the same nomenclature as that of the layout (.mag) file.

![alt text](https://github.com/njose939/OpenLane/blob/master/Images/final_LEF_write.JPG?raw=true) 

## Plugging custom LEF to openlane flow

If a new custom cell needs to be plugged into openlane flow, include the lefs (the one extracted in [Step-5](#defining-lef-properties-and-extracting-lef-file)) as below:

- In the design's config.tcl file add the below line to point to the lef location which is required during spice extraction.
      
          set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]
      
- Include the below command to include the additional lef into the flow:
      
          set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
        
          add_lefs -src $lefs
       
- Run the interactive flow as described [here.](https://github.com/efabless/openlane/blob/master/doc/advanced_readme.md)
  
   **Note:**
     A sample inverter magic file (_sky130_inv.mag_) has been included as a reference resource.

## Observations

The custom inverter successfully included in the picorv32a design. Below is the final routed picorv32a design with the custom cell zoomed in and highlighted.

![alt text](https://github.com/njose939/OpenLane/blob/master/Images/final_routing_1.JPG?raw=true) 

## Challenges

- The biggest challenge was with the legalization of the cell. Initial iteration showed illegal positioning of the cell away from the standard cell rails.

![alt text](https://github.com/njose939/OpenLane/blob/master/Images/legalization_issue_1.jpg?raw=true)

- Upon closer inspection, the issue seemed to be with the dimensions of the drawn power and ground rails and also with the positioning of local1 -> metal1 contacts which was then corrected for further iterations.

![alt text](https://github.com/njose939/OpenLane/blob/master/Images/dimensions.JPG?raw=true)

# Future work

The upcoming work of this project would be detailed IP characterization once all corner spice models is made availabe in Google-skywater [public forum.](https://github.com/google/skywater-pdk)

# Limitations

At present, Openlane has following limitations:
- Limited Timing Constraints.
- Timing Closure.
  - No post-routing optimizations.

Nevertheless, Openlane has a huge community support and improvements are getting pushed every other day. For more details on openlane do visit [openlane1](https://github.com/efabless/openlane/) [openlane2](https://www.youtube.com/watch?v=Vhyv0eq_mLU).

# Acknowledgements
- [Kunal Ghosh](https://github.com/kunalg123), Co-founder (VSD Corp. Pvt. Ltd)
- [Ahmed Ghazy](https://github.com/ax3ghazy), openlane team, efabless

