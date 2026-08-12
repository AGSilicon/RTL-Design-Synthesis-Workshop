# VSD RTL Design and Synthesis Workshop --- Day 05

# My Day 05 Learning

## 1. What I Learned Today

In Day 05, I continued learning how RTL coding style affects the
hardware inferred by synthesis. I worked with incomplete conditional
statements, inferred latches, `case` statements, procedural loops,
generate loops, multiplexers, demultiplexers, and a Ripple Carry Adder.

The main ideas I learned were:

-   `if`, `else if`, and `else` create priority-based conditional logic.
-   An incomplete combinational `if` can infer a latch.
-   An incomplete `case` can infer a latch.
-   Every combinational output should receive a defined value on every
    path.
-   A default assignment is a useful way to avoid incomplete
    assignments.
-   `case` is useful for selector-based logic.
-   A procedural `for` loop can describe repetitive combinational
    hardware such as wide MUX/DEMUX logic.
-   A `generate for` loop is used to replicate hardware structures.
-   A Ripple Carry Adder can be constructed by repeatedly instantiating
    full adders.
-   I should always think about the hardware that synthesis will infer
    from my RTL.

------------------------------------------------------------------------

## 2. Thinking in Terms of Hardware

One of the most important things I reinforced today is:

> I should always look at RTL code from the point of view of the
> hardware that synthesis will infer.

For example:

``` verilog
if (sel)
    y = a;
```

I should ask what happens when `sel = 0`. If no value is assigned to
`y`, the hardware may need to retain the previous value of `y`. In
combinational logic this can result in an inferred latch.

Therefore, while writing RTL I need to consider every possible input
condition.

------------------------------------------------------------------------

## 3. `if` Statement and Priority Logic

A priority structure can be written as:

``` verilog
always @(*)
begin
    if (cond1)
        y = a;
    else if (cond2)
        y = b;
    else
        y = c;
end
```

The conditions are checked in priority order:

``` text
cond1?
 ├── yes → y = a
 └── no  → cond2?
             ├── yes → y = b
             └── no  → y = c
```

If `cond1` is true, the later `else if` is not selected. This is why an
`if / else if / else` chain represents priority-based selection.

------------------------------------------------------------------------

## 4. Incomplete `if` and Inferred Latch

I studied the following coding style:

``` verilog
always @(*)
begin
    if (cond1)
        y = a;
    else if (cond2)
        y = b;
end
```

There is no final `else`. If both conditions are false, `y` receives no
new value.

``` text
cond1 = 0
cond2 = 0
       ↓
No assignment to y
       ↓
Previous value must be retained
       ↓
Latch can be inferred
```

This is a bad coding style when I intend to describe pure combinational
logic.

A complete version is:

``` verilog
always @(*)
begin
    if (cond1)
        y = a;
    else if (cond2)
        y = b;
    else
        y = 1'b0;
end
```

Now every condition has a defined output.

------------------------------------------------------------------------

## 5. Default Assignment Technique

I also learned that I can give an output a default value before
conditional logic:

``` verilog
always @(*)
begin
    y = 1'b0;

    if (cond1)
        y = a;
    else if (cond2)
        y = b;
end
```

The default assignment guarantees that `y` is assigned even when neither
condition is true.

This is especially useful in larger combinational blocks.

------------------------------------------------------------------------

## 6. Counter Coding

I reviewed a clocked counter structure:

``` verilog
module counter (
    input clk,
    input reset,
    input en,
    output reg [2:0] count
);

always @(posedge clk or posedge reset)
begin
    if (reset)
        count <= 3'b000;
    else if (en)
        count <= count + 1'b1;
end

endmodule
```

The hardware interpretation is:

``` text
              reset?
             /      \\
          yes        no
           |          |
        count=0      en?
                      |
                     yes
                      |
                count + 1
```

The important point I reinforced is that RTL describes hardware
behavior, so I need to understand what storage elements and
combinational logic the code represents.

------------------------------------------------------------------------

## 7. `case` Statement

I learned that a `case` statement is useful when a selector chooses one
of several alternatives.

