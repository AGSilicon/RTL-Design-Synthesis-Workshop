# VSD RTL Design and Synthesis Workshop --- Day 04

## My Day 04 Learning

### 1. Introduction

In Day 04, I learned about **Gate-Level Simulation (GLS)** and the
problems that can occur when the behavior of RTL simulation and
synthesized hardware do not match.

The main topics I studied are:

-   Gate-Level Simulation
-   RTL-to-netlist verification
-   Simulation and synthesis mismatches
-   Missing sensitivity lists
-   Blocking assignments
-   Non-blocking assignments
-   Blocking-assignment caveats
-   Practical GLS using a ternary-operator multiplexer
-   Identifying a sensitivity-list mismatch
-   Practical analysis of a blocking-assignment example

The main flow I learned is:

``` text
RTL Design
    │
    ▼
RTL Simulation
    │
    ▼
Synthesis
    │
    ▼
Gate-Level Netlist
    │
    ▼
Gate-Level Simulation
    │
    ▼
Compare Behavior
```

------------------------------------------------------------------------

# 2. Gate-Level Simulation

I learned that **Gate-Level Simulation (GLS)** means running the
testbench with the synthesized gate-level netlist as the Design Under
Test (DUT).

The basic concept is:

``` text
RTL Testbench
      │
      ▼
Gate-Level Netlist
      │
      ▼
Simulation
```

The netlist should represent the same logical behavior as the original
RTL.

Therefore, the same testbench can be used to check the synthesized
design.

------------------------------------------------------------------------

# 3. Why I Need Gate-Level Simulation

I learned that GLS is useful for checking the design after synthesis.

The two main purposes covered are:

1.  Verifying the logical correctness of the synthesized design.
2.  Checking whether the design meets timing requirements.

For complete timing verification, delay information needs to be
annotated during simulation.

The detailed delay-annotation process was outside the scope of this
session.

The basic idea is:

``` text
RTL
 │
 ▼
Synthesis
 │
 ▼
Gate-Level Netlist
 │
 ▼
GLS
 │
 ├── Logical correctness
 └── Timing verification
```

------------------------------------------------------------------------

# 4. RTL Simulation vs Gate-Level Simulation

I understood the difference between the two stages.

### RTL Simulation

``` text
RTL Verilog
    │
    ▼
Testbench
    │
    ▼
Simulator
```

### Gate-Level Simulation

``` text
Synthesized Netlist
        │
        ▼
Same Testbench
        │
        ▼
Simulator
```

The purpose of GLS is to verify that the synthesized implementation
behaves as expected.

------------------------------------------------------------------------

# 5. Simulation and Synthesis Mismatches

I learned that simulation mismatches can occur when the RTL code is
written in a way that causes the simulator and synthesis tool to
interpret the design differently.

The important issues covered in Day 04 are:

``` text
Simulation / Synthesis Mismatch
│
├── Missing Sensitivity List
├── Blocking vs Non-Blocking Assignment
└── Non-Standard Verilog
```

The main focus of the practical exercises was on:

-   Missing sensitivity lists
-   Blocking assignments

------------------------------------------------------------------------

# 6. Missing Sensitivity List

I learned that a simulator responds to changes in signals included in
the sensitivity list of an `always` block.

For example:

``` verilog
always @(sel)
begin
    ...
end
```

means that the procedural block is triggered when `sel` changes.

If the logic inside the block also depends on another signal, that
signal should also be considered.

------------------------------------------------------------------------

# 7. `always @(*)`

I learned that:

``` verilog
always @(*)
```

is useful for combinational logic because the simulator automatically
considers the signals used by the procedural block.

For example:

``` verilog
always @(*)
begin
    if (sel)
        y = a;
    else
        y = b;
end
```

Here, the logic depends on:

``` text
sel
a
b
```

Using:

``` verilog
always @(*)
```

allows the sensitivity list to be automatically derived from the signals
used by the block.

------------------------------------------------------------------------

# 8. Why a Missing Sensitivity List Can Cause a Problem

Suppose the logic is:

``` verilog
always @(sel)
begin
    if (sel)
        y = a;
    else
        y = b;
end
```

The block explicitly depends only on:

``` text
sel
```

but the output also depends on:

``` text
a
b
```

If `a` changes while `sel` remains unchanged, the simulator may not
execute the block again.

This can result in simulation behavior that does not represent the
intended combinational behavior.

The important lesson I learned is:

