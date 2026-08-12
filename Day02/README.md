# VSD RTL Design and Synthesis Workshop --- Day 02

## 1. Overview

Day 02 of the VSD RTL Design and Synthesis Workshop focuses on the
transition from RTL descriptions to technology-mapped digital hardware
using the **Sky130 standard-cell library** and **Yosys**.

The session covers:

-   Sky130 standard-cell libraries and PVT information
-   Standard-cell selection and implementation differences
-   Hierarchical and flat synthesis
-   Design hierarchy and flattening
-   Flip-flop implementation
-   Asynchronous and synchronous reset/set structures
-   Sequential-cell mapping using `dfflibmap`
-   RTL implementation of multiplication by powers of two
-   Practical synthesis and inspection using Yosys

The objective is not only to execute synthesis commands, but also to
understand how the synthesis tool interprets RTL and maps it to cells
available in the target technology.

------------------------------------------------------------------------

# 2. Working Environment

The workshop is located at:

``` bash
~/VLSI/sky130RTLDesignAndSynthesisWorkshop
```

For the Day 02 practical exercises, work primarily from:

``` bash
cd ~/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files
```

The relevant directory structure is:

``` text
~/VLSI/sky130RTLDesignAndSynthesisWorkshop/
├── lib/
│   └── sky130_fd_sc_hd__tt_025C_1v80.lib
├── my_lib/
│   └── verilog_model/
│       ├── primitives.v
│       └── sky130_fd_sc_hd.v
└── verilog_files/
    ├── multiple_modules.v
    ├── multiple_module_flat.v
    ├── multiple_module_hier.v
    ├── dff_asyncres.v
    ├── dff_async_set.v
    ├── dff_syncres.v
    ├── dff_asyncres_syncres.v
    ├── mult_2.v
    ├── mult_8.v
    └── ...
```

### 2.1 Sky130 Liberty File

When the current directory is:

``` text
~/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files
```

the correct relative path to the Liberty file is:

``` text
../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

The Sky130 Verilog models used for gate-level simulation are located at:

``` text
../my_lib/verilog_model/primitives.v
../my_lib/verilog_model/sky130_fd_sc_hd.v
```

> **Note:** The commands in this document use the actual filenames
> present in the workshop directory rather than abbreviated paths.

------------------------------------------------------------------------

# 3. Sky130 Standard-Cell Library

## 3.1 Standard Cells

A standard-cell library provides pre-designed digital building blocks
that can be used by a synthesis tool to implement an RTL design.

Typical standard-cell categories include:

-   Combinational logic cells
-   Buffers and inverters
-   Multiplexers
-   Sequential cells
-   Flip-flops

At RTL, a designer describes functionality. During synthesis, the tool
determines an appropriate implementation using cells available in the
target library.

The basic relationship is:

``` text
RTL Description
       │
       ▼
   Synthesis
       │
       ▼
Technology Mapping
       │
       ▼
Sky130 Standard Cells
```

------------------------------------------------------------------------

# 4. PVT and Sky130 Cell Naming

The Day 02 material introduces Sky130 naming and the concept of:

``` text
Process
Voltage
Temperature
```

commonly referred to as **PVT**.

The Liberty file used in this workshop is:

``` text
sky130_fd_sc_hd__tt_025C_1v80.lib
```

The library information is used by Yosys during technology mapping.

PVT conditions are important because the electrical characteristics of
cells depend on the manufacturing process, operating voltage, and
temperature.

------------------------------------------------------------------------

# 5. Standard-Cell Variants

The session demonstrates that a logical function may have multiple
standard-cell implementations.

For example, different AND-cell variants may provide different
characteristics.

The important design principle is:

``` text
Same logical function
        │
        ├── Implementation A
        │
        └── Implementation B
```

Different implementations can have different physical characteristics
such as:

-   Area
-   Drive capability
-   Timing characteristics
-   Power characteristics

Therefore, the logical description alone does not completely determine
the physical implementation.

------------------------------------------------------------------------

# 6. Hierarchical RTL Design

Day 02 introduces synthesis using a design containing multiple modules.

The primary example is:

``` text
multiple_modules.v
```

The design contains two submodules.

## 6.1 `sub_module1`

``` verilog
module sub_module1 (
    input a,
    input b,
    output y
);
    assign y = a & b;
endmodule
```

Therefore:

``` text
net1 = a & b
```

## 6.2 `sub_module2`

``` verilog
module sub_module2 (
    input a,
    input b,
    output y
);
    assign y = a | b;