``` verilog
always @(*)
begin
    case (sel)
        2'b00: y = c1;
        2'b01: y = c2;
        2'b10: y = c3;
        2'b11: y = c4;
    endcase
end
```

For a 2-bit selector:

``` text
00 → c1
01 → c2
10 → c3
11 → c4
```

This can describe a multiplexer.

------------------------------------------------------------------------

## 8. Incomplete `case`

An incomplete case can also infer a latch:

``` verilog
always @(*)
begin
    case (sel)
        2'b00: y = c1;
        2'b01: y = c2;
    endcase
end
```

For `sel = 2'b10` or `2'b11`, no assignment is made to `y`.

Therefore:

``` text
Missing selector condition
        ↓
No new y value
        ↓
Previous y retained
        ↓
Possible latch inference
```

A `default` branch can define the remaining behavior:

``` verilog
always @(*)
begin
    case (sel)
        2'b00: y = c1;
        2'b01: y = c2;
        2'b10: y = c3;
        default: y = c4;
    endcase
end
```

------------------------------------------------------------------------

## 9. Partial Assignments Inside `case`

I learned that even when all selector values are covered, a latch can
still be inferred if different branches assign different outputs.

For example:

``` verilog
reg [1:0] sel;
reg x, y;

always @(*)
begin
    case (sel)
        2'b00:
        begin
            x = a;
            y = b;
        end

        2'b01:
        begin
            x = c;
            // y is not assigned
        end

        default:
        begin
            x = d;
            y = b;
        end
    endcase
end
```

In the `2'b01` branch, `y` is not assigned. Therefore `y` may retain its
previous value and a latch can be inferred for `y`.

The rule I learned is:

> Every output used by a combinational `case` should be assigned in
> every possible path.

A robust style is:

``` verilog
always @(*)
begin
    x = 1'b0;
    y = 1'b0;

    case (sel)
        2'b00:
        begin
            x = a;
            y = b;
        end

        2'b01:
        begin
            x = c;
            y = d;
        end

        default:
        begin
            x = 1'b0;
            y = 1'b0;
        end
    endcase
end
```

------------------------------------------------------------------------

## 10. `if-else` vs `case`

I compared both styles.

### Priority-style logic

``` verilog
if (cond1)
    y = a;
else if (cond2)
    y = b;
else
    y = c;
```

This naturally expresses priority.

### Selector-style logic

``` verilog
case (sel)
    2'b00: y = a;
    2'b01: y = b;
    2'b10: y = c;
    2'b11: y = d;
endcase
```

This is convenient when a selector chooses among alternatives.

The coding style should match the intended hardware.

------------------------------------------------------------------------

## 11. Day 05 Latch Labs

I worked with the examples using the `incomp` prefix:

``` text
incomp_if.v
incomp_if2.v
incomp_case.v
comp_case.v
bad_case.v
```

The purpose of the experiments was to simulate the RTL, synthesize it,
inspect the inferred hardware, and understand how incomplete conditions
or assignments cause latches.

### `incomp_if.v`

Simulation:

``` bash
cd ~/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files
iverilog incomp_if.v tb_incomp_if.v
./a.out
gtkwave tb_incomp_if.vcd
```

Synthesis:

``` bash
yosys
```

``` yosys
read_verilog incomp_if.v
synth -top incomp_if
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

I used the waveform and synthesized view to understand the inferred
latch.

### `incomp_if2.v`

This example contains an `if` and `else if` structure without a final
`else`.

``` bash
iverilog incomp_if2.v tb_incomp_if2.v
./a.out
gtkwave tb_incomp_if2.vcd
```

Synthesis:

``` yosys
read_verilog incomp_if2.v
synth -top incomp_if2
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

### `incomp_case.v`

Simulation:

``` bash
iverilog incomp_case.v tb_incomp_case.v
./a.out
gtkwave tb_incomp_case.vcd
```

Synthesis:

``` yosys
read_verilog incomp_case.v
synth -top incomp_case
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

### `comp_case.v`

I used the complete case version to compare it with the incomplete case
implementation.

``` bash
iverilog comp_case.v tb_comp_case.v
./a.out
gtkwave tb_comp_case.vcd
```

Synthesis:

``` yosys
read_verilog comp_case.v
synth -top comp_case
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

### `bad_case.v`

I also studied partial case assignments.

``` bash
iverilog bad_case.v tb_bad_case.v
./a.out
gtkwave tb_bad_case.vcd
```

Synthesis and netlist generation:

``` yosys
read_verilog bad_case.v
synth -top bad_case
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog -noattr bad_case_net.v
```

Gate-level simulation:

``` bash
iverilog \
../my_lib/verilog_model/primitives.v \
../my_lib/verilog_model/sky130_fd_sc_hd.v \
bad_case_net.v \
tb_bad_case.v
./a.out
gtkwave tb_bad_case.vcd
```

------------------------------------------------------------------------

## 12. Procedural `for` Loop

I then learned how a procedural `for` loop can describe repetitive
combinational hardware.

For a 32-input multiplexer:

``` verilog
integer i;

always @(*)
begin
    y = 1'b0;

    for (i = 0; i < 32; i = i + 1)
    begin
        if (i == sel)
            y = inp[i];
    end
end
```

For a 256-input multiplexer:

``` verilog
integer i;

always @(*)
begin
    y = 1'b0;

    for (i = 0; i < 256; i = i + 1)
    begin
        if (i == sel)
            y = inp[i];
    end
end
```

The loop is not a software loop executing sequentially in the fabricated
circuit. Synthesis interprets the repeated structure and creates the
corresponding combinational hardware.

------------------------------------------------------------------------

## 13. Loop-Based Demultiplexer

I worked with an 1-to-8 demultiplexer using a loop.

``` verilog
module demux_generate (
    output o0,
    output o1,
    output o2,
    output o3,
    output o4,
    output o5,
    output o6,
    output o7,
    input [2:0] sel,
    input i
);

reg [7:0] y_int;
integer k;

assign {o7,o6,o5,o4,o3,o2,o1,o0} = y_int;

always @(*)
begin
    y_int = 8'b0;

    for (k = 0; k < 8; k = k + 1)
    begin
        if (k == sel)
            y_int[k] = i;
    end
end

endmodule
```

The default:

``` verilog
y_int = 8'b0;
```

ensures that all outputs start at zero for each evaluation. The selected
bit is then driven by `i`.

The selection is:

``` text
sel = 000 → o0
sel = 001 → o1
sel = 010 → o2
sel = 011 → o3
sel = 100 → o4
sel = 101 → o5
sel = 110 → o6
sel = 111 → o7
```

------------------------------------------------------------------------

## 14. Procedural `for` vs Generate `for`

This distinction was important for me.

### Procedural `for`

Used inside an `always` block:

``` verilog
always @(*)
begin
    for (i = 0; i < 8; i = i + 1)
    begin
        // procedural combinational description
    end
end
```

It is useful for describing repeated logic behavior.

### Generate `for`

Used outside procedural blocks:

``` verilog
genvar i;

generate
    for (i = 0; i < 8; i = i + 1)
    begin
        // hardware instance
    end
endgenerate
```

It is used to replicate hardware structures.

------------------------------------------------------------------------

## 15. Generate Loop Example

I learned that repeated hardware can be instantiated using `generate`.

``` verilog
genvar i;

generate
    for (i = 0; i < 8; i = i + 1)
    begin
        and u_and (
            .a(in1[i]),
            .b(in2[i]),
            .y(y[i])
        );
    end
endgenerate
```

This represents eight AND-gate instances, one for each bit.

Without generate, I would need to write separate instances manually.
Generate makes the repeated hardware structure compact and easier to
maintain.

------------------------------------------------------------------------

## 16. Ripple Carry Adder

I then learned the structure of a Ripple Carry Adder (RCA).

For example:

``` text
  110
+ 011
-----
 1001
```

Each full adder produces a sum bit and a carry-out. The carry-out of one
stage becomes the carry-in of the next stage.

``` text
FA0 → carry0 → FA1 → carry1 → FA2 → carry2 → FA3
```