``` text
Combinational logic
       ↓
Sensitivity must represent all relevant inputs
       ↓
always @(*)
       ↓
Safer combinational simulation
```

------------------------------------------------------------------------

# 9. Blocking and Non-Blocking Assignments

I learned the difference between:

``` verilog
=
```

and:

``` verilog
<=
```

These are:

``` text
=   → Blocking assignment

<=  → Non-blocking assignment
```

The difference becomes particularly important inside procedural `always`
blocks.

------------------------------------------------------------------------

# 10. Blocking Assignment

A blocking assignment uses:

``` verilog
=
```

For example:

``` verilog
always @(*)
begin
    a = b;
    c = a;
end
```

The statements execute in the order in which they are written.

Conceptually:

``` text
First statement
      ↓
Update
      ↓
Second statement
```

Therefore, the second statement can observe the result produced by the
first statement within the same procedural execution.

------------------------------------------------------------------------

# 11. Non-Blocking Assignment

A non-blocking assignment uses:

``` verilog
<=
```

For example:

``` verilog
always @(posedge clk)
begin
    q1 <= d;
    q2 <= q1;
end
```

I learned that the right-hand sides are evaluated when the block is
entered, and the left-hand-side updates occur as non-blocking updates.

Conceptually:

``` text
Evaluate RHS values
        │
        ├─────────────┐
        │             │
        ▼             ▼
      q1 <= d      q2 <= q1
```

This is why non-blocking assignments are commonly associated with
clocked sequential logic.

------------------------------------------------------------------------

# 12. Blocking vs Non-Blocking

The key difference I learned is:

  -----------------------------------------------------------------------
  Blocking `=`                        Non-blocking `<=`
  ----------------------------------- -----------------------------------
  Statements execute in written order RHS values are evaluated before
                                      updates

  Later statements can see earlier    Updates are scheduled without
  blocking updates                    immediately changing the other RHS
                                      evaluations

  Useful when procedural ordering is  Commonly used for clocked
  intentionally required              sequential logic
  -----------------------------------------------------------------------

The important point is not to choose an assignment operator randomly. I
need to understand the intended hardware and the behavior of the
procedural block.

------------------------------------------------------------------------

# 13. Blocking Assignment Caveats

I studied an example showing that careless use of blocking assignments
can create unexpected behavior.

A simplified sequential structure is:

``` verilog
module example (
    input clk,
    input reset,
    input d,
    output reg q,
    output reg q0
);

always @(posedge clk, posedge reset)
begin
    if (reset)
    begin
        q  = 1'b0;
        q0 = 1'b0;
    end
    else
    begin
        // Sequential assignments
    end
end

endmodule
```

The exact behavior depends on the complete RTL.

The important lesson I learned is:

> When using blocking assignments in sequential logic, I must be very
> clear about the ordering and the intended behavior.

Otherwise, the simulation behavior can become difficult to reason about
and can contribute to mismatches.

------------------------------------------------------------------------

# 14. GLS Practical --- Ternary Operator Multiplexer

I worked with:

``` text
ternary_operator_mux.v
```

The first step was to simulate the RTL using its testbench.

From the working directory:

``` bash
cd ~/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files
```

I can compile the RTL and testbench using:

``` bash
iverilog ternary_operator_mux.v tb_ternary_operator_mux.v
```

Then:

``` bash
./a.out
```

The generated waveform can be opened using:

``` bash
gtkwave <generated_vcd_file>
```

The exact VCD filename should be taken from the testbench.

------------------------------------------------------------------------

# 15. Synthesizing the Ternary Multiplexer

After RTL simulation, I synthesized the design using Yosys.

Start Yosys from:

``` bash
cd ~/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files
yosys
```

Then:

``` yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog ternary_operator_mux.v
synth -top ternary_operator_mux
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog -noattr ternary_operator_mux_net.v
```

The command:

``` yosys
write_verilog -noattr ternary_operator_mux_net.v
```

writes the synthesized gate-level netlist to:

``` text
ternary_operator_mux_net.v
```

------------------------------------------------------------------------

# 16. Why `write_verilog -noattr` Is Used

I learned that:

``` yosys
write_verilog -noattr
```

can be used to write the synthesized design as a Verilog netlist without
additional Yosys attributes.

For example:

``` yosys
write_verilog -noattr ternary_operator_mux_net.v
```

produces a gate-level Verilog file that can then be used for simulation.

The flow is:

