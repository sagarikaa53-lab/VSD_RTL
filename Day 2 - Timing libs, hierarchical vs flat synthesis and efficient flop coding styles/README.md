# Day 2: Standard-Cell Libraries, Synthesis and RTL Optimization

## Overview

This section explores how standard-cell timing libraries are used in digital design, how hierarchical and flat synthesis differ, and how RTL coding styles affect hardware implementation.

The experiments use Verilog, Yosys and the SKY130 standard-cell library to examine synthesized designs, flip-flop coding styles and simple RTL optimizations.

## Contents

- [1. Exploring Timing Libraries](#1-exploring-timing-libraries)
- [2. Hierarchical and Flat Synthesis](#2-hierarchical-and-flat-synthesis)
- [3. Synthesizing Individual Modules](#3-synthesizing-individual-modules)
- [4. Flip-Flop Coding Styles](#4-flip-flop-coding-styles)
- [5. RTL Optimization](#5-rtl-optimization)
- [6. Key Learnings](#6-key-learnings)
- [7. Tools Used](#7-tools-used)

---

## 1. Exploring Timing Libraries

### 1.1 What is a `.lib` file?

A Liberty (`.lib`) file describes the characteristics of standard cells available in a technology library. Synthesis tools use this information to map RTL logic into suitable standard cells.

The library contains information such as:

- Cell names and logical functions
- Area of standard cells
- Timing arcs and delay information
- Input capacitance and output characteristics
- Power-related information
- Operating conditions, including process, voltage and temperature (PVT)

### 1.2 Understanding PVT corners

PVT represents three operating conditions:

| Parameter | Meaning |
|---|---|
| P — Process | Manufacturing variation in the semiconductor process |
| V — Voltage | Operating supply voltage |
| T — Temperature | Operating temperature |

Different PVT conditions affect cell delay, power and performance. Therefore, the selected library corner matters when analyzing a design.

### 1.3 Inspecting a Liberty file

First, locate the SKY130 standard-cell library in your installed PDK. The exact directory depends on your installation.

From the terminal, navigate to your design or work directory:

```bash
cd ~/your_working_directory
```

Locate the timing library:

```bash
find "$PDK_ROOT" -name 'sky130_fd_sc_hd__tt_025C_1v80.lib'
```

Set the path to the library returned by the command. For example, if it is located in a directory named `lib` one level above your current directory:

```text
../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

Open it for inspection:

```bash
less ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

Press `q` to exit `less`.

You can also search for a cell name or cell declaration:

```bash
grep -n 'cell (' ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

### 1.4 Comparing cell variants

Inspect different drive-strength variants of a standard cell, such as AND gates. Compare their area, timing and power-related data.

The exact values depend on the cell and operating conditions described in the library.

**Screenshot placeholder:** Add a screenshot showing the Liberty file and the cell parameters being examined.

`![Liberty file inspection](images/liberty-inspection.png)`

---

## 2. Hierarchical and Flat Synthesis

Synthesis converts Verilog RTL into a gate-level representation using cells from a target technology library.

### 2.1 Hierarchical synthesis

In hierarchical synthesis, the design retains its module structure. A top-level module can instantiate smaller submodules, allowing the design to be understood and synthesized in a modular manner.

For this experiment, the input design is `multiple_modules.v`, and the top-level module is `multiple_modules`.

### 2.2 Open the RTL file

Navigate to the directory containing your Verilog design:

```bash
cd /path/to/your/verilog/files
```

Open the file in a text editor:

```bash
gvim multiple_modules.v
```

Alternatively, use:

```bash
nano multiple_modules.v
```

Make sure the file contains the required module definitions and that the top-level module is named `multiple_modules`.

### 2.3 Run Yosys

Start Yosys from the terminal:

```bash
yosys
```

The following commands are entered **at the Yosys prompt**, not directly in the Linux shell.

Read the standard-cell library:

```tcl
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

Read the Verilog design:

```tcl
read_verilog multiple_modules.v
```

Synthesize the selected top-level module:

```tcl
synth -top multiple_modules
```

Map the logic using the target library:

```tcl
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

Display the synthesized design:

```tcl
show multiple_modules
```

Write the resulting Verilog netlist to a file:

```tcl
write_verilog multiple_modules_hier.v
```

Exit Yosys:

```tcl
exit
```

**Note:** The library path must point to an existing Liberty file. If your PDK installation uses a different location or filename, replace the example path accordingly.

### 2.4 Inspect the synthesized netlist

From the Linux terminal, open the generated netlist:

```bash
gvim multiple_modules_hier.v
```

Or print it in the terminal:

```bash
less multiple_modules_hier.v
```

The hierarchical netlist can be examined to understand how the modules and their connections are represented after synthesis.



<img width="1672" height="1010" alt="Screenshot 2026-11-02 134610" src="https://github.com/user-attachments/assets/6f2e0f63-35bc-4852-9acc-558a898da61e" />



---

## 3. Synthesizing Individual Modules

A submodule can also be selected as the synthesis top to examine its implementation separately.

For example, to synthesize `sub_module1`, start Yosys and run:

```tcl
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog multiple_modules.v
synth -top sub_module1
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show sub_module1
write_verilog sub_module1_netlist.v
```

Exit Yosys:

```tcl
exit
```

Inspect the output:

```bash
gvim sub_module1_netlist.v
```

This allows the logic inside an individual module to be studied independently of the complete top-level design.


<img width="1911" height="995" alt="Screenshot 2026-11-02 141027" src="https://github.com/user-attachments/assets/278f2d0b-5d3e-4d05-bc32-3b986ea5c2ed" />


---

## 4. Converting the Design to Flat Synthesis

### 4.1 What is flat synthesis?

Flat synthesis removes the hierarchy between instantiated modules so that their logic is brought into the parent design. This allows synthesis to work on a flattened representation of the complete design.

In Yosys, the `flatten` command performs this operation.

### 4.2 Run the flat synthesis flow

Start Yosys from the terminal:

```bash
yosys
```

Enter the following commands at the Yosys prompt:

```tcl
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog multiple_modules.v
synth -top multiple_modules
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show multiple_modules
```

Flatten the design:

```tcl
flatten
```

Display the flattened design:

```tcl
show
```

Write the flattened netlist:

```tcl
write_verilog multiple_modules_flat.v
```

Exit Yosys:

```tcl
exit
```

Open the generated netlist from the terminal:

```bash
gvim multiple_modules_flat.v
```

Or inspect it using:

```bash
less multiple_modules_flat.v
```

<img width="1916" height="992" alt="Screenshot 2026-11-02 134946" src="https://github.com/user-attachments/assets/dbb2c0f6-7c40-412d-9e1b-38bbd764773b" />




### 4.3 Hierarchical versus flat synthesis

| Aspect | Hierarchical synthesis | Flat synthesis |
|---|---|---|
| Module structure | Preserved in the synthesized representation | Module hierarchy is flattened |
| Design view | Organized around module boundaries | Logic is represented in a flattened design |
| Main Yosys command | `synth -top multiple_modules` | `flatten` after synthesis |
| Output example | `multiple_modules_hier.v` | `multiple_modules_flat.v` |

The commands above demonstrate the basic flows. The resulting netlists and optimization outcomes depend on the RTL, synthesis sequence, library and tool version.

---

## 5. Flip-Flop Coding Styles

Flip-flops store state and are fundamental elements of sequential digital circuits. Their behavior depends on the clock and on the reset or set conditions specified in the RTL.

This section examines asynchronous reset, asynchronous set and synchronous reset coding styles.

### 5.1 Asynchronous reset

An asynchronous-reset flip-flop responds to the reset signal independently of the active clock edge.

Example Verilog:

```verilog
module dff_async_reset (
    input  wire clk,
    input  wire reset,
    input  wire d,
    output reg  q
);

always @(posedge clk or posedge reset) begin
    if (reset)
        q <= 1'b0;
    else
        q <= d;
end

endmodule
```

The reset in this example is active-high. When `reset` is asserted, `q` is cleared to zero without waiting for a clock edge.

### 5.2 Asynchronous set

An asynchronous-set flip-flop sets its output independently of the active clock edge when the set signal is asserted.

Example Verilog:

```verilog
module dff_async_set (
    input  wire clk,
    input  wire set,
    input  wire d,
    output reg  q
);

always @(posedge clk or posedge set) begin
    if (set)
        q <= 1'b1;
    else
        q <= d;
end

endmodule
```

The set in this example is active-high. When `set` is asserted, `q` becomes one.

### 5.3 Synchronous reset

A synchronous reset is evaluated only at the active clock edge.

Example Verilog:

```verilog
module dff_sync_reset (
    input  wire clk,
    input  wire reset,
    input  wire d,
    output reg  q
);

always @(posedge clk) begin
    if (reset)
        q <= 1'b0;
    else
        q <= d;
end

endmodule
```

Here, the output is reset only when the clock has a rising edge while `reset` is asserted.

### 5.4 Synthesizing a flip-flop design

Save the desired module in a Verilog file, such as `dff_async_reset.v`. Start Yosys:

```bash
yosys
```

For an example asynchronous-reset design, run:

```tcl
read_verilog dff_async_reset.v
hierarchy -top dff_async_reset
proc
opt
show
```

To write a generic synthesized representation:

```tcl
synth -top dff_async_reset
write_verilog dff_async_reset_netlist.v
```

Exit Yosys and inspect the output:

```tcl
exit
```

```bash
gvim dff_async_reset_netlist.v
```

Repeat the flow with the appropriate filename and top-module name for the asynchronous-set and synchronous-reset designs.

**Important:** Generic synthesis and technology mapping are different stages. To map a design to SKY130 standard cells, use the appropriate target Liberty library and a technology-mapping flow compatible with the selected cells. The exact mapped flip-flop depends on the available library cells and their supported control signals.



<img width="1917" height="982" alt="image" src="https://github.com/user-attachments/assets/6b1e1979-5d76-4cd6-996a-cc8935b21840" />


<img width="1916" height="972" alt="image" src="https://github.com/user-attachments/assets/023e0730-fb49-4ae8-9774-1e14b868745d" />


<img width="1917" height="945" alt="image" src="https://github.com/user-attachments/assets/c7b43798-8a27-4efe-b88e-32b755cbb982" />


---

## 6. RTL Optimization

RTL coding style can influence the logic inferred by synthesis. Equivalent expressions may be simplified or mapped differently by the synthesis tool.

### 6.1 Multiplication by two

Consider the following two expressions for an unsigned input:

```verilog
assign y = a * 2;
```

and

```verilog
assign y = {a, 1'b0};
```

Multiplication by two is equivalent to shifting an unsigned binary value left by one position. The concatenation appends a zero as the least significant bit.

For a three-bit input, use a four-bit output:

```verilog
module multiply_by_two (
    input  wire [2:0] a,
    output wire [3:0] y
);

assign y = {a, 1'b0};

endmodule
```

To compare the arithmetic and concatenation forms, create separate modules or change the assignment and synthesize each version. Inspect the synthesized logic and compare the results.

### 6.2 Multiplication by nine

For an unsigned input a:

\[
9a = 8a + a
\]

This can be represented using shifts and addition:

```verilog
assign y = (a << 3) + a;
```

For a three-bit unsigned input, multiplication by nine can also be expressed using concatenation:

```verilog
assign y = {a, a};
```

The concatenation produces the six-bit value 8a+a, so a six-bit output is appropriate for a three-bit input.

Example module:

```verilog
module multiply_by_nine (
    input  wire [2:0] a,
    output wire [5:0] y
);

assign y = {a, a};

endmodule
```

Compare the synthesized implementations of the multiplication, shift-and-add and concatenation versions. Use the generated netlists and Yosys schematic view to observe the logic inferred by the tool.



### 6.3 Comparing synthesis results

When comparing implementations, consider:

- The logic inferred by synthesis
- The number and type of cells in a technology-mapped netlist
- The area reported by the synthesis flow
- Timing estimates, when available
- Whether both versions have the same input and output widths and behavior

Do not assume that a shorter RTL expression always produces a smaller or faster circuit. The result depends on the synthesis tool, optimization settings and target library.

---

## 7. Key Learnings

- Liberty files describe standard-cell characteristics used during technology mapping and timing analysis.
- PVT conditions influence the electrical and timing behavior of standard cells.
- Hierarchical synthesis retains module organization, while the Yosys `flatten` command removes module hierarchy from the synthesized design.
- Individual submodules can be synthesized separately by selecting them as the top module.
- Asynchronous and synchronous reset coding styles describe different sequential behavior and can lead to different hardware implementations.
- Equivalent RTL expressions can be optimized into different or similar gate-level implementations depending on the synthesis flow.

## Tools Used

- **Verilog:** RTL design description
- **Yosys:** RTL synthesis and netlist generation
- **SKY130 PDK:** Standard-cell technology library
- **Liberty (`.lib`):** Cell timing and characterization data
- **GVim / text editor:** Inspecting and editing Verilog files and synthesized netlists
- **Graphviz:** Used by Yosys schematic visualization when available

## Repository Structure

```text
Day2/
├── README.md
├── rtl/
│   ├── multiple_modules.v
│   ├── dff_async_reset.v
│   ├── dff_async_set.v
│   ├── dff_sync_reset.v
│   ├── multiply_by_two.v
│   └── multiply_by_nine.v
└── images/
    ├── liberty-inspection.png
    ├── hierarchical-synthesis.png
    ├── hierarchical-netlist.png
    ├── submodule-synthesis.png
    ├── flat-synthesis.png
    ├── flattened-netlist.png
    ├── async-reset.png
    ├── async-set.png
    ├── sync-reset.png
    ├── multiply-by-two.png
    └── multiply-by-nine.png
```

