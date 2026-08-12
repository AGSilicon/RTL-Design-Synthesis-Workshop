# VSD RTL Design and Synthesis Workshop --- Day 03

## My Day 03 Learning

### 1. Introduction to Logic Optimization

In Day 03, I learned how synthesis tools optimize RTL to obtain a more
efficient hardware implementation.

The main idea I understood is:

``` text
RTL
 ↓
Logic Optimization
 ↓
Optimized Logic
 ↓
Technology Mapping
 ↓
Standard Cells
```

Logic optimization means simplifying the design while maintaining the
same required functionality.

The main goals are:

-   Reduce unnecessary logic
-   Reduce area
-   Reduce power
-   Improve the overall implementation

The basic idea is to **squeeze the logic and obtain the most optimized
design possible**.

------------------------------------------------------------------------

# 2. Types of Logic Optimization

I learned that logic optimization can be broadly considered in two
areas:

``` text
Logic Optimization
│
├── Combinational Logic Optimization
│
└── Sequential Logic Optimization
```

For combinational logic, I studied:

-   Constant propagation
-   Direct optimization
-   Boolean logic optimization

For sequential logic, I studied:

-   Sequential constant propagation
-   State optimization
-   Retiming
-   Sequential logic cloning

------------------------------------------------------------------------

# 3. Combinational Logic Optimization

Combinational logic depends only on the present input values.

Examples include:

-   AND gates
-   OR gates
-   Multiplexers
-   Combinational arithmetic logic

I learned that the synthesizer can simplify combinational logic
automatically.

For example:

``` verilog
assign y = a & 1'b0;
```

Since:

``` text
a AND 0 = 0
```

the output can simply be:

``` verilog
assign y = 1'b0;
```

The input `a` is no longer required for the output.

------------------------------------------------------------------------

# 4. Constant Propagation

Constant propagation is the process of propagating known constant values
through the logic.

For example:

``` verilog
assign y = a & 1'b0;
```

can be reduced to:

``` verilog
assign y = 1'b0;
```

Similarly:

``` verilog
assign y = a | 1'b1;
```

can be reduced to:

``` verilog
assign y = 1'b1;
```

The important point I learned is:

``` text
Known constant
     ↓
Propagate through logic
     ↓
Unnecessary logic can be removed
```

This reduces the amount of hardware required.

------------------------------------------------------------------------

# 5. Direct Optimization

I learned that simple redundant expressions can also be optimized.

For example:

``` verilog
assign y = a & a;
```

is equivalent to:

``` verilog
assign y = a;
```

Similarly:

``` verilog
assign y = a | a;
```

is equivalent to:

``` verilog
assign y = a;
```

Therefore:

``` text
Original logic
     ↓
Identify redundancy
     ↓
Remove redundant operation
     ↓
Simpler logic
```

------------------------------------------------------------------------

# 6. Boolean Logic Optimization

I revised Boolean identities that can be used to simplify digital logic.

Important identities include:

``` text
A + 0 = A
A + 1 = 1

A · 0 = 0
A · 1 = A

A + A = A
A · A = A

A + A' = 1
A · A' = 0
```

These identities help reduce the number of required logic operations.

For example:

``` text
Y = A + 0
```

becomes:

``` text
Y = A
```

and:

``` text
Y = A · 1
```

also becomes:

``` text
Y = A
```

------------------------------------------------------------------------

# 7. K-Map

I also studied the **Karnaugh Map (K-map)** as a method of Boolean logic
minimization.

The basic process is:

``` text
Truth Table
     ↓
K-Map
     ↓
Group adjacent terms
     ↓
Simplified Boolean Expression
```

The purpose is to obtain an equivalent Boolean expression with fewer
logic terms.

This helps in reducing the hardware required to implement a
combinational function.

------------------------------------------------------------------------

# 8. Quine-McCluskey Method

I also learned about the **Quine-McCluskey method** for Boolean
minimization.

The basic idea is:

``` text
Boolean Function
       ↓
Minterms
       ↓
Grouping
       ↓
Combining terms
       ↓
Simplified expression
```

It provides a systematic method of Boolean minimization.

------------------------------------------------------------------------

# 9. Using Yosys for Optimization

I learned that Yosys performs synthesis and optimization of the RTL
before the final technology-mapped implementation.

My basic Yosys flow for combinational optimization is:

``` text
Read Liberty
     ↓
Read Verilog
     ↓
Synthesize
     ↓
Clean unused logic
     ↓
Technology Mapping
     ↓
Show the design
```

I use the following working directory:

``` bash
cd ~/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files
```

------------------------------------------------------------------------

# 10. Combinational Optimization Lab

The main files I worked with are:

``` text
opt_check.v
opt_check2.v
opt_check3.v
opt_check4.v
```