``` text
RTL
 ↓
Yosys
 ↓
Synthesis
 ↓
ABC Mapping
 ↓
write_verilog
 ↓
Gate-Level Netlist
```

------------------------------------------------------------------------

# 17. Simulating the Gate-Level Netlist

The synthesized netlist uses Sky130 standard-cell models.

The required model files in my workshop are:

``` text
../my_lib/verilog_model/primitives.v
../my_lib/verilog_model/sky130_fd_sc_hd.v
```

Therefore, the gate-level simulation needs the cell models along with
the synthesized netlist.

A typical command is:

``` bash
iverilog \
../my_lib/verilog_model/primitives.v \
../my_lib/verilog_model/sky130_fd_sc_hd.v \
ternary_operator_mux_net.v \
tb_ternary_operator_mux.v
```

Then:

``` bash
./a.out
```

After simulation, I can inspect the waveform with:

``` bash
gtkwave <generated_vcd_file>
```

The important concept is:

``` text
Testbench
    │
    ├── RTL design        → RTL simulation
    │
    └── Netlist + models  → Gate-level simulation
```

------------------------------------------------------------------------

# 18. Comparing RTL and Netlist Behavior

I learned that the same testbench can be used to check the gate-level
implementation.

The flow is:

``` text
                 ┌── RTL ────────────────┐
                 │                       │
Testbench ───────┤                       ├── Compare
                 │                       │
                 └── Netlist + Cells ───┘
```

If the RTL and synthesized netlist do not behave as expected, I need to
investigate the RTL coding style and synthesis result.

------------------------------------------------------------------------

# 19. Bad MUX Example

After the ternary multiplexer example, I moved to:

``` text
bad_mux.v
```

and its testbench:

``` text
tb_bad_mux.v
```

The RTL simulation flow is:

``` bash
cd ~/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files
iverilog bad_mux.v tb_bad_mux.v
./a.out
gtkwave tb_bad_mux.vcd
```

This allows me to observe the behavior of the RTL before synthesis.

------------------------------------------------------------------------

# 20. Synthesizing `bad_mux.v`

The synthesis flow is:

``` bash
yosys
```

Then:

``` yosys
read_verilog bad_mux.v
synth -top bad_mux
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog -noattr bad_mux_net.v
```

The synthesized netlist is:

``` text
bad_mux_net.v
```

------------------------------------------------------------------------

# 21. Gate-Level Simulation of `bad_mux.v`

I then use the Sky130 Verilog models with the synthesized netlist.

From:

``` bash
cd ~/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files
```

I can compile:

``` bash
iverilog \
../my_lib/verilog_model/primitives.v \
../my_lib/verilog_model/sky130_fd_sc_hd.v \
bad_mux_net.v \
tb_bad_mux.v
```

Then:

``` bash
./a.out
```

and:

``` bash
gtkwave tb_bad_mux.vcd
```

------------------------------------------------------------------------

# 22. Sensitivity-List Mismatch

The `bad_mux.v` experiment helped me understand the sensitivity-list
problem more clearly.

The important comparison is:

``` text
RTL simulation
      │
      ▼
Observe waveform
      │
      ▼
Synthesize RTL
      │
      ▼
Gate-level netlist
      │
      ▼
Run same testbench
      │
      ▼
Compare waveforms
```

The mismatch observed in the experiment is related to the
sensitivity-list issue.

The important lesson I learned is:

> The simulator executes an `always` block according to its sensitivity
> list, while synthesis interprets the RTL as hardware behavior. An
> incomplete sensitivity list can therefore create a simulation result
> that does not correctly represent the intended combinational hardware.

------------------------------------------------------------------------

# 23. Why `always @(*)` Helps

For combinational logic, I should prefer:

``` verilog
always @(*)
begin
    ...
end
```

instead of manually maintaining an incomplete sensitivity list.

For example:

``` verilog
always @(*)
begin
    if (sel)
        y = a;
    else
        y = b;
end
```

This makes the combinational intent clearer and allows the simulator to
respond to changes in all signals used by the block.

------------------------------------------------------------------------

# 24. Blocking Assignment Example

The next practical example is:

``` text
blocking_caveat.v
```

with its testbench:

``` text
tb_blocking_caveat.v
```

I simulate it using:

``` bash
cd ~/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files
iverilog blocking_caveat.v tb_blocking_caveat.v
./a.out
gtkwave tb_blocking_caveat.vcd
```

The purpose is to observe the behavior caused by blocking assignments in
a sequential context.

