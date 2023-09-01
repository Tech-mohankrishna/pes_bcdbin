# 🔱 BCD to Binary Conversion 🔱

A BCD-to-binary conversion converts a BCD number to the equivalent binary representation.
Assume that the input is an 8-bit signal in BCD format (i.e., two BCD digits) and the
output is a 7-bit signal in binary representation.

## code

``` v
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

```
## Testbench

``` v
`timescale 1ns / 1ps

module pes_bcdbin_tb;

	// Inputs
	reg clk;
	reg rst_n;
	reg start;
	reg [3:0] dig1;
	reg [3:0] dig0;

	// Outputs
	wire [6:0] bin;
	wire ready;
	wire done_tick;
	integer i;
	// Instantiate the Unit Under Test (UUT)
	pes_bcdbin uut (
		.clk(clk), 
		.rst_n(rst_n), 
		.start(start), 
		.dig1(dig1), 
		.dig0(dig0), 
		.bin(bin), 
		.ready(ready), 
		.done_tick(done_tick)
	);
	always begin
		clk=0;
		#5;
		clk=1;
		#5;
	end

	initial begin
		// Initialize Inputs
		$dumpfile("pes_bcdbin_tb.vcd");
       	 	$dumpvars(0,pes_bcdbin_tb);

		rst_n = 1;
		start = 0;
		dig1 = 0;
		dig0 = 0;
		@(negedge clk); //reset
		rst_n=0;
		@(negedge clk);
		rst_n=1;
		@(negedge clk);
		$display("BCD_input   Binary_output(Decimal_value)");
		for(i=0;i<=99;i=i+1) begin
			start=1;
			dig1=i/10; //second digit of i
			dig0=i-dig1*10; //first digit of i
			wait(done_tick);
			start=0;
			$display("     %0d          %b(%0d)",i,bin,bin);
			repeat(5)@(negedge clk);
		end
		$finish;
		
	end
      
endmodule

```

## Simulation

```
iverilog pes_bcdbin.v pes_bcdbin_tb.v
./a.out
gtkwave pes_bcdbin_tb.vcd
```


![image](https://github.com/Tech-mohankrishna/pes_bcdbin/assets/57735263/e61560e2-f132-46e0-8198-daba76f0148f)