The corresponding testbench files include:

``` text
tb_opt_check.v
tb_opt_check2.v
tb_opt_check3.v
```

------------------------------------------------------------------------

# 11. `opt_check.v` --- Yosys Flow

I can run the optimization flow using:

``` bash
cd ~/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files
yosys
```

Inside Yosys:

``` yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog opt_check.v
synth -top opt_check
opt_clean -purge
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

The purpose of each command is:

### Read the technology library

``` yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

This loads the Sky130 standard-cell information.

### Read the RTL

``` yosys
read_verilog opt_check.v
```

This loads my Verilog design.

### Synthesize

``` yosys
synth -top opt_check
```

This synthesizes the selected top-level module.

### Clean unused logic

``` yosys
opt_clean -purge
```

This removes unused or redundant portions of the design.

### Technology mapping

``` yosys
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

This maps the logic using the Sky130 standard-cell library.

### View the design

``` yosys
show
```

This allows me to inspect the resulting implementation.

------------------------------------------------------------------------

# 12. `opt_check2.v`, `opt_check3.v` and `opt_check4.v`

I follow the same basic procedure for the other optimization examples.

For `opt_check2.v`:

``` yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog opt_check2.v
synth -top opt_check2
opt_clean -purge
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

For `opt_check3.v`:

``` yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog opt_check3.v
synth -top opt_check3
opt_clean -purge
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

For `opt_check4.v`:

``` yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog opt_check4.v
synth -top opt_check4
opt_clean -purge
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

The important thing I learned from these exercises is that different RTL
descriptions can provide different optimization opportunities even when
they are intended to implement related functionality.

------------------------------------------------------------------------

# 13. Hierarchical Design Optimization

I also worked with multiple-module designs.

The relevant files include:

``` text
multiple_modules.v
multiple_module_flat.v
multiple_module_hier.v
multiple_module_opt.v
multiple_module_opt2.v
```

A hierarchical design can be represented as:

``` text
Top Module
│
├── Submodule 1
│
└── Submodule 2
```

The synthesis process can operate on this structure.

------------------------------------------------------------------------

# 14. Flattening

I learned that the hierarchy can be flattened using:

``` yosys
flatten
```

A typical flow is:

``` yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog multiple_modules.v
synth -top multiple_modules
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
flatten
opt_clean -purge
show
```

After flattening:

``` text
Hierarchical modules
       ↓
     flatten
       ↓
Combined logic
       ↓
Further optimization
```

I learned that flattening removes the module hierarchy from the design
representation and makes the lower-level logic part of the higher-level
design.

------------------------------------------------------------------------

# 15. Sequential Logic Optimization

After combinational optimization, I learned about optimization of
sequential logic.

The main techniques I studied are:

``` text
Sequential Logic Optimization
│
├── Sequential Constant Propagation
├── State Optimization
├── Retiming
└── Sequential Logic Cloning
```

------------------------------------------------------------------------

# 16. Sequential Constant Propagation

Constant propagation is not limited to combinational logic.

It can also be applied to sequential logic.

If a flip-flop is proven to always contain a constant value, the
synthesis tool can potentially remove unnecessary sequential hardware.

Conceptually:

``` text
DFF
 │
 └── Always produces a constant
              ↓
       Constant logic
```

This can reduce the number of flip-flops and associated logic.

------------------------------------------------------------------------

# 17. DFF Constant Optimization

I worked with the following files:

``` text
dff_const1.v
dff_const2.v
dff_const3.v
dff_const4.v
dff_const5.v
```

These examples helped me understand how synthesis can recognize constant
behavior in sequential circuits.

------------------------------------------------------------------------

# 18. Simulating `dff_const3.v`

I can simulate the design using:

``` bash
cd ~/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files
iverilog dff_const3.v tb_dff_const3.v
./a.out
```

The generated VCD can be viewed using:

``` bash
gtkwave tb_dff_const3.vcd
```

The waveform helps me verify the behavior of the RTL design.

------------------------------------------------------------------------

# 19. Synthesizing `dff_const3.v`

The synthesis flow is:

``` bash
cd ~/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files
yosys
```

Then:

``` yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_const3.v
synth -top const3
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

Before using `synth -top`, I should verify the actual top-module name in
the source file.

For example:

``` bash
grep -n "^module" dff_const3.v
```

This prevents errors caused by using an incorrect top-module name.

------------------------------------------------------------------------

# 20. DFF Constant Optimization --- My Understanding

The important concept I learned is:

``` text
RTL DFF
   ↓
Analyze its behavior
   ↓
Identify constant behavior
   ↓
Remove unnecessary hardware
   ↓