------------------------------------------------------------------------

# 25. Synthesizing `blocking_caveat.v`

The synthesis flow is:

``` bash
yosys
```

Then:

``` yosys
read_verilog blocking_caveat.v
synth -top blocking_caveat
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog -noattr blocking_caveat_net.v
```

The resulting netlist is:

``` text
blocking_caveat_net.v
```

------------------------------------------------------------------------

# 26. Gate-Level Simulation of `blocking_caveat.v`

The synthesized design can then be simulated with the Sky130 cell
models:

``` bash
iverilog \
../my_lib/verilog_model/primitives.v \
../my_lib/verilog_model/sky130_fd_sc_hd.v \
blocking_caveat_net.v \
tb_blocking_caveat.v
```

Then:

``` bash
./a.out
```

and:

``` bash
gtkwave blocking_caveat.vcd
```

The waveform can then be compared with the RTL simulation.

------------------------------------------------------------------------

# 27. My Understanding of Blocking Assignment Caveats

The main lesson I learned is that blocking assignments execute in
procedural order.

For example:

``` verilog
always @(posedge clk)
begin
    q1 = d;
    q2 = q1;
end
```

The second assignment sees the updated value of `q1` within the same
procedural execution.

That is different from:

``` verilog
always @(posedge clk)
begin
    q1 <= d;
    q2 <= q1;
end
```

where both right-hand sides are evaluated before the non-blocking
updates occur.

Therefore, I need to be careful when using blocking assignments in
sequential logic.

The Day 04 practical demonstrates that unclear use of blocking
assignments can lead to unexpected or mismatching output behavior.

------------------------------------------------------------------------

# 28. Important File Reference

The main Day 04 files I worked with are:

``` text
Gate-Level Simulation
─────────────────────
ternary_operator_mux.v
tb_ternary_operator_mux.v

Sensitivity-List Example
────────────────────────
bad_mux.v
tb_bad_mux.v

Blocking Assignment Example
───────────────────────────
blocking_caveat.v
tb_blocking_caveat.v
```

Generated gate-level netlists include:

``` text
ternary_operator_mux_net.v
bad_mux_net.v
blocking_caveat_net.v
```

The Sky130 simulation models are:

``` text
../my_lib/verilog_model/primitives.v
../my_lib/verilog_model/sky130_fd_sc_hd.v
```

------------------------------------------------------------------------

# 29. Correct Day 04 Working Directory

All Day 04 practical work can be performed from:

``` bash
cd ~/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files
```

The main Sky130 Liberty file is:

``` text
../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

The Sky130 Verilog cell models are:

``` text
../my_lib/verilog_model/primitives.v
../my_lib/verilog_model/sky130_fd_sc_hd.v
```

------------------------------------------------------------------------

# 30. Complete GLS Flow

The complete flow I learned is:

``` text
                   RTL
                    │
                    ▼
             RTL Simulation
                    │
                    ▼
                 Yosys
                    │
                    ▼
                Synthesis
                    │
                    ▼
             ABC Technology
                Mapping
                    │
                    ▼
            Gate-Level Netlist
                    │
                    ▼
        Sky130 Verilog Cell Models
                    │
                    ▼
          Gate-Level Simulation
                    │
                    ▼
              GTKWave
                    │
                    ▼
           Compare Behavior
```

------------------------------------------------------------------------

# 31. Practical GLS Command Flow

## Step 1 --- RTL Simulation

``` bash
iverilog ternary_operator_mux.v tb_ternary_operator_mux.v
./a.out
gtkwave <rtl_vcd_file>
```

## Step 2 --- Synthesis

``` bash
yosys
```

``` yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog ternary_operator_mux.v
synth -top ternary_operator_mux
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog -noattr ternary_operator_mux_net.v
```

## Step 3 --- Gate-Level Simulation

``` bash
iverilog \
../my_lib/verilog_model/primitives.v \
../my_lib/verilog_model/sky130_fd_sc_hd.v \
ternary_operator_mux_net.v \
tb_ternary_operator_mux.v
```

``` bash
./a.out
```

## Step 4 --- View waveform

``` bash
gtkwave <gate_level_vcd_file>
```

## Step 5 --- Compare

I compare:

``` text
RTL waveform
     vs
Gate-level waveform
```

------------------------------------------------------------------------

# 32. Important Yosys Commands

### Read Liberty

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

### Technology mapping

``` yosys
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

