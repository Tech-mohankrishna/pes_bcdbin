# 🔱 BCD to Binary Conversion 🔱

A BCD-to-binary conversion converts a BCD number to the equivalent binary representation.
Assume that the input is an 8-bit signal in BCD format (i.e., two BCD digits) and the
output is a 7-bit signal in binary representation.

## code

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