Smaller optimized implementation
```

The RTL may contain a register, but the final synthesized design does
not necessarily need to retain that register if its value can be proven
to be constant and it does not affect the required functionality.

------------------------------------------------------------------------

# 21. State Optimization

I learned about **state optimization** in sequential designs.

One important case is the optimization of unused states.

For example:

``` text
S0 → S1 → S2
```

If another state can never be reached:

``` text
S3
```

then the synthesis tool may be able to optimize the state
representation.

The important concept is:

``` text
Unused / unreachable state
          ↓
     Optimization
          ↓
Reduced state logic
```

------------------------------------------------------------------------

# 22. Retiming

I learned that **retiming** is an advanced sequential optimization
technique.

The basic idea is to change the positions of registers while preserving
the required sequential behavior.

For example:

``` text
Before:

Logic → FF → Logic → FF → Logic
```

The registers can potentially be repositioned to improve the
distribution of combinational delay.

Conceptually:

``` text
Original register placement
           ↓
       Retiming
           ↓
Improved register placement
           ↓
Better timing balance
```

Retiming is particularly relevant when timing performance is important.

------------------------------------------------------------------------

# 23. Sequential Logic Cloning

I also learned about **sequential logic cloning**.

The basic idea is that a sequential element can be replicated when doing
so can provide a physical or timing advantage.

Conceptually:

``` text
Before:

             ┌─────┐
             │ DFF │
             └──┬──┘
                │
          ┌─────┴─────┐
          │           │
        Logic       Logic
```

After cloning:

``` text
       ┌─────┐       ┌─────┐
       │ DFF │       │ DFF │
       └──┬──┘       └──┬──┘
          │             │
        Logic         Logic
```

I learned that this concept is related to physical-aware or
floor-plan-aware synthesis.

------------------------------------------------------------------------

# 24. Counter Optimization

I then studied sequential optimization using a 3-bit counter.

The main design file is:

``` text
counter_opt.v
```

I can inspect it using:

``` bash
cd ~/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files
cat counter_opt.v
```

------------------------------------------------------------------------

# 25. Synthesizing the Counter

Start Yosys:

``` bash
yosys
```

Then:

``` yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog counter_opt.v
synth -top counter_opt
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

I then inspect the synthesized circuit to understand how many sequential
elements are actually required.

------------------------------------------------------------------------

# 26. Why the Counter Result Is Important

I learned an important lesson from the counter example.

Even though the RTL is described as a 3-bit counter, the synthesized
result can contain fewer active flip-flops when the synthesis tool can
prove that some states or bits are unnecessary.

The reasoning is:

``` text
RTL counter
     ↓
Analyze reachable behavior
     ↓
Identify unnecessary state/bits
     ↓
Optimize
     ↓
Reduced hardware
```

This demonstrates that:

> The number of registers written in RTL does not always equal the
> number of registers present in the optimized hardware.

------------------------------------------------------------------------

# 27. Modifying the Counter RTL

I also learned that changing the RTL condition can change what the
synthesis tool is able to optimize.

For example, explicitly expressing a condition involving:

``` verilog
count == 3'b000
```

can change the set of states that must be represented.

The important relationship is:

``` text
RTL condition
     ↓
Reachable behavior
     ↓
Synthesis analysis
     ↓
Optimization
     ↓
Final hardware
```

Therefore, the way I write the RTL can directly affect the optimization
opportunities available to the synthesis tool.

------------------------------------------------------------------------

# 28. Area and Power

One of the major reasons for optimization is reducing hardware
resources.

If unnecessary logic is removed:

``` text
Fewer gates
    ↓
Less area
```

and potentially:

``` text
Less switching
    ↓
Lower dynamic power
```

Therefore, logic optimization is closely related to:

``` text
Area
Power
Timing
```

The Day 03 exercises mainly focus on understanding the optimization
process and its effect on the synthesized structure.

------------------------------------------------------------------------

# 29. My Day 03 Yosys Flow

The main flow I learned is:

``` text
                    RTL
                     │
                     ▼
              read_verilog
                     │
                     ▼
                  synth
                     │
                     ▼
              Optimization
                     │
          ┌──────────┴──────────┐
          │                     │
   Combinational          Sequential
   Optimization            Optimization
          │                     │
          │              ┌──────┼──────┐
          │              │      │      │
          │            State  Retiming Clone
          │             Opt.
          │
          └──────────┬──────────┘
                     ▼
            opt_clean / ABC
                     │
                     ▼
             Sky130 Mapping
                     │
                     ▼
                  show
```

------------------------------------------------------------------------

# 30. Important Day 03 Commands

### Start Yosys

``` bash
yosys
```

### Read Liberty library

``` yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

### Read RTL

``` yosys
read_verilog <design>.v
```

### Synthesize

``` yosys
synth -top <top_module>
```

### Clean unused logic

``` yosys
opt_clean -purge
```

### Map flip-flops

``` yosys
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

### Technology mapping

``` yosys
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

### Flatten hierarchy

``` yosys
flatten
```

### View design

``` yosys
show
```

------------------------------------------------------------------------

# 31. Important File Reference

The main Day 03 files I worked with are:

``` text
Combinational Optimization
────────────────────────────
opt_check.v
opt_check2.v
opt_check3.v
opt_check4.v

Sequential Constant Optimization
─────────────────────────────────
dff_const1.v
dff_const2.v
dff_const3.v
dff_const4.v
dff_const5.v

Counter Optimization
─────────────────────
counter_opt.v
```

Associated simulation files include:

``` text
tb_opt_check.v
tb_opt_check2.v
tb_opt_check3.v
tb_dff_const1.v
tb_dff_const2.v
tb_dff_const3.v
tb_dff_const4.v
tb_dff_const5.v
```

The exact files available in the workshop directory should always be
checked before running a command:

``` bash
ls
```

------------------------------------------------------------------------

# 32. Day 03 Practical Checklist

-   [ ] Understand the purpose of logic optimization.
-   [ ] Understand constant propagation.
-   [ ] Understand direct optimization.
-   [ ] Revise Boolean identities.
-   [ ] Understand K-map minimization.
-   [ ] Understand the Quine-McCluskey method.
-   [ ] Inspect `opt_check.v`.
-   [ ] Run the Yosys optimization flow.
-   [ ] Use `opt_clean -purge`.
-   [ ] Run ABC technology mapping.
-   [ ] Inspect the result with `show`.
-   [ ] Repeat the experiment with `opt_check2.v`.
-   [ ] Repeat the experiment with `opt_check3.v`.
-   [ ] Repeat the experiment with `opt_check4.v`.
-   [ ] Understand hierarchy and flattening.
-   [ ] Study sequential constant propagation.
-   [ ] Inspect `dff_const1.v` to `dff_const5.v`.
-   [ ] Simulate the DFF constant examples.
-   [ ] View the generated VCD files in GTKWave.
-   [ ] Synthesize the DFF constant examples.
-   [ ] Use `dfflibmap`.
-   [ ] Understand state optimization.
-   [ ] Understand retiming.
-   [ ] Understand sequential logic cloning.
-   [ ] Inspect `counter_opt.v`.
-   [ ] Synthesize the counter.
-   [ ] Observe the optimized number of storage elements.
-   [ ] Modify the counter condition.
-   [ ] Compare the synthesis results.
-   [ ] Understand how RTL coding affects synthesis optimization.

------------------------------------------------------------------------

# 33. What I Learned from Day 03

The most important things I learned are:

### 1. Synthesis is more than translation

The synthesis tool does not simply convert every RTL statement into an
equivalent gate.

It analyzes the logic and removes structures that are not required.

### 2. Constants can remove hardware

If a signal is known to be constant, the logic depending on it can often
be simplified.

### 3. Boolean simplification can reduce hardware

Boolean identities, K-maps, and other minimization methods provide ways
to obtain simpler logic.

### 4. Sequential logic can also be optimized

Optimization is not limited to combinational gates. Registers, states,
and sequential structures can also be optimized.

### 5. Unused states can be removed

If a state is not required or cannot be reached, the state logic may be
reduced.

### 6. Retiming changes register placement

Registers can be repositioned to improve timing while maintaining the
required sequential behavior.

### 7. Cloning can help physical implementation

Sequential logic can be replicated when physical or timing requirements
make that useful.

### 8. RTL coding matters

The synthesis tool is powerful, but I still need to understand how my
RTL is interpreted.

A small change in the RTL can change the optimization result.

------------------------------------------------------------------------

# 34. Final Day 03 Understanding

My overall understanding of Day 03 is:

``` text
                  RTL
                   │
                   ▼
            Understand Logic
                   │
                   ▼
              Optimize
                   │
        ┌──────────┴──────────┐
        │                     │
        ▼                     ▼
 Combinational           Sequential
 Optimization             Optimization
        │                     │
        │          ┌──────────┼──────────┐
        │          │          │          │
        │        State      Retiming   Cloning
        │        Opt.
        │
        └─────────────┬───────────────┘
                      ▼
              Technology Mapping
                      │
                      ▼
                Sky130 Cells
```

The main lesson I take from Day 03 is:

> **I should not only know how to write RTL; I should understand what
> hardware the RTL can produce and how synthesis can optimize that
> hardware.**

Good RTL coding and knowledge of synthesis optimization are therefore
important for achieving efficient implementations in terms of **area,
power, and timing**.
