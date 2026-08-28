# Master RTL Design & Synthesis --- VSD Workshop

![VSD Certificate](docs/certificate.png)

## Overview

This repository contains my practical learning, RTL implementations,
simulation work, synthesis experiments, gate-level verification, and
supporting files from the **Master RTL Design & Synthesis** program
conducted by **VSD (VLSI System Design)**.

The work follows the hardware-design path from writing synthesizable
Verilog RTL to simulation, synthesis, technology mapping, netlist
generation, and gate-level simulation using the Sky130 standard-cell
library.

The certificate confirms successful completion of the **Master RTL
Design & Synthesis** course and the associated **10-day program**.

------------------------------------------------------------------------

## Certificate

**Participant:** Avinash Goyal\
**Course:** Master RTL Design & Synthesis\
**Organization:** VSD --- VLSI System Design\
**Program:** 10-day program

### Certificate Verification

-   [Verify
    Certificate](https://vsdiat.vlsisystemdesign.com/certificates/d6d3a71b-7bcb-4fa0-bd63-ba2851e7e432)
-   [Download
    Certificate](https://backend.vlsisystemdesign.com/api/public/certificates/d6d3a71b-7bcb-4fa0-bd63-ba2851e7e432/download)

The certificate image included in this repository is stored at:

``` text
docs/certificate.png
```

------------------------------------------------------------------------

# My Learning

## RTL Design to Hardware

My overall learning workflow can be represented as:

``` text
RTL Design
    ↓
Functional Simulation
    ↓
Waveform Verification
    ↓
RTL Synthesis
    ↓
Logic Optimization
    ↓
Technology Mapping
    ↓
Gate-Level Netlist
    ↓
Gate-Level Simulation
    ↓
Waveform Verification
```

This workflow helped me understand that Verilog RTL is a description of
hardware and that RTL coding style directly influences the hardware
inferred by synthesis.

------------------------------------------------------------------------

# Major Topics Covered

## 1. Verilog RTL Design

I worked with synthesizable Verilog for:

-   Combinational logic
-   Sequential logic
-   Multiplexers
-   Demultiplexers
-   Counters
-   Shift registers
-   Latches
-   Flip-flops
-   Arithmetic circuits
-   Pattern-detection FSMs
-   Repeated hardware structures
-   Multi-module designs

Representative files include:

``` text
good_mux.v
bad_mux.v
good_counter.v
bad_counter.v
good_latch.v
bad_latch.v
good_shift_reg.v
bad_shift_reg.v
pattern_detect_fsm.v
pattern_detect_fsm_bad_style.v
```

------------------------------------------------------------------------

## 2. Combinational Logic and Latch Inference

I learned that combinational RTL must define the output for every
relevant input condition.

An incomplete structure such as:

``` verilog
always @(*)
begin
    if (sel)
        y = a;
end
```

does not define `y` when `sel` is false. This can cause an inferred
latch.

A complete structure is:

``` verilog
always @(*)
begin
    if (sel)
        y = a;
    else
        y = b;
end
```

Another useful style is a default assignment:

``` verilog
always @(*)
begin
    y = 1'b0;

    if (sel)
        y = a;
end
```

I learned to check combinational RTL carefully for incomplete
assignments.

------------------------------------------------------------------------

## 3. `if`, `else if`, and Priority Logic

I studied priority-based conditional structures:

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

The first true condition has priority.

This is different from independent `if` statements, where a later
assignment can overwrite an earlier assignment when multiple conditions
are true.

------------------------------------------------------------------------

## 4. `case` Statements

I used `case` statements for selector-based logic:

``` verilog
always @(*)
begin
    case (sel)
        2'b00: y = a;
        2'b01: y = b;
        2'b10: y = c;
        2'b11: y = d;
        default: y = 1'b0;
    endcase
end
```

I learned two important rules:

1.  Avoid incomplete case coverage when combinational behavior is
    intended.
2.  Assign every required output in every execution path.

The relevant exercises include:

``` text
incomp_case.v
comp_case.v
bad_case.v
```

------------------------------------------------------------------------

## 5. Blocking and Non-Blocking Assignments

For combinational procedural logic, I practiced blocking assignments:

``` verilog
always @(*)
begin
    y = a & b;
end
```

For clocked sequential logic, I practiced non-blocking assignments:

``` verilog
always @(posedge clk)
begin
    q <= d;
end
```

I also studied blocking-assignment ordering and simulation caveats
using:

``` text
blocking_caveat.v
tb_blocking_caveat.v
```

------------------------------------------------------------------------

## 6. Sequential Logic

I worked with clocked and resettable designs such as:

``` verilog
always @(posedge clk or posedge reset)
begin
    if (reset)
        q <= 1'b0;
    else
        q <= d;
end
```

Representative files include:

``` text
dff_async_set.v
dff_asyncres.v
dff_asyncres_syncres.v
dff_syncres.v
dff_const1.v
dff_const2.v
dff_const3.v
dff_const4.v
dff_const5.v
```

These exercises helped me understand flip-flop inference, reset
behavior, asynchronous versus synchronous behavior, and constant
optimization.

------------------------------------------------------------------------

## 7. Counters

I implemented and analyzed different counter structures:

``` text
upcntr.v
up_dn_cntr.v
up_dn_cntr_with_load.v
up_dn_cntr_with_load_with_start_stop.v
ripple_counter.v
good_counter.v
bad_counter.v
```

A basic counter structure is:

``` verilog
always @(posedge clk or posedge reset)
begin
    if (reset)
        count <= 0;
    else if (en)
        count <= count + 1'b1;
end
```

------------------------------------------------------------------------

## 8. Multiplexers and Demultiplexers

I implemented MUX and DEMUX logic using different RTL styles.

Relevant files include:

``` text
good_mux.v
bad_mux.v
ternary_operator_mux.v
mux_generate.v
demux_case.v
demux_generate.v
```

For example, a MUX can be expressed as:

``` verilog
assign y = sel ? a : b;
```

or with a `case` statement, or with a loop.

I learned that different RTL descriptions can represent the same
intended hardware, while coding style still matters for correctness and
synthesis behavior.

------------------------------------------------------------------------

## 9. Procedural `for` Loops

I used procedural loops to describe repetitive combinational logic.

Example:

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

This provides a compact RTL description of a wide selection structure.

I also used the same idea for demultiplexer logic.

------------------------------------------------------------------------

## 10. Generate Loops

I learned that `generate for` is used to replicate hardware structures.

Example:

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

This differs from a procedural `for` loop because the generate construct
is used to create repeated hardware instances.

------------------------------------------------------------------------

## 11. Ripple Carry Adder

I implemented an 8-bit Ripple Carry Adder using multiple full adders.

The main files are:

``` text
fa.v
rca.v
tb_rca.v
```

The full adder is:

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

The carry from one full adder becomes the carry input of the next stage.

Conceptually:

``` text
FA0 → FA1 → FA2 → FA3 → ... → FA7
      carry ripples from LSB toward MSB
```

The RCA was also a practical example of using a generate loop for
repeated hardware.

------------------------------------------------------------------------

# Simulation Flow

I used **Icarus Verilog** for simulation and **GTKWave** for waveform
analysis.

## Basic RTL Simulation

``` bash
cd ~/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files

iverilog design.v testbench.v
./a.out
gtkwave waveform.vcd
```

For a multi-module design such as the RCA:

``` bash
iverilog fa.v rca.v tb_rca.v
./a.out
gtkwave tb_rca.vcd
```

The testbench is compiled together with all required RTL modules.

------------------------------------------------------------------------

# Synthesis Flow

I used **Yosys** for RTL synthesis.

Basic flow:

``` bash
yosys
```

``` yosys
read_verilog design.v
synth -top design
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

For a multi-module design:

``` yosys
read_verilog fa.v
read_verilog rca.v
synth -top rca
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

------------------------------------------------------------------------

# Sky130 Technology Library

The repository uses the Sky130 standard-cell library.

The main Liberty file is:

``` text
lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

The workshop also contains:

``` text
DC_WORKSHOP/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
DC_WORKSHOP/lib/sky130_fd_sc_hd__tt_025C_1v80.db
```

The Verilog models used for gate-level simulation are:

``` text
my_lib/verilog_model/primitives.v
my_lib/verilog_model/sky130_fd_sc_hd.v
```

------------------------------------------------------------------------

# Technology Mapping with ABC

After synthesis, I used ABC with the Sky130 Liberty file:

``` yosys
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

This maps the synthesized logic toward cells available in the target
technology library.

------------------------------------------------------------------------

# Netlist Generation

I generated gate-level Verilog netlists using:

``` yosys
write_verilog -noattr design_net.v
```

For example:

``` yosys
write_verilog -noattr rca_net.v
```

The generated netlist can then be used for gate-level simulation.

------------------------------------------------------------------------

# Gate-Level Simulation

The general GLS flow is:

``` text
RTL
 ↓
Yosys synthesis
 ↓
ABC technology mapping
 ↓
Gate-level Verilog netlist
 ↓
Sky130 Verilog cell models
 ↓
Icarus Verilog
 ↓
VCD
 ↓
GTKWave
```

Example:

``` bash
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v rca_net.v tb_rca.v

./a.out
gtkwave tb_rca.vcd
```

This allowed me to compare the synthesized implementation with the
original RTL behavior.

------------------------------------------------------------------------

# Optimization and Structural Analysis

The repository also contains exercises related to synthesis optimization
and structural analysis:

``` text
counter_opt.v
counter_opt2.v
multiple_module_opt.v
multiple_module_opt2.v
opt_check.v
opt_check2.v
opt_check3.v
opt_check4.v
resource_sharing_mult_check.v
check_logic_sharing.v
check_reg_retime.v
check_clock_gating.v
```

I used these exercises to understand that synthesis tools analyze and
optimize RTL rather than simply translating every RTL statement directly
into a separate gate.

------------------------------------------------------------------------

# Hierarchical and Multi-Module Designs

I worked with hierarchical designs using files such as:

``` text
multiple_modules.v
multiple_modules_flat.v
multiple_modules_hier.v
multiple_module_flatten.v
multiple_module_opt.v
multiple_module_opt2.v
```

This helped me understand module hierarchy, flattening, optimization,
and the relationship between RTL modules and the final synthesized
structure.

------------------------------------------------------------------------

# FSM and Pattern Detection

The repository includes:

``` text
pattern_detect_fsm.v
pattern_detect_fsm_bad_style.v
tb_pattern_detect_fsm.v
```

These designs helped me examine how RTL coding style can influence
state-machine implementation and synthesis.

------------------------------------------------------------------------

# Arithmetic and Datapath Examples

Additional RTL examples include:

``` text
fa.v
rca.v
mult_2.v
mult_8.v
```

These exercises provided practical experience with arithmetic datapath
structures and synthesis.

------------------------------------------------------------------------

# Repository Structure

``` text
sky130RTLDesignAndSynthesisWorkshop/
│
├── DC_WORKSHOP/
│   ├── README.md
│   ├── lib/
│   │   ├── sky130_fd_sc_hd__tt_025C_1v80.db
│   │   └── sky130_fd_sc_hd__tt_025C_1v80.lib
│   └── verilog_files/
│       ├── RTL designs
│       ├── testbenches
│       ├── constraints
│       ├── reports
│       └── synthesis outputs
│
├── lib/
│   └── sky130_fd_sc_hd__tt_025C_1v80.lib
│
├── my_lib/
│   └── verilog_model/
│       ├── primitives.v
│       └── sky130_fd_sc_hd.v
│
├── verilog_files/
│   ├── RTL designs
│   ├── testbenches
│   ├── synthesized netlists
│   └── simulation files
│
├── yosys_run.sh
│
└── README.md
```

------------------------------------------------------------------------

# Important Working Paths

Main RTL directory:

``` bash
~/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files
```

Technology library:

``` text
~/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

Sky130 Verilog models:

``` text
~/VLSI/sky130RTLDesignAndSynthesisWorkshop/my_lib/verilog_model/primitives.v
~/VLSI/sky130RTLDesignAndSynthesisWorkshop/my_lib/verilog_model/sky130_fd_sc_hd.v
```

------------------------------------------------------------------------

# Verification Checklist

Before finalizing a design, I check:

-   [ ] The RTL describes the intended hardware.
-   [ ] Combinational outputs are assigned on all required paths.
-   [ ] No unintended latch is inferred.
-   [ ] `if / else if / else` priority is intentional.
-   [ ] `case` coverage is complete where required.
-   [ ] `default` behavior is defined when appropriate.
-   [ ] Every output is assigned in every required branch.
-   [ ] Blocking and non-blocking assignments are used appropriately.
-   [ ] The testbench covers important input conditions.
-   [ ] RTL simulation produces the expected waveform.
-   [ ] Synthesis produces the expected hardware structure.
-   [ ] The generated netlist can be simulated.
-   [ ] Gate-level simulation agrees with the intended functionality.

------------------------------------------------------------------------

# Tools Used

  Tool / Technology   Purpose
  ------------------- -------------------------------------------
  Verilog             RTL and gate-level hardware description
  Icarus Verilog      Simulation
  GTKWave             Waveform analysis
  Yosys               RTL synthesis
  ABC                 Logic optimization and technology mapping
  Sky130              Standard-cell technology library
  Linux / WSL         Development environment
  Git / GitHub        Version control and documentation

------------------------------------------------------------------------

# Overall Learning

The central learning from this work is the relationship between RTL code
and physical hardware.

``` text
Verilog RTL
    ↓
Logic interpretation
    ↓
Synthesis
    ↓
Optimization
    ↓
Standard-cell mapping
    ↓
Gate-level netlist
    ↓
Verification
```

I learned to approach RTL with hardware awareness rather than treating
it purely as a programming language.

In particular, I learned that:

1.  RTL coding style directly affects inferred hardware.
2.  Incomplete combinational assignments can create latches.
3.  `if-else` structures can represent priority logic.
4.  `case` statements are useful for selector-based logic.
5.  Procedural loops can describe repetitive combinational logic.
6.  Generate loops can replicate hardware structures.
7.  Multiple full adders can be connected to build an RCA.
8.  Simulation is required to verify RTL behavior.
9.  Synthesis reveals the hardware inferred from RTL.
10. Gate-level simulation provides another verification stage after
    synthesis.
11. Technology libraries are required to map logic to real standard
    cells.
12. Waveform analysis is essential for understanding and debugging
    hardware behavior.

------------------------------------------------------------------------

# Project Purpose

This repository serves as my formal record of practical work in:

**RTL Design → Verification → Synthesis → Technology Mapping →
Gate-Level Verification**

It contains both the Verilog implementations and the supporting material
required to reproduce and study the design flow.

------------------------------------------------------------------------

# Author

**Avinash Goyal**

**Focus:** RTL Design • Verilog • Digital Design • Synthesis •
Gate-Level Simulation • Sky130

------------------------------------------------------------------------

## Certificate Verification

[Verify
Certificate](https://vsdiat.vlsisystemdesign.com/certificates/d6d3a71b-7bcb-4fa0-bd63-ba2851e7e432)

[Download
Certificate](https://backend.vlsisystemdesign.com/api/public/certificates/d6d3a71b-7bcb-4fa0-bd63-ba2851e7e432/download)
