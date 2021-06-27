# TABLE OF CONTENT
* Introduction to Verilog RTL Design and Synthesis.
* Timing libs, hierarchy vs flat synthesis and effective flop coding style.
* Combinational and Sequential optimizations.
* GLS, blocking vs non-blocking and synthesis simulation mismatches.
* If, case, for loop and generate statement.

# Introduction to Verilog RTL Design and Synthesis.

**Design: -** Behavioral representation of specification written in HDL language like vhdl, Verilog or system Verilog.

**Testbench: -** Component use to generate set of inputs for design and compare the output results with the specification.

**Simulator: -** tool use to check the design meets the defined requirements when some sets of inputs are applied through testbench. It takes design, testbench files and generate vcd file i.e value change dump file.

**Synthesizer: -** Checks the design and translate the RTL design into the netlist. Netlist is the representation of design in the from of standard cells like gates, flops and muxes.

![iver_gtk](https://user-images.githubusercontent.com/86521351/123533592-4c1a5f00-d734-11eb-8526-b4b155f5935b.PNG)

## **Iverilog:-** open source simulator we have used for simulation.
**Commands:-**
 * iverilog design.v tb.v -> generated binary file a.out
 * ./a.out -> generates tb.vcd file

## **Gtkwave:-** open source tool used for viewing waveform. It takes vcd file as input.
**Command:-**
 * gtkwave tb.vcd
 
 ![blocking_cveat_synth_wave](https://user-images.githubusercontent.com/86521351/123533688-0742f800-d735-11eb-83b2-e03b7df8aa04.PNG)
  Fig. Opening vcd file with gtkwave
  
## **YOSYS:-** It is an open source synthesizer tool.
**Commands:-**
* Yosys -> invoke the yosys prompt.
* read_liberty \<lib file\> -> read the standard cell library file.
* read_verilog \<Verilog file\> -> read design file
* synth -top top module name -> synthesize the design file
* abc -liberty \<lib file\> -> link the standard cells with the design defined in the library.
* Show -> show the circuit of the design in the from standard cells.
* Write_verilog -noattr \<filename.v\> -> dump the results of the synthesis into the Verilog file

![yosys](https://user-images.githubusercontent.com/86521351/123534112-4f174e80-d738-11eb-9f1b-ac2061068859.PNG)

![yosys1](https://user-images.githubusercontent.com/86521351/123534114-5dfe0100-d738-11eb-9824-d2631a74380e.PNG)

![yosys2](https://user-images.githubusercontent.com/86521351/123534116-648c7880-d738-11eb-8eb9-bd61a736451e.PNG)

![yosys3](https://user-images.githubusercontent.com/86521351/123534121-6ce4b380-d738-11eb-8933-ac1edffd3254.PNG)


# .lib, hierarchy vs flat synthesis and effective flop coding style

## .lib

* collection of standard cells, like or , nand , or , and gates etc
* after synthesis the design is converted into the cells which are defined in the .lib file
* contains different flavor of same cell


## What are different flavors of the gates and why we need them.

**Step time:-** the amount of time for which the input to flip flop should remain stable before the clock edge is arrived . If the input does not remain stable when the clock edge is arrives, it results in  metastable output.

**Hold time:-** The amount of time after the clock edge for which the output of the flop should remain stable so that it’s previous value should be captured by the next flop.  Failing to do results in loss of previous data.

![timings](https://user-images.githubusercontent.com/86521351/123538159-0b7d0e80-d751-11eb-9175-e42cd93431e9.PNG)


**Frequency of the circuit depends upon:-**

Tclk > Tca + Tcomb + Tsetup_b
Thold_a < Tqa + Tcomb

Freq = 1/tclk
Tca -> flop A propagation delay
Tcombi:- combinational delay
Tsetup_b:- setup time
Thold_b:- hold time


Maximum frequency of the circuit depends upon the combinational path of the circuit. To make the combinational delay less we need cells with less propagation delay. The cells with less propagation delay are known as fast cells.
The propagation of the input to the output of a cell in digital in circuit depends upon the load(capacitance). Fast charging /discharging of capacitance results less propagation delay from input to output. For fast charging of capacitor, we need it to be driven by high source current.
For that we need wider transistor.
But making cells faster result in hold time violation. So, for a circuit to work properly we need to maximum performance without any hold violations. So that’s why different flavors of same cells are given in the .lib file. These cells are selected depending upon the constraints given to synthesizer.

**Fast cells:-**
* Low proppogation delay
* Wider transistor
* More power
* Increase the performance of the circuit but results in more power and may leads to hold violations.

**Slow cells:-**
* Large propagation delay.
* Smaller transistor
* Less power
* Make the circuit slow.

**Sky130_fd_sc_hd_tt_025c_1v80.lib**
* Library used for synthesizing 
* 130nm technology node
* tt -> typical library contains fast ,slow or medium flavors of the same cell 
* 025c -> temperature 
* 1v80 -> voltage

![lib](https://user-images.githubusercontent.com/86521351/123538196-41ba8e00-d751-11eb-9f12-1c3fb3002cff.PNG)

**Example of and2_0 and and2_2 cell**

![cell_diff](https://user-images.githubusercontent.com/86521351/123538221-67e02e00-d751-11eb-9fab-4876db4d5c7e.PNG)


We can see the difference between the cells in terms of power and area. Leakage power is associated with pins of the cells.

## Hierarchical vs Flat synthesis

Hierarchical synthesis treats the sub module instantiated inside the top module as block box. Whereas in flat synthesis the hierarchy is flattened out. It means all the design will be in a single file.

**Hierarchical synthesis**

![multi_hier](https://user-images.githubusercontent.com/86521351/123538252-a4ac2500-d751-11eb-910e-50e0246a7178.PNG)

**Flat Synthesis**

![flatten_hier](https://user-images.githubusercontent.com/86521351/123538284-c9080180-d751-11eb-8571-060446d0319a.PNG)

**Yosys command for flat synthesis:-**

* read_liberty -lib ../my_lib/lib/ Sky130_fd_sc_hd_tt_025c_1v80.lib
* read_verilog multiple_modules.v
* synth -top multiple_modules
* abc -liberty ../my_lib/lib/ Sky130_fd_sc_hd_tt_025c_1v80.lib
* flatten
* Show

**Benefits of Hierarchical synthesis**

* Replication:- In design where a module is instantiated  multiple times. In synthesis, we can synthesize the module one time and stitch is together rather than synthesizing the same module multiple times.
* Divide & conquer:- For lager design, hierarchical synthesis is used so the synthesis will become fast.


## FLIP FLOPS

* Memory elements, used to store the values
* Make the circuit less glitchy.
* Used to meet the timings.

**Types of flip flop**

**Asynchronous set/reset flop**

![code_async_flop](https://user-images.githubusercontent.com/86521351/123538386-4cc1ee00-d752-11eb-95bd-6119cb8a4a0c.PNG)

![wave_asyn](https://user-images.githubusercontent.com/86521351/123538408-68c58f80-d752-11eb-9dcf-5129af297a73.PNG)

![async_flop_net](https://user-images.githubusercontent.com/86521351/123538398-5cd9cd80-d752-11eb-853b-255623d9d088.PNG)

Flops where the output reset/set whenever the set/reset pin changes are called asynchronous resets. Extra Set/reset is present. 

**synchronous set/reset flop**

![code_sync_flop](https://user-images.githubusercontent.com/86521351/123538452-a0343c00-d752-11eb-9293-4947f21d3d9b.PNG)

![wave_sync](https://user-images.githubusercontent.com/86521351/123538468-b04c1b80-d752-11eb-9361-09d9cd2a4ce3.PNG)

![dff_sync_net](https://user-images.githubusercontent.com/86521351/123538478-b8a45680-d752-11eb-93f8-9967d7066371.PNG)

Synchronous set/reset flop: the output only changes at the edge of the clock even if the set/reset signal changes. For synchronous flop, the set/reset signal functionality is realized along with the d input. 


# Gate level simulation and synthesis simulation mismatches

## GLS

* Gate level simulation
* Netlist obtained after synthesis is used as DUT in the testbench.
* two types of GLS simulation i.e functional and functional + timing aware
* functional used to check the logical behavior of the design is same after the synthesis or not.
* Timing aware simulation can also be performed for checking the timing of the design but for that delay annotation is required.

**GLS using iverilog and gtkwave** 

![gls_sim](https://user-images.githubusercontent.com/86521351/123538624-8ba47380-d753-11eb-90f3-0af5f312bf04.PNG)

## Synthesis & simulation mismatches 

**Missing sensitivity list**

Missing some signals in the sensitivity list may result into synthesis and simulation mismatches.

![mux](https://user-images.githubusercontent.com/86521351/123538742-e6d66600-d753-11eb-9192-1990532c9dcb.PNG)
verilog code of the good and bad mux

![bad_mux_show](https://user-images.githubusercontent.com/86521351/123538761-f2c22800-d753-11eb-8bef-5b6b9fc8e7a4.PNG)


![bad_mux_snip](https://user-images.githubusercontent.com/86521351/123538851-59dfdc80-d754-11eb-9ff9-a06b974767b9.PNG)

In bad_mux. V the sensitivity list only depends upon the sel signal and when we simulate this behavior the y output depend upon the i0 and i1 when sel changes. 

![bad_mux_netlist](https://user-images.githubusercontent.com/86521351/123538787-09687f00-d754-11eb-8b94-5760ff8ad080.PNG)

When we synthesize it and do GLS simulation. We can see the mismatches between two waves. In this case the output depends upon the change in input signals sel, io , i1.

![good_mux_wave](https://user-images.githubusercontent.com/86521351/123538930-bc38dd00-d754-11eb-819b-3e0427a6cd61.PNG)

Waves from the good_mux.v. The sensitivity list now depends upon all input signals. When we simulate this design , output y depends upon input signals sel, i0, i1.

## Blocking vs non-blocking statement

**Non-blocking statement: -** The RHS of the statement is evaluated in active region and updated in the LHS side in non-blocking   region.
**Blocking statement: -** The RHS of the statement is updated in the active region.

Some blocking statement caveat: -

D = (a | b) & c.

![blocking_cveat](https://user-images.githubusercontent.com/86521351/123538993-15a10c00-d755-11eb-99d1-4a5584fd48ad.PNG)

![blocking_cveat_show](https://user-images.githubusercontent.com/86521351/123539021-42552380-d755-11eb-9246-fde3f4552012.PNG)

![blocking_cveat_wave](https://user-images.githubusercontent.com/86521351/123539005-25b8eb80-d755-11eb-938c-616f01b4c333.PNG)
The waveform appears to take in account the previous value of a or b but the desired one are to take current values of a or b.

![blocking_cveat_synth_wave](https://user-images.githubusercontent.com/86521351/123539027-4ed97c00-d755-11eb-92dc-6a517a0f0855.PNG)

Waveforms after synthesis and the output is depending on the current values of the input signals.

## Verilog coding standard

* Use nonblocking statement for sequential logic.
* Use blocking statements for combinational logic.