Therefore, for an N-bit RCA:

``` text
N full adders are required.
```

For an 8-bit RCA, I need 8 full adders.

------------------------------------------------------------------------

## 17. Full Adder

The full-adder module is:

``` verilog
module fa (
    input a,
    input b,
    input c,
    output co,
    output sum
);

assign {co, sum} = a + b + c;

endmodule
```

The inputs are:

``` text
a       → first operand bit
b       → second operand bit
c       → carry-in
```

The outputs are:

``` text
sum     → sum bit
co      → carry-out
```

The concatenation:

``` verilog
{co, sum}
```

holds the two-bit result of the addition.

------------------------------------------------------------------------

## 18. 8-bit RCA Code

The RCA I worked with can be written as:

``` verilog
module rca (
    input  [7:0] num1,
    input  [7:0] num2,
    output [8:0] sum
);

wire [7:0] int_sum;
wire [7:0] int_co;

genvar i;

generate
    for (i = 1; i < 8; i = i + 1)
    begin
        fa u_fa (
            .a   (num1[i]),
            .b   (num2[i]),
            .c   (int_co[i-1]),
            .co  (int_co[i]),
            .sum (int_sum[i])
        );
    end
endgenerate

fa u_fa_0 (
    .a   (num1[0]),
    .b   (num2[0]),
    .c   (1'b0),
    .co  (int_co[0]),
    .sum (int_sum[0])
);

assign sum[7:0] = int_sum;
assign sum[8]   = int_co[7];

endmodule
```

The first full adder receives a carry-in of zero. Every following full
adder receives the carry from the previous stage.

------------------------------------------------------------------------

## 19. Why RCA Has 9-bit Output

Two 8-bit values can generate a carry beyond bit 7.

Therefore:

``` verilog
input  [7:0] num1;
input  [7:0] num2;
output [8:0] sum;
```

The lower eight bits are:

``` verilog
assign sum[7:0] = int_sum;
```

and the final carry is:

``` verilog
assign sum[8] = int_co[7];
```

So:

``` text
sum[8]   → final carry
sum[7:0] → sum bits
```

------------------------------------------------------------------------

## 20. RCA Simulation

Because `rca.v` instantiates the `fa` module, both files must be
compiled with the testbench.

``` bash
cd ~/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files
iverilog fa.v rca.v tb_rca.v
./a.out
gtkwave tb_rca.vcd
```

If I compile only `rca.v` and the testbench without `fa.v`, the
simulator does not have the definition of the `fa` module.

This reinforced the importance of including all required modules in a
hierarchical Verilog design.

------------------------------------------------------------------------

## 21. RCA Synthesis

I synthesize the RCA using Yosys:

``` bash
yosys
```

``` yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog fa.v
read_verilog rca.v
synth -top rca
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog -noattr rca_net.v
```

The generated netlist is:

``` text
rca_net.v
```

------------------------------------------------------------------------

## 22. RCA Gate-Level Simulation

I can simulate the synthesized RCA using the Sky130 Verilog cell models:

``` bash
iverilog \
../my_lib/verilog_model/primitives.v \
../my_lib/verilog_model/sky130_fd_sc_hd.v \
rca_net.v \
tb_rca.v
```

Then:

``` bash
./a.out
gtkwave tb_rca.vcd
```

The comparison is:

``` text
RTL RCA waveform
       vs
Gate-level RCA waveform
```

------------------------------------------------------------------------

## 23. MUX Loop Lab --- `mux_generate.v`

A clean version of the 4-to-1 MUX loop implementation is:

``` verilog
module mux_generate (
    input i0,
    input i1,
    input i2,
    input i3,
    input [1:0] sel,
    output reg y
);

wire [3:0] i_int;
assign i_int = {i3, i2, i1, i0};

integer k;

always @(*)
begin
    y = 1'b0;

    for (k = 0; k < 4; k = k + 1)
    begin
        if (k == sel)
            y = i_int[k];
    end
end

endmodule
```

Simulation:

``` bash
iverilog mux_generate.v tb_mux_generate.v
./a.out
gtkwave tb_mux_generate.vcd
```