endmodule
```

Therefore:

``` text
y = a | b
```

## 6.3 Top-Level Module

``` verilog
module multiple_modules (
    input a,
    input b,
    input c,
    output y
);

    wire net1;

    sub_module1 u1 (
        .a(a),
        .b(b),
        .y(net1)
    );

    sub_module2 u2 (
        .a(net1),
        .b(c),
        .y(y)
    );

endmodule
```

The resulting Boolean relationship is:

``` text
net1 = a & b

y = net1 | c

Therefore:

y = (a & b) | c
```

------------------------------------------------------------------------

# 7. Design Hierarchy

The structure of `multiple_modules.v` can be represented as:

``` text
multiple_modules
│
├── u1 : sub_module1
│       └── a AND b → net1
│
└── u2 : sub_module2
        └── net1 OR c → y
```

This is a hierarchical RTL representation because the top-level module
contains instances of lower-level modules.

------------------------------------------------------------------------

# 8. Yosys Synthesis Flow

The basic Day 02 synthesis flow is:

``` text
Read Liberty Library
        │
        ▼
Read RTL
        │
        ▼
Synthesis
        │
        ▼
Technology Mapping
        │
        ▼
Inspect Result
```

The corresponding Yosys commands are:

``` yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog multiple_modules.v
synth -top multiple_modules
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

------------------------------------------------------------------------

# 9. Step-by-Step Hierarchical Synthesis

## Step 1 --- Enter the Verilog directory

``` bash
cd ~/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files
```

## Step 2 --- Start Yosys

``` bash
yosys
```

## Step 3 --- Read the Sky130 Liberty library

``` yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

This provides Yosys with information about the available standard cells.

## Step 4 --- Read the RTL

``` yosys
read_verilog multiple_modules.v
```

Yosys reads the modules and converts the Verilog description into its
internal representation.

## Step 5 --- Synthesize the top-level module

``` yosys
synth -top multiple_modules
```

The option:

``` text
-top multiple_modules
```

identifies the top-level design.

## Step 6 --- Perform technology mapping

``` yosys
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

This maps the synthesized logic using the target Sky130 library.

## Step 7 --- View the result

``` yosys
show
```

This opens a graphical representation of the synthesized design.

------------------------------------------------------------------------

# 10. Why the Synthesized Circuit May Look Different

An RTL description such as:

``` verilog
assign y = (a & b) | c;
```

describes the required Boolean functionality.

It does not require the final technology-mapped implementation to appear
exactly as:

``` text
AND → OR
```

The synthesis process can transform the logic into an equivalent
implementation.

Conceptually:

``` text
RTL
 │
 ▼
Logic optimization
 │
 ▼
Technology mapping
 │
 ▼
Sky130 standard cells
```

Therefore, the synthesized circuit may look different from the original
RTL while implementing the same logical function.

------------------------------------------------------------------------

# 11. Hierarchical and Flat Synthesis

The Day 02 session introduces two forms of design representation:

### Hierarchical representation

``` text
Top Module
│
├── Submodule 1
│
└── Submodule 2
```

### Flat representation

``` text
Top Module
│
└── Combined lower-level logic
```

Hierarchy preserves module boundaries, whereas flattening removes those
boundaries from the synthesized representation.

------------------------------------------------------------------------

# 12. Flattening the Design

The Yosys command used to flatten the hierarchy is:

``` yosys
flatten
```

A complete example is:

``` yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog multiple_modules.v
synth -top multiple_modules
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
flatten
show
```

The purpose of the exercise is to compare the hierarchical and flattened
representations.

------------------------------------------------------------------------

# 13. Why Study Hierarchy and Flattening?

The exercise demonstrates an important synthesis concept:

``` text
RTL hierarchy
      ↓
Synthesis representation
      ↓
Optimization
      ↓
Technology mapping
```

Understanding hierarchy is important because real RTL designs are
normally composed of many modules and submodules.

Flattening provides the synthesis tool with a representation in which
lower-level logic can be incorporated into the parent design.

The Day 02 material specifically uses the `multiple_modules.v` example
to study this difference.

------------------------------------------------------------------------

# 14. Sequential Logic --- Flip-Flops

After combinational synthesis, Day 02 introduces sequential logic.

A flip-flop is a storage element controlled by a clock.

A simplified D flip-flop can be represented as:

``` text
             ┌─────────┐
D ──────────►│   DFF   │────► Q
             │         │
CLK ────────►│         │
             └─────────┘
```

The flip-flop allows a digital system to store state and operate
synchronously with the clock.

