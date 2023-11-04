# 🔱 BCD to Binary Conversion 🔱

A BCD-to-binary conversion converts a BCD number to the equivalent binary representation.
Assume that the input is an 8-bit signal in BCD format (i.e., two BCD digits) and the
output is a 7-bit signal in binary representation.

## code
'''
`timescale 1ns / 1ps

module pes_bcdbin(
	input clk,rst_n,
	input start,
	input[3:0] dig1,dig0,
	output reg[6:0] bin, //2-digit number takes at most 7 bits
	output reg ready,done_tick
    );
	 //FSM state declarations
	 localparam[1:0] idle=2'd0,
							op=2'd1,
							done=2'd2;
	 reg[1:0] state_reg,state_nxt;
	 reg[6:0] bin_nxt;
	 reg[3:0] dig1_reg,dig1_nxt;
	 reg[3:0] dig0_reg,dig0_nxt;
	 reg[2:0] n_reg,n_nxt; //stores the width of the resulting binary
	 
	 //FSM register operation
	 always @(posedge clk,negedge rst_n) begin
		if(!rst_n) begin
			state_reg<=idle;
			bin<=0;
			dig1_reg<=0;
			dig0_reg<=0;
			n_reg<=0;
		end
		else begin
			state_reg<=state_nxt;
			bin<=bin_nxt;
			dig1_reg<=dig1_nxt;
			dig0_reg<=dig0_nxt;
			n_reg<=n_nxt;
		end
	 end
	 //FSM next-state logic
	 always @* begin
		state_nxt=state_reg;
		bin_nxt=bin;
		dig1_nxt=dig1_reg;
		dig0_nxt=dig0_reg;
		n_nxt=n_reg;
		ready=0;
		done_tick=0;
		case(state_reg)
				idle: begin
							ready=1;
							if(start) begin
								bin_nxt=0;
								dig1_nxt=dig1;
								dig0_nxt=dig0;
								n_nxt=7; //binary has 7 bits of output thus 7 "shifts" are needed
								state_nxt=op;
							end
						end
				  op: begin //special shift-operation for converting bcd to bin.Check the book for more info
							if( {dig1_reg[0],dig0_reg[3:1]} >= 8 ) ///special shift-operation for converting bcd to bin.Check the book for more info
								dig0_nxt= {dig1_reg[0],dig0_reg[3:1]} - 3;
							else dig0_nxt= {dig1_reg[0],dig0_reg[3:1]};
							dig1_nxt=dig1_reg>>1;
							bin_nxt={dig0_reg[0],bin[6:1]};
							n_nxt=n_reg-1;
							if(n_nxt==0) state_nxt=done;
						end
				done: begin
							done_tick=1;
							state_nxt=idle;
						end
			default: state_nxt=idle;
		endcase
	 end
	 


endmodule

'''

## Simulation

```
iverilog pes_bcdbin.v pes_bcdbin_tb.v
./a.out
gtkwave pes_bcdbin_tb.vcd
```