Synthesis:

``` bash
yosys
```

``` yosys
read_verilog mux_generate.v
synth -top mux_generate
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
write_verilog -noattr mux_generate_net.v
```

------------------------------------------------------------------------

## 24. DEMUX Loop Lab --- `demux_generate.v`

Simulation:

``` bash
iverilog demux_generate.v tb_demux_generate.v
./a.out
gtkwave tb_demux_generate.vcd
```

Synthesis:

``` bash
yosys
```

``` yosys
read_verilog demux_generate.v
synth -top demux_generate
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
write_verilog -noattr demux_generate_net.v
```

I compared the loop-based demultiplexer with the case-based
implementation and observed their waveforms.

------------------------------------------------------------------------

## 25. My Day 05 RTL-to-Hardware Flow

The complete flow I followed is:

``` text
RTL source
   │
   ▼
Functional simulation
   │
   ▼
GTKWave waveform
   │
   ▼
Yosys synthesis
   │
   ▼
ABC technology mapping
   │
   ▼
Gate-level Verilog netlist
   │
   ▼
Sky130 cell models + netlist + testbench
   │
   ▼
Gate-Level Simulation
   │
   ▼
GTKWave comparison
```

For latch-related examples, I also used:

``` yosys
show
```

to inspect the synthesized structure.

------------------------------------------------------------------------

## 26. Important Paths in My Workshop

My Day 05 RTL files are under:

``` text
~/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files/
```

Important files include:

``` text
incomp_if.v
incomp_if2.v
incomp_case.v
comp_case.v
bad_case.v
mux_generate.v
demux_generate.v
rca.v
fa.v
```

Testbenches include:

``` text
tb_incomp_if.v
tb_incomp_if2.v
tb_incomp_case.v
tb_comp_case.v
tb_bad_case.v
tb_mux_generate.v
tb_demux_generate.v
tb_rca.v
```

Sky130 Liberty:

``` text
~/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

Sky130 Verilog models:

``` text
~/VLSI/sky130RTLDesignAndSynthesisWorkshop/my_lib/verilog_model/primitives.v
~/VLSI/sky130RTLDesignAndSynthesisWorkshop/my_lib/verilog_model/sky130_fd_sc_hd.v
```

------------------------------------------------------------------------

## 27. Important Yosys Commands I Used

Read RTL:

``` yosys
read_verilog <design>.v
```

Read Liberty:

``` yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

Synthesize:

``` yosys
synth -top <top_module>
```

Technology mapping:

``` yosys
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

Inspect:

``` yosys
show
```

Write netlist:

``` yosys
write_verilog -noattr <design>_net.v
```

------------------------------------------------------------------------

## 28. Important Simulation Commands I Used

Single-module RTL simulation:

``` bash
iverilog <design>.v <testbench>.v
./a.out
```

Multi-module simulation:

``` bash
iverilog fa.v rca.v tb_rca.v
./a.out
```

Gate-level simulation:

``` bash
iverilog \
../my_lib/verilog_model/primitives.v \
../my_lib/verilog_model/sky130_fd_sc_hd.v \
<netlist>.v \
<testbench>.v
./a.out
```

Waveform:

``` bash
gtkwave <vcd_file>
```

------------------------------------------------------------------------

## 29. What I Learned About Latches

I learned that an inferred latch is commonly caused by incomplete
combinational assignments.

For an intended combinational block, I should make sure:

``` text
Every possible condition
        ↓
Output gets a value
```

Instead of:

``` text
Some condition
        ↓
No assignment
        ↓
Previous output retained
        ↓
Latch
```

This applies to both `if` and `case` coding.

------------------------------------------------------------------------

## 30. What I Learned About Loops

A Verilog loop does not mean that hardware is executing a software loop
one iteration after another.

For example:

``` verilog
for (i = 0; i < 32; i = i + 1)
```

is interpreted by synthesis as a repeated hardware description.

This makes loops useful for describing wide MUX and DEMUX structures
without writing every connection manually.

------------------------------------------------------------------------

## 31. What I Learned About Generate

I learned that `generate` is especially useful when I need multiple
instances of the same hardware module.

The basic pattern is:

``` verilog
genvar i;

