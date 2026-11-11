# Day 3 – Introduction to RTL Logic Optimizations

## Overview

This section focuses on understanding how synthesis tools optimize Verilog RTL designs to reduce hardware complexity while maintaining the intended functionality.

The experiments explore combinational and sequential logic optimization techniques using Yosys. Different Verilog designs are synthesized to observe how unnecessary gates, registers, and logic connections are identified and removed.

The practical exercises also demonstrate how RTL coding styles and signal dependencies influence the final synthesized circuit.

## Table of Contents

- [1. Introduction to Logic Optimization](#1-introduction-to-logic-optimization)
- [2. Combinational Logic Optimization](#2-combinational-logic-optimization)
  - [2.1 Constant Propagation](#21-constant-propagation)
  - [2.2 Boolean Logic Simplification](#22-boolean-logic-simplification)
  - [2.3 Practical Synthesis Experiments](#23-practical-synthesis-experiments)
- [3. Sequential Logic Optimization](#3-sequential-logic-optimization)
  - [3.1 Sequential Constant Propagation](#31-sequential-constant-propagation)
  - [3.2 Flip-Flop Dependency](#32-flip-flop-dependency)
  - [3.3 Register Optimization](#33-register-optimization)
- [4. Optimization of Unused Outputs](#4-optimization-of-unused-outputs)
- [5. Important Yosys Commands](#5-important-yosys-commands)
- [6. Key Learnings](#6-key-learnings)
- [7. Conclusion](#7-conclusion)

---

## 1. Introduction to Logic Optimization

Logic optimization is an important stage in digital circuit synthesis. It transforms the RTL description into a simpler and more efficient hardware implementation without changing its required functionality.

The primary objectives are:

- Reducing the number of logic gates and flip-flops.
- Minimizing hardware area.
- Improving circuit performance.
- Eliminating redundant logic and unused signals.
- Reducing unnecessary switching activity and power consumption.

Logic optimization is broadly classified into two categories:

1. Combinational Logic Optimization
2. Sequential Logic Optimization

## 2. Combinational Logic Optimization

Combinational optimization deals with circuits whose outputs depend only on their current inputs.

Yosys applies different optimization techniques to simplify Boolean expressions and eliminate unnecessary logic.

### 2.1 Constant Propagation

Constant propagation is a technique in which signals having fixed logic values (0 or 1) are substituted into the logic expressions.

Once constant values are identified, the synthesis tool simplifies the remaining circuit by removing redundant operations.

**Example:**

Consider a 2:1 multiplexer with one of its data inputs permanently connected to logic 0.

The multiplexer can be simplified into an AND gate, depending on the input and select connections.

This reduces the number of hardware components required to implement the same functionality.

**Observation:**

Constant propagation helps eliminate unnecessary logic by identifying signals that do not need to be dynamically evaluated.

### 2.2 Boolean Logic Simplification

Boolean optimization reduces complex logic expressions using Boolean algebraic identities and minimization techniques.

Synthesis tools can use methods such as:

- Karnaugh Map (K-Map) minimization.
- Quine–McCluskey algorithm.
- Boolean algebraic transformations.

**Example:**

Consider the following RTL expression:

```verilog
assign y = a ? (b ? c : (c ? a : 0)) : (!c);
```

The expression can be simplified mathematically into an equivalent Boolean function.

Yosys performs such transformations to obtain a simpler gate-level implementation.

**Observation:**

A complicated RTL expression does not necessarily result in complicated hardware. The synthesis tool evaluates the logic and attempts to find an equivalent, optimized implementation.

### 2.3 Practical Synthesis Experiments

The following experiments demonstrate how Yosys optimizes different combinational logic designs.

**Step 1: Launch Yosys**

```bash
yosys
```

**Step 2: Load the SKY130 standard-cell library**

```tcl
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

**Step 3: Read the Verilog design**

```tcl
read_verilog opt_check.v
```

**Step 4: Perform synthesis**

```tcl
synth -top opt_check
```

**Step 5: Remove unused cells and wires**

```tcl
opt_clean -purge
```

**Step 6: Perform technology mapping**

```tcl
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

**Step 7: Display the synthesized circuit**

```tcl
show
```

### Lab 1 – opt_check.v

The initial design contains a multiplexer with a constant input. After optimization, the logic is reduced to an equivalent two-input AND gate.

**Screenshot – RTL / Synthesized Circuit**

<img width="1897" height="938" alt="Screenshot 2026-11-03 173154" src="https://github.com/user-attachments/assets/b072288b-2cb9-4c35-ba66-591563f33126" />



### Lab 2 – opt_check2.v

The second design demonstrates simplification of a multiplexer with a constant logic input. The resulting optimized circuit is equivalent to an OR gate.

**Screenshot – Synthesized Circuit**

<img width="1901" height="925" alt="Screenshot 2026-11-03 173426" src="https://github.com/user-attachments/assets/3b853dc0-fd8f-46d2-bf8a-89f15a52e990" />


### Lab 3 – opt_check3.v

This experiment demonstrates how constant propagation and Boolean simplification can reduce a more complex logic network.

The resulting circuit is simplified to a three-input AND gate.

**Screenshot – Synthesized Circuit**

<img width="1887" height="956" alt="Screenshot 2026-11-03 173629" src="https://github.com/user-attachments/assets/488763b4-2623-42f9-80ac-1a32df6659be" />


### Lab 4 – opt_check4.v

This design explores another Boolean simplification case. The original logic is optimized into an XNOR-equivalent implementation.

**Screenshot – Synthesized Circuit**

<img width="1917" height="978" alt="Screenshot 2026-11-03 165243" src="https://github.com/user-attachments/assets/d070b901-5d97-478c-8b9d-35af8563384d" />


### Lab 5 – multiple_module_opt.v

This experiment introduces hierarchical Verilog designs containing multiple modules.

The design hierarchy is flattened before optimization so that Yosys can analyze the combined logic and simplify it.

Commands:

```tcl
read_verilog multiple_module_opt.v
synth -top multiple_module_opt
flatten
opt_clean -purge
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
show
```

**Observation:** Flattening allows optimization across module boundaries. The resulting implementation contains simplified logic.

**Screenshot – Flattened Design**

<img width="1917" height="936" alt="Screenshot 2026-11-03 171940" src="https://github.com/user-attachments/assets/68f48df1-3dea-4c9b-b95e-2b325f07e45c" />


### Lab 6 – multiple_module_opt2.v

A second hierarchical design is synthesized to study the effect of flattening and optimization on interconnected modules.

The same synthesis procedure is followed to examine the optimized gate-level structure.

**Screenshot – Synthesized Circuit**

<img width="1917" height="923" alt="Screenshot 2026-11-03 171617" src="https://github.com/user-attachments/assets/47682968-f2c2-4d04-a495-04841862b72a" />


---

## 3. Sequential Logic Optimization

Sequential circuits contain memory elements such as flip-flops. Their outputs depend on both current inputs and previously stored states.

Unlike combinational optimization, sequential optimization must preserve the required clock-dependent behavior of the circuit.

Some important sequential optimization techniques include:

- Sequential constant propagation.
- State optimization.
- Retiming.
- Sequential logic cloning.
- Removal of redundant registers.

### 3.1 Sequential Constant Propagation

Sequential constant propagation identifies registers whose outputs can be determined to have fixed values under the specified reset, set, and data conditions.

When a flip-flop is found to have a constant output, Yosys may replace it with a constant connection and eliminate unnecessary sequential hardware.

#### Lab 1 – dff_const1.v and dff_const2.v

These designs are simulated and synthesized to understand how reset and data inputs influence the behavior of flip-flops.

The simulation waveforms are examined first, followed by synthesis to observe the resulting hardware.

**GTKWave Simulation Results**

<img width="1866" height="911" alt="Screenshot 2026-11-11 222450" src="https://github.com/user-attachments/assets/d4e1f6ab-f29d-4476-9fc9-772c97ff9685" />

-----------

<img width="1822" height="908" alt="Screenshot 2026-11-11 223546" src="https://github.com/user-attachments/assets/2029c9cb-8b34-4f95-9639-3861cf26ee09" />



**Synthesized Circuit Comparison**

<img width="1887" height="928" alt="Screenshot 2026-11-11 223423" src="https://github.com/user-attachments/assets/41c57826-2685-4905-9636-92ea8646eebe" />


**Observation:**

The experiments demonstrate that constant values at the flip-flop inputs can simplify sequential hardware. The reset polarity and the relationship between reset and data inputs determine whether the register can be eliminated.

### 3.2 Flip-Flop Dependency

#### Lab 2 – dff_const3.v

This experiment studies the dependency between two flip-flops.

The output of one sequential element is connected to the set input of another. Because the second flip-flop responds according to its clock and control behavior, the output cannot be treated as an immediate constant without considering the timing relationship.

The design is simulated and synthesized to observe how the sequential elements behave.

**GTKWave Results and Synthesized Circuit**

<img width="1871" height="921" alt="Screenshot 2026-11-11 224445" src="https://github.com/user-attachments/assets/a9e64948-0ee9-4ffa-abf5-bb08b4d8ebae" />


**Observation:**

The flip-flops are retained because their behavior depends on sequential transitions and the relationship between their control signals.

### 3.3 Register Optimization

#### Lab 3 – dff_const4.v

This experiment demonstrates a case where the set and reset conditions allow the sequential elements to be simplified.

The RTL is synthesized to observe how constant sequential behavior can eliminate unnecessary flip-flops.

**Screenshot – Synthesized Circuit**

<img width="1881" height="942" alt="Screenshot 2026-11-11 225007" src="https://github.com/user-attachments/assets/10cc299d-8c90-41e3-8ac5-d74a0efe1bf9" />


#### Lab 4 – dff_const5.v

This design provides another example of sequential logic behavior with different reset conditions.

The synthesis result is examined to understand why the flip-flops are preserved instead of being replaced by constant connections.

**Screenshot – Synthesized Circuit**

<img width="1895" height="951" alt="Screenshot 2026-11-11 225240" src="https://github.com/user-attachments/assets/ce44c22c-322c-4060-82e0-fa2ac00cd300" />


**Important Observation:**

A flip-flop cannot be removed simply because its data input is connected to a constant. Reset, set, clock, and other control conditions must also be considered to determine whether its output is truly constant.

---

## 4. Optimization of Unused Outputs

Synthesis tools also eliminate logic that does not contribute to any observable output or required circuit behavior.

This technique is known as unused logic removal.

### Lab 5 – counter_opt.v

Consider a three-bit counter:

```verilog
reg [2:0] count;
```

Suppose only the least significant bit, `count[0]`, is connected to the output.

The remaining bits, `count[1]` and `count[2]`, do not affect the observable output.

Yosys can therefore remove the unnecessary counter bits and reduce the hardware implementation.

**Screenshot – Original and Optimized Circuit**

<img width="1725" height="908" alt="Screenshot 2026-11-11 225840" src="https://github.com/user-attachments/assets/176a9191-1e96-4ad7-b9e6-de858a0f6af4" />


### Experiment 2 – counter_opt2.v

In this experiment, the output depends on the complete counter value:

```verilog
assign q = (count == 3'b100);
```

Since all three counter bits contribute to the comparison, they are required for the output functionality.

Consequently, the synthesis tool retains the necessary flip-flops and comparison logic.

**Screenshot – Synthesized Circuit**

<img width="1800" height="892" alt="Screenshot 2026-11-11 230407" src="https://github.com/user-attachments/assets/b72527f3-1d47-48d4-a50b-00a8c462fa24" />


**Comparison:**

The two experiments demonstrate that synthesis tools retain only the hardware necessary to produce the required outputs. Logic that does not influence observable behavior can be removed, whereas logic involved in output generation must be preserved.

---

## 5. Important Yosys Commands

The following commands were used throughout the optimization experiments.

| Command | Purpose |
|---|---|
| `read_verilog` | Loads the Verilog RTL source file |
| `read_liberty -lib` | Reads the standard-cell library |
| `synth -top` | Performs synthesis for the specified top module |
| `opt` | Runs general logic optimization passes |
| `opt_clean -purge` | Removes unused cells and wires |
| `flatten` | Combines hierarchical modules into a flat design |
| `abc -liberty` | Performs logic optimization and technology mapping |
| `show` | Displays the synthesized circuit |
| `stat` | Displays synthesis statistics |

---

## 6. Key Learnings

1. Understood how constant propagation reduces unnecessary combinational logic.
2. Explored Boolean simplification and its impact on gate-level circuit implementation.
3. Used Yosys optimization passes to remove redundant logic and simplify RTL designs.
4. Observed how reset, set, and clock conditions influence sequential optimization.
5. Studied the effect of hierarchical flattening on multi-module designs.
6. Understood how unused register bits and logic can be eliminated during synthesis.
7. Compared RTL functionality with the resulting synthesized gate-level implementations.

---

## 7. Conclusion

Day 3 provided practical exposure to combinational and sequential logic optimization using Verilog and Yosys.

Through multiple synthesis experiments, different optimization techniques were examined, including constant propagation, Boolean simplification, sequential constant optimization, hierarchy flattening, and unused logic removal.

These exercises demonstrate the importance of understanding how synthesis tools interpret RTL code and transform it into an efficient hardware implementation while preserving the intended functionality.

---

## Tools and Technologies

- **Verilog HDL** – RTL design description.
- **Icarus Verilog** – Functional simulation.
- **GTKWave** – Waveform visualization.
- **Yosys** – RTL synthesis and logic optimization.
- **SKY130 PDK** – Standard-cell technology library.
- **Ubuntu Linux** – Development environment.

## References

1. [VSD RTL Design and Synthesis Workshop – Day 3](https://github.com/ananyarf/VSD-RTL/tree/main/Day-03-Introduction-to-Optimizations)
2. [Yosys Documentation](https://yosyshq.net/yosys/)
3. [SkyWater SKY130 PDK](https://github.com/google/skywater-pdk)
4. [Icarus Verilog](https://github.com/steveicarus/iverilog)
5. [GTKWave](https://gtkwave.sourceforge.net/)