The session also discusses the role of flip-flops in controlling
glitches and separating sequential stages.

------------------------------------------------------------------------

# 15. Reset and Set

The Day 02 session covers:

-   Asynchronous reset
-   Asynchronous set
-   Synchronous reset
-   Combined asynchronous and synchronous reset behavior

These mechanisms provide controlled initialization or modification of
the flip-flop state.

------------------------------------------------------------------------

# 16. Asynchronous Reset

An asynchronous reset can affect the flip-flop without waiting for a
clock edge.

Conceptually:

``` text
                 ┌─────────┐
D ──────────────►│         │
CLK ────────────►│   DFF   │────► Q
                 │         │
ASYNC_RESET ────►│         │
                 └─────────┘
```

The Day 02 notes describe asynchronous reset as not waiting for the
clock.

------------------------------------------------------------------------

# 17. Asynchronous Set

Asynchronous set operates similarly, but forces the flip-flop toward its
set state.

For an active-high set:

``` text
async_set = 1
     │
     ▼
Q is forced to 1
```

The operation is independent of the normal clock edge.

------------------------------------------------------------------------

# 18. Synchronous Reset

A synchronous reset is evaluated with respect to the clock.

Conceptually:

``` text
                 ┌─────────┐
D ──────────────►│         │
CLK ────────────►│   DFF   │────► Q
SYNC_RESET ─────►│         │
                 └─────────┘
```

The Day 02 material describes synchronous reset as waiting for the
clock.

Therefore:

``` text
Asynchronous Reset
    → does not wait for the clock

Synchronous Reset
    → acts with the clock
```

------------------------------------------------------------------------

# 19. Day 02 DFF Examples

The workshop contains the following sequential designs:

``` text
dff_asyncres.v
dff_async_set.v
dff_syncres.v
dff_asyncres_syncres.v
```

The corresponding testbench files are also available in:

``` text
~/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files
```

These examples are used to understand reset/set behavior and how the
corresponding sequential structures are mapped to the Sky130 library.

------------------------------------------------------------------------

# 20. DFF Simulation

Move to the working directory:

``` bash
cd ~/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files
```

For example, compile an asynchronous-reset DFF together with its
testbench:

``` bash
iverilog dff_asyncres.v tb_dff_asyncres.v
```

Run the generated executable:

``` bash
./a.out
```

If the testbench generates a VCD file, open the generated VCD with:

``` bash
gtkwave <vcd_file>
```

Use the VCD filename specified by the actual testbench.

------------------------------------------------------------------------

# 21. DFF Synthesis and Mapping

A typical Day 02 Yosys flow for a DFF is:

``` yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_asyncres.v
synth -top dff_asyncres
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

The important additional step for sequential logic is:

``` yosys
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

------------------------------------------------------------------------

# 22. Purpose of `dfflibmap`

At RTL level, a flip-flop is represented generically.

The target Sky130 library contains actual sequential standard cells.

The mapping process can therefore be represented as:

``` text
Generic RTL DFF
       │
       ▼
   dfflibmap
       │
       ▼
Sky130 DFF Cell
```

The Liberty library provides the information required for this mapping.

------------------------------------------------------------------------

# 23. Sequential Synthesis Flow

For sequential designs, the Day 02 flow is:

``` text
                RTL DFF
                   │
                   ▼
                Yosys
                   │
                synth
                   │
                   ▼
          Generic Sequential Logic
                   │
                   ▼
               dfflibmap
                   │
                   ▼
             Sky130 DFF Cell
                   │
                   ▼
                  ABC
                   │
                   ▼
          Technology-mapped design
```

------------------------------------------------------------------------

# 24. Multiplication by Powers of Two

Day 02 also examines the implementation of multiplication by:

``` text
2
4
8
```

The key observation is that powers of two correspond to binary left
shifts.

``` text
×2 → shift left by 1 bit
×4 → shift left by 2 bits
×8 → shift left by 3 bits
```

This provides a simple hardware implementation for these specific
constant multipliers.

------------------------------------------------------------------------

# 25. Multiplication by 2

Consider:

``` text
A = abc
```

Multiplication by 2 results in:

``` text
abc × 2 = abc0
```

Conceptually, one zero is inserted at the least-significant-bit side.

For a vector:

``` text
A[2:0]
```

the shifted representation can be expressed as:

``` verilog
{A[2:0], 1'b0}
```

------------------------------------------------------------------------

# 26. Multiplication by 4

Since:

``` text
4 = 2²
```

multiplication by 4 corresponds to a two-bit left shift:

``` text
abc × 4 = abc00
```