generate
    for (i = 0; i < N; i = i + 1)
    begin
        // repeated hardware instance
    end
endgenerate
```

The RCA is a practical example because the same full-adder module is
replicated for each bit.

------------------------------------------------------------------------

## 32. Day 05 Practical Checklist

-   [ ] Understand `if` as priority-based conditional logic.
-   [ ] Understand incomplete `if`.
-   [ ] Understand inferred latches.
-   [ ] Understand the importance of a final `else`.
-   [ ] Understand default assignments.
-   [ ] Understand `case` statements.
-   [ ] Understand incomplete `case` statements.
-   [ ] Understand the use of `default`.
-   [ ] Understand partial assignments inside `case` branches.
-   [ ] Compare `if-else` and `case`.
-   [ ] Simulate `incomp_if.v`.
-   [ ] Synthesize `incomp_if.v`.
-   [ ] Simulate `incomp_if2.v`.
-   [ ] Synthesize `incomp_if2.v`.
-   [ ] Simulate `incomp_case.v`.
-   [ ] Synthesize `incomp_case.v`.
-   [ ] Compare with `comp_case.v`.
-   [ ] Work with `bad_case.v`.
-   [ ] Generate a gate-level netlist.
-   [ ] Run gate-level simulation.
-   [ ] Understand procedural `for` loops.
-   [ ] Build a loop-based MUX.
-   [ ] Build a loop-based DEMUX.
-   [ ] Compare loop and case implementations.
-   [ ] Understand `generate for`.
-   [ ] Understand hardware replication.
-   [ ] Build the full adder.
-   [ ] Build the RCA using generated full adders.
-   [ ] Simulate the RCA.
-   [ ] Synthesize the RCA.
-   [ ] Run RCA gate-level simulation.

------------------------------------------------------------------------

## 33. My Day 05 Learning Summary

### Conditional logic

I learned that an incomplete `if` can infer a latch when an output is
not assigned for every possible condition.

### Case logic

I learned that an incomplete `case` can also infer a latch. A `default`
branch or complete assignments can prevent this when the intended
hardware is combinational.

### Partial assignments

I learned that covering every selector value is not enough if every
output is not assigned in every branch.

### Priority

I learned that an `if / else if / else` chain represents priority-based
selection.

### Procedural loops

I learned that a procedural `for` loop can compactly describe wide
repetitive combinational logic such as MUX and DEMUX structures.

### Generate loops

I learned that a `generate for` loop is used to replicate hardware
structures.

### Ripple Carry Adder

I learned how full adders are connected through carry signals to form an
RCA, and how `generate` can be used to create repeated full-adder
instances.

### Verification

I continued following:

``` text
RTL
 ↓
Functional Simulation
 ↓
Synthesis
 ↓
Gate-Level Netlist
 ↓
Gate-Level Simulation
 ↓
Waveform Comparison
```

------------------------------------------------------------------------

# 34. Final Understanding

My overall understanding of Day 05 is:

``` text
                 RTL Coding
                     │
          ┌──────────┼───────────┐
          │          │           │
          ▼          ▼           ▼
         IF         CASE        LOOP
          │          │           │
          ▼          ▼           ▼
       Priority   Selection   Repetition
          │          │           │
          └──────────┼───────────┘
                     ▼
              Hardware Inference
                     │
          ┌──────────┴──────────┐
          │                     │
       Complete              Incomplete
          │                     │
          ▼                     ▼
   Intended hardware        Latch / issue
```

For repeated hardware:

``` text
                 Generate
                    │
                    ▼
          Replicate hardware
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
         FA        FA        FA
          │         │         │
          └─────────┼─────────┘
                    ▼
                  RCA
```

The main lesson I take from Day 05 is:

> **I should write RTL by thinking about the hardware that I want
> synthesis to infer. Every combinational path should have a defined
> output, and repeated hardware should be described using the
> appropriate procedural loop or generate construct.**