![image](https://github.com/Tech-mohankrishna/pes_bcdbin/assets/57735263/e61560e2-f132-46e0-8198-daba76f0148f)

## Synthesis

```
read_liberty -lib ../pes_asic_class/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib 
read_verilog pes_bcdbin.v
synth -top pes_bcdbin
abc -liberty ../pes_asic_class/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

![image](https://github.com/Tech-mohankrishna/pes_bcdbin/assets/57735263/74d2930d-17e4-4a28-9a34-ebf8cfb41513)


## GLS

![image](https://github.com/Tech-mohankrishna/pes_bcdbin/assets/57735263/496e32ea-a52e-40ae-94cd-e52d8df06241)



## Stage 2 (RTL2GDSII - OPENLANE) - Table of contents



![image](https://github.com/Pavan2280/pes_pd/assets/131603225/24e63c09-da0d-4da6-943c-f54d6abbda85)

#### Design Stages

1) **Synthesis**
   1. **yosys** - Yosys performs RTL synthesis, converting high-level RTL descriptions into gate-level netlists.
   2. **abc** - ABC is used for further optimization and technology mapping to enhance the gate-level design.
   3. **OpenSTA** - OpenSTA conducts static timing analysis to verify if the synthesized design meets timing constraints in the OpenLane flow.

2) **Floorplan & PND**
   1. **init_fp (Initial Floorplan)** - Floorplanning involves determining the initial placement and arrangement of various functional blocks or cells within the chip's       
   layout area.
   2. **ioplacer** - ioplacer is a tool used in the physical design process to place Input/Output (I/O) pads or pins on the chip's boundary.
   3. **pdn** - The PDN is responsible for distributing power (supply voltage) and ground (reference voltage) throughout the chip, ensuring that all components receive the       necessary power supply and maintain stable electrical operation.
   4. **tapcell** - A "tapcell" is a special type of cell used in digital integrated circuit design, particularly in standard cell libraries.It is typically used to create 
   tap connections for the bulk terminals in digital CMOS (Complementary Metal-Oxide-Semiconductor) designs.

3) **Placement**
   1. **Replace** - RePlace is a tool used in the OpenLane flow for cell placement optimization.It focuses on optimizing the placement of standard cells within the chip's   
   layout to achieve better area utilization, timing, and power efficiency.
   2. **Resizer** - Resizer is a tool employed during the physical design process to perform cell resizing and optimization.
   3. **OpenDP (Open Detailed Placement)** - OpenDP, or Open Detailed Placement, is a detailed placement tool used in OpenLane.It is responsible for the fine-grained 
   placement of cells, ensuring that they are precisely positioned within rows and tracks while adhering to design constraints and achieving optimal utilization of the chip's 
   layout area.
   4. **OpenPhysyn (Open Physical Synthesis)** - OpenPhysyn is a tool within OpenLane that performs physical synthesis tasks.It optimizes the logical and physical aspects of 
   the design simultaneously, improving the placement, power, area, and timing by considering both logic and physical information during the optimization process.

4) **CTS**
   1. **TritonCTS** - TritonCTS generates a clock distribution network.

5) **Routing**
   1. **FastRoute** - FastRoute is a global routing tool used in the physical design stage of ASIC chip design.
   2. **TritonRoute** - TritonRoute is a detailed or global routing tool used in the later stages of ASIC chip design, following placement and initial global routing.
   
6) **GDSII Generation**
   1. **Magic** - Magic is primarily a layout tool used for creating and editing IC layouts, and it is often used for digital CMOS design.
   2. **KLayout** - KLayout is primarily used for viewing, editing, and analyzing IC layouts but is not a layout creation tool like Magic.
   
8) **Checks**
   1. **CVC** - CVC is a tool primarily used for verification and debugging of digital designs.
   2. **Netgen** - Netgen is an open-source digital netlist comparison and LVS (Layout vs. Schematic) tool.


Open terminal and type the following commands.
```
cd OpenLane/ 
make mount 
./flow.tcl -interactive
package require openlane 0.9
prep -design openlane/pes_bcdbin -tag run-1
```
![prep_design](https://github.com/Tech-mohankrishna/pes_bcdbin/assets/57735263/dea59dea-885d-4cea-8c28-f6c653e206c1)


**Synthesis**
+ Command to exectue
```
run_synthesis
```
![synthesis](https://github.com/Tech-mohankrishna/pes_bcdbin/assets/57735263/ed688602-b730-48e1-9362-cfc5bd6b539d)


**Floorplan**
+ Command to exectue
```
run_floorplan
```
![flr](https://github.com/Tech-mohankrishna/pes_bcdbin/assets/57735263/34ca151f-a53e-4fd1-b06e-6a0d8180f657)


**Note we need to use libs.tech file so we need to gitclone this https://github.com/hwiiiii/sky130A into pdks folder**
```
git clone https://github.com/hwiiiii/sky130A
```

```
magic -T /home/mohankrishna/sky130A/sky130A/1lbs.tech/magic/sky130A. tech lef read ../../tmp/merged.nom.lef def read pes_bcdbin.def
```
![floor_plan_command](https://github.com/Tech-mohankrishna/pes_bcdbin/assets/57735263/03e6ec54-22f5-4f1e-8817-e2cd88667472)
![floorplan](https://github.com/Tech-mohankrishna/pes_bcdbin/assets/57735263/aeadfc1d-2f12-4603-ab2b-37e693dbbe7f)
![floorplan_result](https://github.com/Tech-mohankrishna/pes_bcdbin/assets/57735263/0c56c6c5-7e33-433b-ac38-ecc7d10086cb)


**Placement**
+ Command to exectue
```
run_placement
```
![placement](https://github.com/Tech-mohankrishna/pes_bcdbin/assets/57735263/0122f17b-5a1f-4fb7-b9a0-79bcf81a8883)


```
magic -T /home/mohankrishna/sky130A/sky130A/1lbs.tech/magic/sky130A. tech lef read ../../tmp/merged.nom.lef def read pes_bcdbin.def
```

![placement](https://github.com/Tech-mohankrishna/pes_bcdbin/assets/57735263/b9d9c041-9ad6-463a-8cde-ec1b73a570e3)


**CTS**
+ Command to exectue
```
run_cts
```
![cts](https://github.com/Tech-mohankrishna/pes_bcdbin/assets/57735263/853305e9-2e11-469f-a594-a7c2bd60e8d7)


**Routing**
+ Command to exectue
```![Screenshot from 2023-11-04 17-35-22](https://github.com/Tech-mohankrishna/pes_bcdbin/assets/57735263/fbdd02cf-414e-4fcd-9ea3-7d0c3bb02871)
![Screenshot from 2023-11-04 17-35-29](https://github.com/Tech-mohankrishna/pes_bcdbin/assets/57735263/82f628e4-486b-48f7-831d-add74b055e87)

run_routing
```
![run_routing](https://github.com/Tech-mohankrishna/pes_bcdbin/assets/57735263/8dfe329e-c02a-449b-8d5d-fbcc1266925e)



```
magic -T /home/mohankrishna/sky130A/sky130A/1lbs.tech/magic/sky130A. tech lef read ../../tmp/merged.nom.lef def read pes_bcdbin.def
```
![routing](https://github.com/Tech-mohankrishna/pes_bcdbin/assets/57735263/dc1ec91c-672d-4ab6-8097-2c580854c7bd)


**These reports generated are given below , after executing run_routing command**

![Screenshot from 2023-11-04 17-36-16](https://github.com/Tech-mohankrishna/pes_bcdbin/assets/57735263/8764bb5e-0fed-4748-9691-552ca56e8786)
![Screenshot from 2023-11-04 17-36-07](https://github.com/Tech-mohankrishna/pes_bcdbin/assets/57735263/b7df8855-d773-470e-9ee0-28706a87300b)
![Screenshot from 2023-11-04 17-35-52](https://github.com/Tech-mohankrishna/pes_bcdbin/assets/57735263/c554c34e-57b1-42b1-ac25-c2d55be21c19)
![Screenshot from 2023-11-04 17-35-44](https://github.com/Tech-mohankrishna/pes_bcdbin/assets/57735263/4c1eed3f-ddbe-4aca-8fa7-ba9254a7cf5c)
![Screenshot from 2023-11-04 17-35-38](https://github.com/Tech-mohankrishna/pes_bcdbin/assets/57735263/0d11fea0-3b72-438b-a118-8bc302bdbafa)
![Screenshot from 2023-11-04 17-35-14](https://github.com/Tech-mohankrishna/pes_bcdbin/assets/57735263/20a3e154-baa5-4a9f-b85a-5da24f55d41f)
![power_report_new](https://github.com/Tech-mohankrishna/pes_bcdbin/assets/57735263/25857549-4c41-4076-bbfd-8b8ae40f4a02)
![area_report_new](https://github.com/Tech-mohankrishna/pes_bcdbin/assets/57735263/8fe1db38-ae1b-4652-b6f0-16c5912c502d)
![summary_report_new](https://github.com/Tech-mohankrishna/pes_bcdbin/assets/57735263/ca2926cf-a897-4d59-91b5-d93b6d1d75b9)
![skew_report_new](https://github.com/Tech-mohankrishna/pes_bcdbin/assets/57735263/2658b53b-027d-4a78-9a95-7a90a39e13a4)




#### Statistics
- Area = 1230 um2
- Internal Power = 2.81e-04 W
- Switching Power = 1.81e-04 W
- Leakage Power = 6.73e-10 W
- Total Power = 4.61e-04 W