Conceptually:

``` verilog
{A[2:0], 2'b00}
```

------------------------------------------------------------------------

# 27. Multiplication by 8

Since:

``` text
8 = 2³
```

multiplication by 8 corresponds to a three-bit left shift:

``` text
abc × 8 = abc000
```

Conceptually:

``` verilog
{A[2:0], 3'b000}
```

------------------------------------------------------------------------

# 28. Why This Optimization Is Important

A general multiplication operation can require significantly more
hardware than a simple shift when the multiplier is a power of two.

Therefore, recognizing mathematical properties in RTL is important.

The Day 02 examples demonstrate:

``` text
×2
 ↓
left shift by 1


×4
 ↓
left shift by 2


×8
 ↓
left shift by 3
```

This is an example of how the RTL description can expose a simpler
implementation.

------------------------------------------------------------------------

# 29. Multiplier Examples in the Workshop

The relevant files include:

``` text
mult_2.v
mult_8.v
```

Inspect the files from:

``` bash
cd ~/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files
```

For example:

``` bash
cat mult_2.v
```

and:

``` bash
cat mult_8.v
```

------------------------------------------------------------------------

# 30. Synthesizing `mult_2.v`

Start Yosys:

``` bash
cd ~/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files
yosys
```

Then execute:

``` yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog mult_2.v
synth -top mult_2
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

Inspect the resulting structure and relate it to the shift operation.

------------------------------------------------------------------------

# 31. Synthesizing `mult_8.v`

Similarly:

``` yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog mult_8.v
synth -top mult_8
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

The synthesized result should be interpreted in relation to
multiplication by 8 and its equivalent left-shift operation.

------------------------------------------------------------------------

# 32. Importance of Signal Width

The Day 02 session introduces a case where:

``` text
Input  = 3 bits
Output = 6 bits
```

Signal width must be considered carefully when performing arithmetic
operations.

For a 3-bit unsigned value:

``` text
000 → 0
001 → 1
...
111 → 7
```

When the value is multiplied, additional bits may be required to
represent the result.

Therefore, always verify:

``` text
Input width
      +
Operation
      +
Constant
      ↓
Required output width
```

Incorrect assumptions about width can lead to an implementation that
does not preserve the intended information.

------------------------------------------------------------------------

# 33. Practical Day 02 Workflow

## Combinational Design

``` text
1. Enter verilog_files
2. Start Yosys
3. Read Liberty library
4. Read RTL
5. Synthesize the top module
6. Run ABC technology mapping
7. View the design
8. Compare hierarchy and flattened representation
```

Commands:

``` bash
cd ~/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files
yosys
```

``` yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog multiple_modules.v
synth -top multiple_modules
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

For flattening:

``` yosys
flatten
show
```

------------------------------------------------------------------------

# 34. Sequential Design Workflow

``` text
1. Write/read DFF RTL
2. Simulate with Icarus Verilog
3. Observe the waveform in GTKWave
4. Read the Sky130 Liberty library
5. Read the DFF RTL
6. Run synthesis
7. Map DFFs using dfflibmap
8. Run ABC
9. Inspect the synthesized design
```

Example:

``` yosys
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog dff_asyncres.v
synth -top dff_asyncres
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

------------------------------------------------------------------------

# 35. Command Reference

  -----------------------------------------------------------------------
  Command                             Purpose
  ----------------------------------- -----------------------------------
  `yosys`                             Starts Yosys

  `read_liberty -lib <file>`          Reads the target standard-cell
                                      library

  `read_verilog <file>`               Reads Verilog RTL

  `synth -top <module>`               Synthesizes the selected top-level
                                      module

  `dfflibmap -liberty <file>`         Maps generic flip-flops to library
                                      cells

  `abc -liberty <file>`               Performs technology mapping using
                                      the library

  `flatten`                           Removes module hierarchy

  `show`                              Displays the synthesized design

  `iverilog <files>`                  Compiles Verilog for simulation

  `./a.out`                           Runs the compiled simulation

  `gtkwave <file>`                    Opens a VCD waveform
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# 36. Important Path Reference

### Workshop root

``` bash
~/VLSI/sky130RTLDesignAndSynthesisWorkshop
```

### Day 02 working directory

``` bash
~/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files
```

### Sky130 Liberty file

``` text
~/VLSI/sky130RTLDesignAndSynthesisWorkshop/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

### Relative Liberty path from `verilog_files`

``` text
../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

### Sky130 Verilog models