### Write gate-level Verilog

``` yosys
write_verilog -noattr <netlist>.v
```

------------------------------------------------------------------------

# 33. Important Simulation Commands

### Compile RTL

``` bash
iverilog <design>.v <testbench>.v
```

### Run simulation

``` bash
./a.out
```

### Compile gate-level netlist

``` bash
iverilog \
../my_lib/verilog_model/primitives.v \
../my_lib/verilog_model/sky130_fd_sc_hd.v \
<netlist>.v \
<testbench>.v
```

### View waveform

``` bash
gtkwave <vcd_file>
```

------------------------------------------------------------------------

# 34. Day 04 Practical Checklist

-   [ ] Understand the purpose of Gate-Level Simulation.
-   [ ] Understand why the synthesized netlist is simulated.
-   [ ] Understand the relationship between RTL and gate-level
    simulation.
-   [ ] Understand why timing verification may require delay annotation.
-   [ ] Study missing sensitivity lists.
-   [ ] Understand `always @(sel)`.
-   [ ] Understand `always @(*)`.
-   [ ] Study blocking assignment `=`.
-   [ ] Study non-blocking assignment `<=`.
-   [ ] Understand the difference between blocking and non-blocking
    execution.
-   [ ] Study blocking-assignment caveats.
-   [ ] Simulate `ternary_operator_mux.v`.
-   [ ] Synthesize `ternary_operator_mux.v`.
-   [ ] Generate `ternary_operator_mux_net.v`.
-   [ ] Simulate the synthesized netlist.
-   [ ] Use the Sky130 Verilog cell models.
-   [ ] Compare RTL and gate-level waveforms.
-   [ ] Simulate `bad_mux.v`.
-   [ ] Synthesize `bad_mux.v`.
-   [ ] Generate `bad_mux_net.v`.
-   [ ] Run gate-level simulation.
-   [ ] Identify the sensitivity-list mismatch.
-   [ ] Simulate `blocking_caveat.v`.
-   [ ] Synthesize `blocking_caveat.v`.
-   [ ] Generate `blocking_caveat_net.v`.
-   [ ] Run gate-level simulation.
-   [ ] Observe the effect of blocking assignments.
-   [ ] Compare RTL and synthesized behavior.

------------------------------------------------------------------------

# 35. What I Learned from Day 04

### 1. GLS verifies the synthesized implementation

I learned that I should not stop after RTL simulation. The synthesized
netlist can also be simulated to verify the post-synthesis
implementation.

### 2. The same testbench can be used

I learned that the testbench can be used with the synthesized netlist as
the DUT.

### 3. RTL coding style matters

I learned that certain Verilog coding styles can create differences
between simulation behavior and synthesized hardware.

### 4. Sensitivity lists are important

I learned that an incomplete sensitivity list can prevent the simulator
from responding to a signal change even though the intended
combinational hardware depends on that signal.

### 5. `always @(*)` is useful for combinational logic

I learned that `always @(*)` automatically includes the signals used by
the procedural block in the sensitivity calculation.

### 6. Blocking and non-blocking assignments behave differently

I learned that:

``` text
=   → blocking
<=  → non-blocking
```

and that their execution behavior is different.

### 7. Blocking assignments require care in sequential logic

I learned that the ordering of blocking assignments can affect the
result within the same procedural block.

### 8. Gate-level simulation helps expose problems

I learned that comparing RTL simulation and gate-level simulation can
help identify issues related to the RTL coding style and synthesized
implementation.

------------------------------------------------------------------------

# 36. Final Day 04 Understanding

My overall understanding of Day 04 is:

``` text
                RTL
                 │
                 ▼
          RTL Simulation
                 │
                 ▼
              Synthesis
                 │
                 ▼
          Gate-Level Netlist
                 │
                 ▼
        Gate-Level Simulation
                 │
                 ▼
        Compare with RTL
                 │
          ┌──────┴──────┐
          │             │
        Match        Mismatch
          │             │
          ▼             ▼
      Confidence    Investigate RTL
                    / coding issue
```

The main lesson I take from Day 04 is:

> **I need to understand not only whether my RTL produces the expected
> simulation output, but also how that RTL is interpreted during
> synthesis and how the resulting gate-level implementation behaves.**

In particular, I need to be careful with:

``` text
Sensitivity Lists
Blocking Assignments
Non-Blocking Assignments
RTL Coding Style
```

because these can affect the relationship between simulation and
synthesized hardware.
