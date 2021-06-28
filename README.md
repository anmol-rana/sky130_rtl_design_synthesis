![image](https://www.vlsisystemdesign.com/wp-content/uploads/2021/05/Verilog-flyer.png)

# TABLE OF CONTENT
* [Introduction to Verilog RTL Design and Synthesis.](https://github.com/anmol-rana/sky130_rtl_design_synthesis/blob/main/README.md#introduction-to-verilog-rtl-design-and-synthesis)
* [Timing libs, hierarchy vs flat synthesis and effective flop coding style.](https://github.com/anmol-rana/sky130_rtl_design_synthesis/blob/main/README.md#lib-hierarchy-vs-flat-synthesis-and-effective-flop-coding-style)
* [Combinational and Sequential optimizations.](https://github.com/anmol-rana/sky130_rtl_design_synthesis/blob/main/README.md#combinational-and-sequential-optimizations)
* [GLS, blocking vs non-blocking and synthesis simulation mismatches.](https://github.com/anmol-rana/sky130_rtl_design_synthesis/blob/main/README.md#gls-blocking-vs-non-blocking-and-synthesis-simulation-mismatches)
* [If, case, for loop and generate statement.](https://github.com/anmol-rana/sky130_rtl_design_synthesis/blob/main/README.md#if-case-for-loop-and-generate-statement)

# Introduction to Verilog RTL Design and Synthesis

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


## What are different flavors of the same gates and why we need them.

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
But making cells faster result in hold time violation. So, for a circuit to work properly we need to have maximum performance without any hold violations. So that’s why different flavors of same cells are given in the .lib file. These cells are selected depending upon the constraints given to synthesizer.

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

* Replication:- In design where a module is instantiated  multiple times. In oder to synthesis this design, we can synthesize the module one time and stitch is together rather than synthesizing the same module multiple times.
* Divide & conquer:- For lager design, hierarchical synthesis is used so that synthesis will be faster.


## FLIP FLOPS

* Memory elements, used to store the values
* Make the circuit less glitchy.
* Used to meet the timings.

**Types of flip flop**

* Asynchronous set/reset flops
* Synchronous set/reset flops


**Yosys commands for adding flip flops:-**

* read_liberty -lib ../my_lib/lib/ Sky130_fd_sc_hd_tt_025c_1v80.lib
* read_verilog multiple_modules.v
* synth -top multiple_modules
* dfflibmap -liberty ../my_lib/lib/ Sky130_fd_sc_hd_tt_025c_1v80.lib -> somtimes contains flops information in different lib
* abc -liberty ../my_lib/lib/ Sky130_fd_sc_hd_tt_025c_1v80.lib
* Show


**Asynchronous set/reset flop**

![code_async_flop](https://user-images.githubusercontent.com/86521351/123538386-4cc1ee00-d752-11eb-95bd-6119cb8a4a0c.PNG)

![wave_asyn](https://user-images.githubusercontent.com/86521351/123538408-68c58f80-d752-11eb-9dcf-5129af297a73.PNG)

![async_flop_net](https://user-images.githubusercontent.com/86521351/123538398-5cd9cd80-d752-11eb-853b-255623d9d088.PNG)

Flops where the output reset/set whenever the set/reset pin changes are called asynchronous resets. Extra Set/reset is present. 

**Synchronous set/reset flop**

![code_sync_flop](https://user-images.githubusercontent.com/86521351/123538452-a0343c00-d752-11eb-9293-4947f21d3d9b.PNG)

![wave_sync](https://user-images.githubusercontent.com/86521351/123538468-b04c1b80-d752-11eb-9361-09d9cd2a4ce3.PNG)

![dff_sync_net](https://user-images.githubusercontent.com/86521351/123538478-b8a45680-d752-11eb-93f8-9967d7066371.PNG)

Synchronous set/reset flop: the output only changes at the edge of the clock even if the set/reset signal changes. For synchronous flop, the set/reset signal functionality is realized along with the d input. 

## Interesting Optimizations

y= a*2
optimized code for above expression can be written as:-

y= {a,1'b0};

![mul2_show](https://user-images.githubusercontent.com/86521351/123617844-48b2d080-d825-11eb-9aa7-56d7c61351fc.PNG)

After synthesis, the result also shows the optimzation


y = a*9  
say y is 8 bit and a is 3bit  
The above expresion can be written as:-     
y = a*8 + a;     
y = {a,3'b000} + a;     
y = {a,a};    

![mult8_show](https://user-images.githubusercontent.com/86521351/123618387-c70f7280-d825-11eb-8d62-969467387082.PNG)
 
 After synthesis, the results are also similar.

# Combinational and Sequential optimizations

Optimization in the design is required for are and power savings. Different techniques are used for this optimzation by the designer and also by the synthesizer tool.

## Combinational Optimization techniques

* Direct optimization -> sometimes the synthesizer looks for the constant propogation and optimized the same in the genearted netlist
* Boolean optimization -> these optimzation are based some rules. K-map and quine mckluskey are widely used ones

**Yosys commands for optimization:-**

* read_liberty -lib ../my_lib/lib/ Sky130_fd_sc_hd_tt_025c_1v80.lib
* read_verilog multiple_modules.v
* synth -top multiple_modules
* opt_clean -purge -> used for optimization
* abc -liberty ../my_lib/lib/ Sky130_fd_sc_hd_tt_025c_1v80.lib
* Show

**Examples of Combinational optimization**

![opt_chk_code](https://user-images.githubusercontent.com/86521351/123621804-3aff4a00-d829-11eb-969f-2c15610cd455.PNG)

1. For opt_check example  
   y = a?b:0   
   if we expand it, ab + a'0 = ab   
   
   ![opt_check_show](https://user-images.githubusercontent.com/86521351/123622322-b7922880-d829-11eb-95e2-26ee1326467c.PNG)
   
   Synthesizer optimized the code and results in a and gate.

2. For opt_check2     
   y = a ? 1 : b;   
   y = a * 1 + a'b = a + a'b;    
   y = a+b;      
   
   ![opt_check2](https://user-images.githubusercontent.com/86521351/123622356-c24cbd80-d829-11eb-9f21-d3c303252274.PNG)

    After synthesism the final netlist having a or gate  
    But due to some issue in yosys 0.7 version, we are seeing isolation cell.

3. For opt_check3        
   y = a ? (c ? b : 0) : 0;           
   y = a(cb + c'*0): a'*0;            
   y = abc;     

   ![opt_check3_show](https://user-images.githubusercontent.com/86521351/123622382-c7aa0800-d829-11eb-9c41-666a50abedac.PNG)
                 
    After synthesism the final netlist having a 3 input and gate.
    
4. For opt_check4   
   
   y = a ? (b ? (a & c) : c) : (!c)  
   y = a(b(ac) + b'c) + a'c'   
   y = a(abc + b'c) + a'c'   
   y = ac(ab + b') + a'c'   
   y = ac + a'c'   
   y = ~(a ^ b)   
    
   ![opt_check4_wave](https://user-images.githubusercontent.com/86521351/123622451-d98bab00-d829-11eb-8caa-f4369357c2c0.PNG)

   After synthesis, we can see that the netlist only contains a 2 input xnor gate
   
   
## Sequential Logic Optimization

* Sequential propagation optimization
* State optimization:- used for optimizing state machines
* Cloning:- Optimization used in physical design. When a flop is driving two flops which are having large routing area results in larger routing delay.  If we have large positive skew, the driver flop can be cloned twice to reduce the routing delay.
* Retiming : to partition the  combinational logic between the driver and collection flop to match the timing parameters.


**Examples of Sequential optimization**

![dff_const12_code](https://user-images.githubusercontent.com/86521351/123627595-d4c9f580-d82f-11eb-91ba-971a5f6f0be9.PNG)

1. For dff_const1,
   The synthesis will result in a flop because for when reset goes low, the output will be updated at the next clock edge.
   
   ![dff_const1_wave](https://user-images.githubusercontent.com/86521351/123627869-23778f80-d830-11eb-800c-74be96af7ebf.PNG)    
   Waveforms show that the output is updated at the positive edge of the clock    
   
   ![dff_const_show](https://user-images.githubusercontent.com/86521351/123627938-35593280-d830-11eb-9f72-655f4ca4477e.PNG)     
   After synthesis, the netlist contains a flop as expected.

2. For dff_const2,  
   the synthesis will result in a wire tied to 1 because the output remain high all the time
   ![dff_const2_wave](https://user-images.githubusercontent.com/86521351/123627888-283c4380-d830-11eb-8bcd-060f56e0d3ad.PNG)      
    Waveforms show that the output is high continously 
    
    ![dff_const2_show](https://user-images.githubusercontent.com/86521351/123627991-443fe500-d830-11eb-8af2-171f2a5ac6c8.PNG)   
    After synthesis, the netlist contains not flop as expected.
    
3.  For dff_const3, results in 2 flops
  
   ![dff_const3_code](https://user-images.githubusercontent.com/86521351/123628633-f24b8f00-d830-11eb-93e1-bf5c8955e04c.PNG)      

   ![dff_const3_show](https://user-images.githubusercontent.com/86521351/123628983-55d5bc80-d831-11eb-87e3-847986884e9b.PNG)     
    As expected, the netlist contains 2 flops   

4. For dff_const4, q and q1 are always at 1, so synthesis will result in 2 wires at logic high   

   ![dff_const4_code](https://user-images.githubusercontent.com/86521351/123628683-0099ab00-d831-11eb-90de-99d74a4ad5b3.PNG)       

   ![dff_const4_show](https://user-images.githubusercontent.com/86521351/123628867-3343a380-d831-11eb-83d5-fa64c8455c18.PNG)     
    As expected, the netlist contain only 2 wires at logic high     
   
5. For dff_const5, the netlist will contain two flops   
   
   ![dff_const5_code](https://user-images.githubusercontent.com/86521351/123628818-258e1e00-d831-11eb-9d5a-c51c8704026a.PNG)      
   
   ![dff_const5_show](https://user-images.githubusercontent.com/86521351/123628897-3b9bde80-d831-11eb-8161-cc01a69fbcea.PNG)   
    As expected, the netlist contains 2 flops 
   
##  Sequential Optimization for unused outputs

**Examples of optimization for unused outputs**

1. For counter_opt code, the output depends upon the 0th bit of the count variable 
   The q will be high:-      
   count[2]  count[1]  count[0]         q     
   0            0         0             0    
   0            0         1             1     
   0            1         0             0     
   0            1         1             1     
   1            0         0             0     
   1            0         1             1     
   1            1         0             0     
   1            1         1             1     
   
   the ouput q is toggling, so the syhtesis will result in a flop with its input connected to invert of the output.   


  ![count_opt_code](https://user-images.githubusercontent.com/86521351/123630200-dea12800-d832-11eb-8d21-83c5af1d0c24.PNG)   

  ![counter_const1_show](https://user-images.githubusercontent.com/86521351/123630230-e6f96300-d832-11eb-96c9-8d25d06e23c7.PNG)   
  
   As expected, the netlist contains a flop with its input connected to invert of the output

2. For counter_opt2, the output depends upon the 3 bits of count, the netlist will result in 3 flops

  ![counter_const2_code](https://user-images.githubusercontent.com/86521351/123630278-f9739c80-d832-11eb-9138-ca02348538c4.PNG)

  ![counter_const2_show](https://user-images.githubusercontent.com/86521351/123630314-02646e00-d833-11eb-9cf4-aa47db4eac68.PNG)
  
  As expected, the netlist contains 3 flops


#  GLS, blocking vs non-blocking and synthesis simulation mismatches

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

# If, case, for loop and generate statement

## If construct

* Generates priority logic.
* If “Else” part is missed, it results in latches in combinational logic. In sequential it has no effect
* If any of branch of the “if, elseif, else” is true, it will come out and at next triggering event it will start from first branch.

![comp_if_show](https://user-images.githubusercontent.com/86521351/123546086-d1276780-d778-11eb-9064-9669c809b506.PNG)
Synthesis of complete if statement

![incomp_if_code](https://user-images.githubusercontent.com/86521351/123544940-53ad2880-d773-11eb-8f47-a183bb9d5f7b.PNG)
Example of missing else statement

![incomp_if_wave](https://user-images.githubusercontent.com/86521351/123544968-73445100-d773-11eb-8874-5b93b9c41a69.PNG)
Marked region showing output is latched

![incomp_if2_show](https://user-images.githubusercontent.com/86521351/123544961-6c1d4300-d773-11eb-823f-b857c7416a63.PNG)

Synthesis results in D-latche

## Case Construct 

* generates mux logic
* incomplete case result in latche
* partial assignment of outputs result in latch
* bad way of using case result in simulation mismatch




![comp_case](https://user-images.githubusercontent.com/86521351/123546145-06cc5080-d779-11eb-8368-acb25113c63d.PNG)
Synthesis of complete case statement

**Incomplete case result in latche:-**

![incom_case_code](https://user-images.githubusercontent.com/86521351/123546174-23688880-d779-11eb-8413-e7a15ca3c0ec.PNG)
code of incomp case.

![incomp_case_wave](https://user-images.githubusercontent.com/86521351/123546182-2f544a80-d779-11eb-9c57-b7f9cdddd52a.PNG)
Waveforms of incomp case

![incomp_case_show](https://user-images.githubusercontent.com/86521351/123546200-4135ed80-d779-11eb-86df-e57cd1640aef.PNG)
synthesis result in d latch

**partial assignment of outputs result in latch:-**

![partial_case_code](https://user-images.githubusercontent.com/86521351/123546237-6165ac80-d779-11eb-928b-55721ac0c12d.PNG)
code of partial assignement case

![partial_case_show](https://user-images.githubusercontent.com/86521351/123546247-69255100-d779-11eb-870c-92ebf61a033a.PNG)

Sythesis result in d latch

**bad way of using case result in simulation mismatch**

![bad_case_code](https://user-images.githubusercontent.com/86521351/123546289-9bcf4980-d779-11eb-89c6-5c42a95c91fa.PNG)
Example of bad case coding

![bad_case_show](https://user-images.githubusercontent.com/86521351/123546297-aab5fc00-d779-11eb-8f88-66141d0017f6.PNG)
After synthesism it results in 4:1 mux

![bad_case_wave](https://user-images.githubusercontent.com/86521351/123546317-bd303580-d779-11eb-9e5a-8dbe9c6c3fe0.PNG)
waveform with rtl code

![bad_case_net_wave](https://user-images.githubusercontent.com/86521351/123546329-cb7e5180-d779-11eb-96a8-6910aeab57f3.PNG)
waveform with netlist.

This show synthesis simulation mismatch.



## For loop

* evaluate expression
* used inside the always construct
* helps in optimizing verilog code

![mux_gen_code](https://user-images.githubusercontent.com/86521351/123546939-52ccc480-d77c-11eb-81ff-5d87ee112d85.PNG)

 mux code with the help of for construct

![mux_gen_show](https://user-images.githubusercontent.com/86521351/123546952-6415d100-d77c-11eb-9e05-7c4d5ae2e028.PNG)

 synthesis of mux created with the help of for construct
 
![demux_gen_show](https://user-images.githubusercontent.com/86521351/123546991-8b6c9e00-d77c-11eb-9c9d-ac65c38facc5.PNG)

Demux code with the help of for construct

![demux_gen](https://user-images.githubusercontent.com/86521351/123546972-7c85eb80-d77c-11eb-8b52-f32d8edcf7ef.PNG)

Synthesis of the Demux

## Generate

* used for replication of hardware
* Always used outside of always construct
* for inside generate results in replication
* If inside generate results in selection of hardware.


![fa_show](https://user-images.githubusercontent.com/86521351/123547016-a6d7a900-d77c-11eb-86a1-1b194eb15fad.PNG)

Synthesis of fca block