``` text
~/VLSI/sky130RTLDesignAndSynthesisWorkshop/my_lib/verilog_model/primitives.v
~/VLSI/sky130RTLDesignAndSynthesisWorkshop/my_lib/verilog_model/sky130_fd_sc_hd.v
```

------------------------------------------------------------------------

# 37. Day 02 Practical Checklist

-   [ ] Navigate to the Day 02 working directory.
-   [ ] Verify the Sky130 Liberty file path.
-   [ ] Inspect `multiple_modules.v`.
-   [ ] Understand the submodule hierarchy.
-   [ ] Read the Liberty library in Yosys.
-   [ ] Read the RTL using `read_verilog`.
-   [ ] Synthesize using `synth -top`.
-   [ ] Perform technology mapping using `abc`.
-   [ ] Inspect the synthesized design using `show`.
-   [ ] Flatten the design using `flatten`.
-   [ ] Compare hierarchical and flattened representations.
-   [ ] Review the DFF examples.
-   [ ] Simulate a DFF using Icarus Verilog.
-   [ ] Observe the waveform using GTKWave.
-   [ ] Synthesize a DFF.
-   [ ] Use `dfflibmap` for sequential-cell mapping.
-   [ ] Review asynchronous reset behavior.
-   [ ] Review asynchronous set behavior.
-   [ ] Review synchronous reset behavior.
-   [ ] Inspect `mult_2.v`.
-   [ ] Inspect `mult_8.v`.
-   [ ] Synthesize the multiplier examples.
-   [ ] Relate multiplication by powers of two to left shifting.
-   [ ] Verify input and output widths.

------------------------------------------------------------------------

# 38. Key Learning Outcomes

After completing Day 02, you should be able to explain the following.

### Standard cells

-   What a standard-cell library provides.
-   Why the target technology library is required during synthesis.
-   The significance of PVT conditions.
-   Why different cells can implement related logic functions with
    different physical characteristics.

### Synthesis

-   The purpose of `read_liberty`.
-   The purpose of `read_verilog`.
-   The role of `synth -top`.
-   The purpose of `abc -liberty`.
-   Why synthesized logic may differ structurally from the original RTL.

### Hierarchy

-   What hierarchical RTL means.
-   What flat synthesis means.
-   What `flatten` does.
-   Why hierarchical and flattened implementations are compared.

### Sequential logic

-   What a flip-flop does.
-   The difference between asynchronous and synchronous reset.
-   The purpose of asynchronous set.
-   The purpose of `dfflibmap`.

### RTL arithmetic

-   Why multiplication by 2 corresponds to a one-bit left shift.
-   Why multiplication by 4 corresponds to a two-bit left shift.
-   Why multiplication by 8 corresponds to a three-bit left shift.
-   Why signal width must be considered during arithmetic operations.

------------------------------------------------------------------------

# 39. Day 02 Conceptual Summary

The complete Day 02 flow can be summarized as:

``` text
                    RTL
                     │
                     ▼
              ┌─────────────┐
              │    Yosys    │
              └──────┬──────┘
                     │
                     ▼
                 Synthesis
                     │
                     ▼
              Logic Optimization
                     │
                     ▼
             Technology Mapping
                     │
                     ▼
            Sky130 Standard Cells
                     │
                     ▼
              Mapped Netlist
```

For combinational logic:

``` text
RTL
 ↓
synth
 ↓
abc
 ↓
Sky130 combinational cells
```

For sequential logic:

``` text
RTL
 ↓
synth
 ↓
dfflibmap
 ↓
abc
 ↓
Sky130 sequential + combinational cells
```

------------------------------------------------------------------------

# 40. Final Takeaway

Day 02 establishes the connection between **RTL design, synthesis, and
the target semiconductor technology**.

The essential relationship is:

``` text
Verilog RTL
     │
     ▼
Synthesis
     │
     ▼
Optimization
     │
     ▼
Technology Mapping
     │
     ▼
Sky130 Standard Cells
```

The practical exercises demonstrate that:

1.  RTL hierarchy can be preserved or flattened.
2.  The synthesis result does not necessarily resemble the original RTL
    structure.
3.  Flip-flops require appropriate sequential-cell mapping.
4.  Reset and set behavior affects the type of sequential hardware
    inferred.
5.  Mathematical properties such as multiplication by powers of two can
    result in simple hardware structures such as shifts.
6.  Signal width must be considered carefully when implementing
    arithmetic logic.

The primary objective of Day 02 is therefore to develop an understanding
of **how Yosys transforms an RTL description into a technology-specific
hardware implementation using the Sky130 standard-cell library**.
