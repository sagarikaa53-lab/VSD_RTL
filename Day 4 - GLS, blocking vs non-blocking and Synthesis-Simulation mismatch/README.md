# Day 4 – Gate-Level Simulation, Blocking vs Non-Blocking Assignments and Synthesis-Simulation Mismatch

## Table of Contents

1. [Introduction](#1-introduction)
2. [Gate-Level Simulation (GLS)](#2-gate-level-simulation-gls)
3. [Lab 1 – Functional GLS Using a Ternary Operator MUX](#3-lab-1--functional-gls-using-a-ternary-operator-mux)
4. [Lab 2 – Synthesis-Simulation Mismatch Due to Missing Sensitivity List](#4-lab-2--synthesis-simulation-mismatch-due-to-missing-sensitivity-list)
5. [Lab 3 – Blocking Assignment and Simulation Mismatch](#5-lab-3--blocking-assignment-and-simulation-mismatch)
6. [Key Observations](#8-key-observations)
7. [Key Learnings](#9-key-learnings)
8. [Tools Used](#10-tools-used)

---

## 1. Introduction

Day 4 focuses on understanding Gate-Level Simulation (GLS), the differences between blocking and non-blocking assignments, and the issues that can arise when RTL simulation does not represent the synthesized hardware correctly.

The practical sessions demonstrate how to verify a synthesized design using Icarus Verilog and how coding styles can influence simulation results and hardware implementation.

### Objectives

- Understand the purpose of Gate-Level Simulation.
- Perform RTL and gate-level simulations using Icarus Verilog.
- Generate a synthesized netlist using Yosys.
- Identify synthesis-simulation mismatches.
- Understand the effect of incomplete sensitivity lists.
- Compare blocking and non-blocking assignments.
- Examine the behavior of sequential circuits through shift-register experiments.

## 2. Gate-Level Simulation (GLS)

### 2.1 What is Gate-Level Simulation?

Gate-Level Simulation is the process of simulating a synthesized netlist instead of the original RTL design.

During synthesis, Verilog RTL is converted into a network of logic gates and standard cells. GLS verifies whether the synthesized implementation preserves the expected functionality of the original design.

The same testbench used for RTL simulation can generally be reused for gate-level simulation because both representations implement the same design interface.

### 2.2 RTL Simulation vs Gate-Level Simulation

| RTL Simulation | Gate-Level Simulation |
|---|---|
| Uses RTL Verilog code | Uses synthesized netlist |
| Describes behavioral logic | Describes interconnected standard cells |
| Checks design functionality | Checks post-synthesis functionality |
| Uses RTL modules | Requires standard-cell simulation models |

### 2.3 GLS Flow Using Icarus Verilog

The basic simulation flow consists of the following stages:

1. Write the RTL design and its testbench.
2. Perform RTL simulation using Icarus Verilog.
3. Synthesize the RTL using Yosys.
4. Generate the gate-level netlist.
5. Include the standard-cell Verilog models.
6. Run gate-level simulation using the same testbench.
7. Compare the generated waveforms.

**GLS Flow:**

```text
       RTL Design
           |
           v
    RTL Simulation
      (Icarus)
           |
           v
    RTL Waveform
           |
           v
     RTL Synthesis
        (Yosys)
           |
           v
    Gate-Level Netlist
           |
           v
  Netlist + Cell Models
       + Testbench
           |
           v
    Gate-Level Simulation
        (Icarus)
           |
           v
     GLS Waveform
           |
           v
    Compare Results
```

*Note: Functional GLS verifies logical behavior. Timing-aware GLS additionally requires appropriate delay annotation, commonly using SDF.*

---

## 3. Lab 1 – Functional GLS Using a Ternary Operator MUX

### Objective

To perform RTL simulation, synthesize a 2:1 multiplexer using Yosys, and verify the generated gate-level netlist using Icarus Verilog.

### Step 1: Understand the Design

A 2:1 multiplexer selects one of two input signals depending on the select line.

The ternary operator is used to describe the selection:

```verilog
assign y = sel ? i1 : i0;
```

- When `sel = 0`, output `y` follows `i0`.
- When `sel = 1`, output `y` follows `i1`.

### Step 2: Perform RTL Simulation

Compile the Verilog design and testbench:

```bash
iverilog ternary_operator_mux.v tb_ternary_operator_mux.v
```

Execute the simulation:

```bash
./a.out
```

Open the generated VCD file:

```bash
gtkwave tb_ternary_operator_mux.vcd
```

### Step 3: Synthesize Using Yosys

Start Yosys:

```bash
yosys
```

Read the SKY130 standard-cell library and RTL design:

```tcl
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog ternary_operator_mux.v
```

Perform synthesis:

```tcl
synth -top ternary_operator_mux
```

Map the design to the standard-cell library:

```tcl
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

Generate the synthesized netlist:

```tcl
write_verilog -noattr ternary_operator_mux_net.v
```

Exit Yosys:

```tcl
exit
```






### Step 4: Perform Gate-Level Simulation

The synthesized netlist must be simulated along with the standard-cell Verilog models and the original testbench.

Example command:

```bash
iverilog -o mux_gls \
../my_lib/verilog_model/primitives.v \
../my_lib/verilog_model/sky130_fd_sc_hd.v \
ternary_operator_mux_net.v \
tb_ternary_operator_mux.v
```

Run the simulation:

```bash
./a.out
```

View the waveform:

```bash
gtkwave tb_ternary_operator_mux.vcd
```

Adjust the library model paths and filenames according to the installed SKY130 PDK.

**Screenshot 3 – Gate-Level Simulation**

<img width="1892" height="940" alt="Screenshot 2026-11-11 153922" src="https://github.com/user-attachments/assets/37dbfc90-d04a-415c-bc79-55e684d35a55" />

### Observation

- The RTL simulation produces the expected MUX functionality.
- Yosys converts the RTL description into a gate-level netlist.
- The generated netlist is simulated using standard-cell models.
- The RTL and GLS waveforms are compared to verify functional consistency.

**Result:** The synthesized MUX implementation preserves the expected functionality of the original RTL design.

---

## 4. Lab 2 – Synthesis-Simulation Mismatch Due to Missing Sensitivity List

### Objective

To understand how an incomplete sensitivity list affects RTL simulation and creates a difference between RTL and synthesized hardware.

### Step 1: Understand the Problem

Consider a 2:1 MUX implemented using a procedural block.

```verilog
module bad_mux (
    input i0,
    input i1,
    input sel,
    output reg y
);

always @(sel)
begin
    if (sel)
        y = i1;
    else
        y = i0;
end

endmodule
```

Here, the sensitivity list contains only `sel`.

The simulator executes the block only when `sel` changes. Changes in `i0` and `i1` alone do not trigger the block.

This can result in an outdated output during RTL simulation.

### Step 2: Perform RTL Simulation

Compile and execute the design with its testbench:

```bash
iverilog -o bad_mux_sim bad_mux.v tb_bad_mux.v
./a.out
```

Open the waveform:

```bash
gtkwave tb_bad_mux.vcd
```



### Step 3: Perform Synthesis

Open Yosys:

```bash
yosys
```

Execute:

```tcl
read_verilog bad_mux.v
synth -top bad_mux
show
```

Generate the synthesized netlist:

```tcl
write_verilog -noattr bad_mux_net.v
```



### Step 4: Perform Gate-Level Simulation

Compile the generated netlist with the standard-cell models and the same testbench.

```bash
iverilog -o bad_mux_gls \
../my_lib/verilog_model/primitives.v \
../my_lib/verilog_model/sky130_fd_sc_hd.v \
bad_mux_net.v tb_bad_mux.v
```

Execute:

```bash
./a.out
```

View the waveform:

```bash
gtkwave tb_bad_mux.vcd
```

<img width="1912" height="966" alt="Screenshot 2026-11-11 155419" src="https://github.com/user-attachments/assets/6fd2e745-0261-4106-86c3-3026e6fd9573" />




### Step 5: Correct the RTL Code

Use `always @(*)` for combinational logic.

```verilog
module good_mux (
    input i0,
    input i1,
    input sel,
    output reg y
);

always @(*)
begin
    if (sel)
        y = i1;
    else
        y = i0;
end

endmodule
```

The wildcard sensitivity list ensures that changes in all signals read by the block trigger reevaluation.





### Observation

- The incomplete sensitivity list causes incorrect RTL simulation behavior.
- The synthesized circuit represents the intended MUX functionality.
- The corrected combinational block responds to changes in all relevant inputs.

**Result:** An incomplete sensitivity list can cause synthesis-simulation mismatch. Using `always @(*)` avoids this problem in combinational procedural blocks.

---

## 5. Lab 3 – Blocking Assignment and Simulation Mismatch

### Objective

To examine how the execution order of blocking assignments affects simulation results and can create synthesis-simulation mismatches.

### Step 1: Understand Blocking Assignments

Blocking assignments use the `=` operator.

The statement is executed immediately, and the next statement is evaluated only after the current assignment completes.

Consider the following RTL:

```verilog
module blocking_caveat (
    input a,
    input b,
    input c,
    output reg d
);

reg x;

always @(*)
begin
    d = x & c;
    x = a | b;
end

endmodule
```

In this example:

- `x` is an intermediate signal.
- `d` is calculated using the existing value of `x`.
- The new value of `x` is assigned only after calculating `d`.

This can cause the RTL simulation to use a previous value of `x`, whereas synthesis interprets the combinational logic relationship.

### Step 2: Perform RTL Simulation

Compile and run:

```bash
iverilog blocking_sim blocking_caveat.v tb_blocking_caveat.v
./a.out
```

View the waveform:

```bash
gtkwave tb_blocking_caveat.vcd
```



### Step 3: Perform Synthesis

Run Yosys:

```bash
yosys
```

Execute:

```tcl
read_verilog blocking_caveat.v
synth -top blocking_caveat
show
write_verilog -noattr blocking_caveat_net.v
```



### Step 4: Perform Gate-Level Simulation

Compile the netlist with the required standard-cell models:

```bash
iverilog -o blocking_gls \
../my_lib/verilog_model/primitives.v \
../my_lib/verilog_model/sky130_fd_sc_hd.v \
blocking_caveat_net.v tb_blocking_caveat.v
```

Run:

```bash
./a.out
```

Open GTKWave:

```bash
gtkwave tb_blocking_caveat.vcd
```
<img width="1815" height="891" alt="Screenshot 2026-11-11 160959" src="https://github.com/user-attachments/assets/d2ad5bfe-f50f-452f-af83-df4c2892d16e" />


### Observation

- Blocking assignments execute in procedural order.
- The order of statements affects the values observed during simulation.
- The RTL simulation may use a previous intermediate value.
- The synthesized combinational circuit represents the logic relationship inferred from the RTL.

**Result:** Blocking assignment order must be considered carefully, especially when intermediate signals are used in combinational logic.

---






## 8. Key Observations

1. Gate-Level Simulation provides an additional verification step after synthesis by checking the behavior of the generated netlist.
2. The same testbench can be used for RTL and functional gate-level simulation.
3. An incomplete sensitivity list can produce incorrect RTL simulation results even when synthesis generates the intended combinational circuit.
4. Blocking assignments execute immediately, making statement order important.
5. Non-blocking assignments evaluate right-hand-side expressions before applying the scheduled updates, making them suitable for sequential logic.
6. Comparing RTL and GLS waveforms helps identify differences between simulated RTL behavior and the synthesized implementation.

## 9. Key Learnings

- Understood the purpose and importance of Gate-Level Simulation in the RTL-to-netlist verification process.
- Learned to use Icarus Verilog, Yosys and GTKWave together for RTL and gate-level verification.
- Identified the effect of missing signals in combinational sensitivity lists.
- Studied how blocking assignment ordering influences the behavior of procedural code.
- Understood why non-blocking assignments are preferred for clocked sequential logic.
- Observed how different coding styles can influence the hardware inferred during synthesis.

## 10. Tools Used

| Tool | Purpose |
|---|---|
| Icarus Verilog | RTL and gate-level simulation |
| Yosys | RTL synthesis and netlist generation |
| GTKWave | Viewing simulation waveforms |
| SKY130 HD Library | Standard-cell definitions and simulation models |
| Ubuntu | Development and execution environment |

---

**Conclusion**

Day 4 provided practical exposure to post-synthesis verification and the importance of writing correct Verilog RTL. The experiments demonstrated how incomplete sensitivity lists and inappropriate assignment styles can lead to differences between RTL simulation and synthesized hardware. Gate-Level Simulation helps verify that the synthesized implementation retains the expected functionality.
