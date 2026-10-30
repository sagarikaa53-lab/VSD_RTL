# VSD RTL Design and Synthesis – Day 1

This repository documents the Day 1 activities of the VSD RTL Design and Synthesis Workshop, covering RTL design concepts, Verilog simulation, waveform analysis, and logic synthesis using Yosys and the SKY130 standard-cell library.

## Index

1. [Workshop Repository Setup](#1-workshop-repository-setup)
2. [Introduction to RTL Design, Testbench and Simulation](#2-introduction-to-rtl-design-testbench-and-simulation)
3. [Repository Directory Structure](#3-repository-directory-structure)
4. [Simulating the good Mux Design](#4-simulating-the-good-mux-design)
5. [Viewing Verilog Files](#5-viewing-verilog-files)
6. [Introduction to Yosys and Synthesis](#6-introduction-to-yosys-and-synthesis)
7. [Introduction to Logic Synthesis](#7-introduction-to-logic-synthesis)
8. [Yosys Lab: good Mux](#8-yosys-lab-good-mux)

## 1. Workshop Repository Setup

Create a working directory and clone the workshop repository.

```bash
mkdir VLSI
cd VLSI
git clone https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop.git
cd sky130RTLDesignAndSynthesisWorkshop
```

## 2. Introduction to RTL Design, Testbench and Simulation

### RTL Design

RTL stands for **Register Transfer Level**. RTL design describes the behavior and data flow of a digital circuit using a hardware description language such as Verilog. It specifies how data is processed and transferred between registers and logic elements.

The Verilog design is the description of the hardware we want to implement. For example, a multiplexer selects one of several inputs based on its select signal.

### Testbench

A testbench is a Verilog module used to test a design. It provides input signals to the design, observes its outputs, and helps verify whether the circuit behaves as expected. A testbench is used for simulation and is generally not part of the final hardware implementation.

### Simulator

A simulator executes the Verilog design and testbench over simulated time. It allows us to check the circuit's behavior before proceeding to synthesis or hardware implementation.

**Icarus Verilog (`iverilog`)** is an open-source Verilog simulator. It compiles the design and testbench into an executable that can be run to perform the simulation.

### VCD and GTKWave

A VCD (Value Change Dump) file records changes in signals during simulation when waveform dumping is enabled in the testbench.

**GTKWave** is a waveform viewer that opens VCD files and displays signal transitions over time. This helps examine inputs, outputs, and timing relationships.

The basic simulation flow is:

1. Write the Verilog design.
2. Create a testbench.
3. Compile and run the simulation using Icarus Verilog.
4. Generate a VCD file.
5. View the signal waveforms in GTKWave.

## 3. Repository Directory Structure

The workshop repository contains files organized by their purpose.

- **`lib/`** – SKY130 standard-cell library files.
- **Verilog models** – Verilog models representing standard cells.
- **Verilog source files** – Design modules and testbench files used for simulation and synthesis.

To locate the relevant files:

```bash
find . -name "good_mux.v"
find . -name "tb_good_mux.v"
find . -name "*.lib"
```

## 4. Simulating the "good mux" Design

Navigate to the directory containing `good_mux.v` and `tb_good_mux.v`.

### Step 1: Compile the design and testbench

```bash
iverilog good_mux.v tb_good_mux.v
```

This compiles the Verilog files and generates the default executable, `a.out`, if compilation succeeds.

### Step 2: Run the simulation

```bash
./a.out
```

If waveform dumping is configured in the testbench, the simulation generates a VCD file.

### Step 3: Open the waveform

```bash
gtkwave tb_good_mux.vcd
```
<img width="1917" height="1025" alt="Screenshot 2026-10-30 151013" src="https://github.com/user-attachments/assets/ec461a12-183b-46a9-8bb0-ab2f3354c7c4" />

GTKWave displays the recorded signal waveforms for analysis.

## 5. Viewing Verilog Files

Open the design and testbench side by side using GVim:

```bash
gvim tb_good_mux.v -o good_mux.v
```
<img width="1917" height="1007" alt="Screenshot 2026-10-30 151905" src="https://github.com/user-attachments/assets/6adb5755-1f6e-4390-b798-f7bf7dfc4a52" />

This helps compare the design module with its testbench.

## 6. Introduction to Yosys and Synthesis

### What is Synthesis?

Synthesis is the process of converting a hardware description, such as RTL written in Verilog, into a representation of hardware built from logic elements. The synthesis tool analyzes the design and maps its logic to cells available in a target technology library.

### What is a Synthesizer?

A synthesizer is a tool that performs logic synthesis. **Yosys** is an open-source synthesis tool that reads Verilog designs, processes their logic, and produces a synthesized netlist.

### Simulator vs. Synthesizer

Icarus Verilog and Yosys serve different purposes:

- **Icarus Verilog:** Simulates the design to check its functional behavior.
- **Yosys:** Synthesizes the design into a hardware representation, such as a gate-level netlist.

A **netlist** describes a circuit in terms of cells and their connections. A standard-cell library, such as the SKY130 library, contains cell definitions and characteristics used during technology mapping.

## 7. Introduction to Logic Synthesis

The Day 1 workshop introduces the fundamentals of logic synthesis in two sessions:

These sessions provide background for the practical Yosys synthesis lab.

## 8. Yosys Lab: "good mux" design

Start Yosys in the terminal from the directory containing the design and the relevant library path:

```bash
yosys
```

Run the following commands in the Yosys prompt.

### Step 1: Read the standard-cell library

```tcl
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025c_1v80.lib
```

This loads the SKY130 high-density standard-cell library for use during synthesis.

### Step 2: Read the Verilog design

```tcl
read_verilog good_mux.v
```

This loads the good Mux design into Yosys.

### Step 3: Run synthesis

```tcl
synth -top good_mux
```
```tcl
abc -liberty ../lib/sky130_fd_sc_hd__tt_025c_1v80.lib
```
This tells which model to simulate, optimizes and creates the netlist of the file.

```tcl
show
```
This displays the graphical representation of synthesized design.
<img width="1915" height="965" alt="Screenshot 2026-10-30 171617" src="https://github.com/user-attachments/assets/42156d7a-cbe8-4d73-9c9f-6120bb450967" />


### Step 4:  Exporting and Inspecting the Netlist

```tcl
write_verilog -noattr good_mux_netlist.v
```
This command generates a Verilog netlist representing the synthesized design.
```tcl
 !gvim good_mux_netlist.v 
```
Following command opens netlist in the GVim tool

<img width="1896" height="956" alt="image" src="https://github.com/user-attachments/assets/48e51494-28b3-4611-8f46-68ce31285133" />

To verify the outputs:
1. Simulate the original RTL and synthesized netlist using the same testbench.
2. Compare their output waveforms to verify functional consistency.

## Summary of Learning Outcomes

- Learned the fundamentals of RTL design, testbenches, and simulation.
- Practiced compiling and running Verilog simulations using Icarus Verilog.
- Explored signal waveforms with GTKWave.
- Understood the role of Yosys in logic synthesis.
- Learned how standard-cell libraries support technology mapping.
- Synthesized a 2:1 multiplexer and generated its Verilog netlist.
- Gained an introduction to process corners and timing characteristics.
- Understood the importance of checking synthesized designs against the original RTL.   

## Tools Covered

- **Icarus Verilog (`iverilog`)** – Verilog simulation
- **GTKWave** – Waveform visualization
- **GVim** – Viewing Verilog source files
- **Yosys** – Logic synthesis
- **SKY130 standard-cell library** – Standard-cell definitions and characteristics used in synthesis
