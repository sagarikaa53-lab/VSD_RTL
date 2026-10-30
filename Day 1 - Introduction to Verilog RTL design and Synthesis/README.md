# VSD RTL Design and Synthesis – Day 1

This repository documents the Day 1 activities of the VSD RTL Design and Synthesis Workshop, covering Icarus Verilog simulation, GTKWave waveform viewing, and an introduction to Yosys synthesis using the SKY130 standard-cell library.

## Index

1. [Workshop Repository Setup](#1-workshop-repository-setup)
2. [Introduction to Icarus Verilog](#2-introduction-to-icarus-verilog)
3. [Repository Directory Structure](#3-repository-directory-structure)
4. [Simulating the Good Mux Design](#4-simulating-the-good-mux-design)
5. [Viewing Verilog Files](#5-viewing-verilog-files)
6. [Introduction to Yosys](#6-introduction-to-yosys)
7. [Introduction to Logic Synthesis](#7-introduction-to-logic-synthesis)
8. [Yosys Lab: Good Mux](#8-yosys-lab-good-mux)

## 1. Workshop Repository Setup

Create a working directory and clone the workshop repository.

```bash
mkdir VLSI
cd VLSI
git clone https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop.git
cd sky130RTLDesignAndSynthesisWorkshop
```

## 2. Introduction to Icarus Verilog

Icarus Verilog (`iverilog`) is used to simulate Verilog designs and verify their behavior with a testbench.

The simulation flow covered in this session includes:

- Verilog design files
- Testbench files
- Icarus Verilog compilation and simulation
- VCD waveform generation
- GTKWave waveform visualization

## 3. Repository Directory Structure

The workshop repository contains files organized by their purpose.

- **`lib/`** – SKY130 standard-cell libraries.
- **Verilog models** – Verilog models of standard cells.
- **Verilog source files** – Design files and testbench files used for simulation and synthesis.

To locate the relevant files:

```bash
find . -name "good_mux.v"
find . -name "tb_good_mux.v"
find . -name "*.lib"
```

## 4. Simulating the Good Mux Design

Navigate to the directory containing `good_mux.v` and `tb_good_mux.v`.

### Step 1: Compile the design and testbench

```bash
iverilog good_mux.v tb_good_mux.v
```

This compiles the Verilog files and generates the default output executable, `a.out`.

### Step 2: Run the simulation

```bash
./a.out
```

The simulation generates a VCD waveform file when the testbench is configured to dump waveform data.

### Step 3: Open the waveform

```bash
gtkwave tb_good_mux.vcd
```
<img width="1917" height="1025" alt="Screenshot 2026-10-30 151013" src="https://github.com/user-attachments/assets/0ea938bf-eb57-4abe-a5e8-7a8af4438be3" />

GTKWave displays the recorded signal waveforms for analysis.

## 5. Viewing Verilog Files

Open the design and testbench side by side using GVim:

```bash
gvim tb_good_mux.v -o good_mux.v
```
<img width="1917" height="1007" alt="Screenshot 2026-10-30 151905" src="https://github.com/user-attachments/assets/7833935b-0a64-4b67-9e37-36676f552ef0" />


This helps compare the design module with its testbench.

## 6. Introduction to Yosys

Yosys is a digital logic synthesis tool. It processes Verilog design descriptions and converts them into a synthesized hardware representation.

Icarus Verilog and Yosys serve different purposes:

- **Icarus Verilog:** Simulates a design to check its behavior.
- **Yosys:** Synthesizes a design into a gate-level representation.

A netlist represents the synthesized design using logic cells. The standard-cell library provides information about the available cells.

## 7. Introduction to Logic Synthesis

The Day 1 sessions introduce:

- Logic synthesis – Part 1
- Logic synthesis – Part 2

These sessions lead into the practical Yosys synthesis lab.

## 8. Yosys Lab: Good Mux

Start Yosys in the terminal from the directory containing the design and the relevant library path:

```bash
yosys
```

### Step 1: Read the standard-cell library

```tcl
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025c_1v80.lib
```

This loads the SKY130 high-density standard-cell library for synthesis.

### Step 2: Read the Verilog design

```tcl
read_verilog good_mux.v
```

This loads the Good Mux design into Yosys.

### Step 3: Run synthesis

```tcl
synth -top good_mux
```


```tcl
abc -liberty ../lib/sky130_fd_sc_hd__tt_025c_1v80.lib
```
---

<img width="1915" height="965" alt="Screenshot 2026-10-30 171617" src="https://github.com/user-attachments/assets/fa8647d3-87c5-4b4e-b4a4-26dfb568b259" />

## Tools Covered

- Icarus Verilog (`iverilog`) – Verilog simulation
- GTKWave – Waveform visualization
- GVim – Verilog source viewing
- Yosys – Logic synthesis
- SKY130 standard-cell library – Cell definitions used in synthesis

