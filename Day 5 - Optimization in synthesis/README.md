# Day 5 – Optimization in Synthesis

## Introduction

Synthesis is the process of converting Verilog RTL (Register Transfer Level) code into a hardware representation consisting of logic gates, multiplexers, flip-flops, latches, and other components.

The coding style used to describe a design has a direct influence on the hardware inferred by the synthesis tool. Even when two RTL descriptions perform the same function, their synthesized implementations may differ.

Day 5 focuses on understanding how conditional statements, loops, and case constructs affect hardware generation and how proper RTL coding practices help achieve the intended synthesis results.

## Index

- [1. Introduction to Synthesis Optimization](#1-introduction-to-synthesis-optimization)
- [2. Conditional Statements and MUX Inference](#2-conditional-statements-and-mux-inference)
- [3. Latch Inference](#3-latch-inference-in-combinational-logic)
- [4. Case Statements](#4-case-statements)
- [5. Case, Casez and Casex](#5-case-casez-and-casex)
- [6. Lab Experiments](#6-lab-experiments)
  - [Lab 1 – Incomplete If Statement](#lab-1--incomplete-if-statement)
  - [Lab 2 – Multiple Incomplete If Conditions](#lab-2--multiple-incomplete-if-conditions)
  - [Lab 3 – Complete and Incomplete Case Statements](#lab-3--complete-and-incomplete-case-statements)
  - [Lab 4 – Partial Assignments in Case Statements](#lab-4--partial-assignments-in-case-statements)
  - [Lab 5 – Overlapping Case Conditions](#lab-5--overlapping-case-conditions)
  - [Lab 6 – MUX Implementation Using For Loop](#lab-6--mux-implementation-using-for-loop)
  - [Lab 7 – DEMUX Using Loop](#lab-7--demux-using-loop)
  - [Lab 8 – DEMUX Using Case Statement](#lab-8--demux-using-case-statement)
  - [Lab 9 – Generate For Loop](#lab-9--generate-for-loop)
  - [Lab 10 – Ripple Carry Adder Using Generate](#lab-10--ripple-carry-adder-using-generate)
- [7. Key Learnings](#7-key-learnings)
- [8. Tools Used](#8-tools-used)

---

## 1. Introduction to Synthesis Optimization

RTL synthesis does more than translate Verilog statements into gates. It interprets the behavior described in the code and determines the hardware required to implement that behavior.

The following coding constructs are particularly important:

- Conditional statements (`if-else`) for describing selection and priority logic.
- `case` statements for implementing selection circuits.
- Complete and incomplete assignments for determining whether combinational logic or storage elements are required.
- Procedural `for` loops for describing repetitive operations.
- Generate loops for creating repeated hardware structures during elaboration.
- Different RTL implementations of MUXes, DEMUXes, and adders.

The experiments in this session demonstrate how these constructs influence the resulting hardware.

## 2. Conditional Statements and MUX Inference

Conditional statements are commonly used to select between multiple data inputs.

Consider the following RTL description:

```verilog
always @(*)
begin
    if (sel1)
        y = a;
    else if (sel2)
        y = b;
    else
        y = c;
end
```

The conditions are evaluated sequentially. The first condition has the highest priority, and the next condition is considered only when the previous condition is false.

**Hardware interpretation:**

The synthesis tool implements this selection behavior using multiplexers. Multiple conditional branches can result in cascaded MUX structures.

This demonstrates how an `if-else` chain describes priority-based selection logic.

## 3. Latch Inference in Combinational Logic

A latch is inferred when a variable inside a combinational block is not assigned a value under every possible condition.

For example:

```verilog
always @(*)
begin
    if (sel)
        y = a;
end
```

In this example, `y` receives a new value only when `sel` is high. When `sel` becomes low, no new assignment is provided.

The synthesis tool must therefore preserve the previous value of `y`, which requires a storage element.

**Important observations:**

- When `sel = 1`, the output follows the input `a`.
- When `sel = 0`, the output retains its previous value.
- A D latch is inferred during synthesis.

To avoid unintended latch inference, every output in a combinational block should receive an assignment for every possible execution path.

## 4. Case Statements

A `case` statement provides an alternative method for describing selection logic.

Example:

```verilog
always @(*)
begin
    case (sel)
        2'b00: y = i0;
        2'b01: y = i1;
        2'b10: y = i2;
        2'b11: y = i3;
        default: y = 1'b0;
    endcase
end
```

Each case item specifies the output corresponding to a particular select value.

Unlike an `if-else if` chain, a normal `case` statement does not establish priority between its mutually exclusive items.

**Hardware interpretation:**

The synthesis tool maps the selection behavior into combinational logic, typically a multiplexer structure.

A complete case statement ensures that the output receives a value for all possible select conditions.

## 5. Case, Casez and Casex

Verilog provides three commonly used case constructs with different matching rules.

| Construct | Matching behavior |
|---|---|
| `case` | Exact four-state matching |
| `casez` | Treats `Z` and `?` as wildcards |
| `casex` | Treats `X`, `Z`, and `?` as wildcards |

### Understanding wildcard matching

Consider:

```verilog
casez (sel)
    2'b00: y = i0;
    2'b01: y = i1;
    2'b10: y = i2;
    2'b1?: y = i3;
endcase
```

The pattern `2'b1?` matches both `2'b10` and `2'b11`.

This creates an overlap with the earlier `2'b10` condition.

**Points to remember:**

- `case` performs exact matching.
- `casez` supports wildcard matching for `Z` and `?`.
- `casex` treats unknown and high-impedance values as wildcards.
- Overlapping wildcard patterns should be avoided.
- `casex` is generally discouraged in synthesizable RTL because it can hide unknown values and create simulation-versus-synthesis mismatches.

---

## 6. Lab Experiments

The following experiments demonstrate the relationship between RTL coding style, simulation behavior, and the hardware inferred during synthesis.

### Lab 1 – Incomplete If Statement

**Objective:** To understand how an incomplete conditional assignment results in latch inference.

In this experiment, the output is assigned only when the input condition is satisfied.

When the condition is false, the output has no new assignment and must retain its previous value.

**Observation:**

- During RTL simulation, the output follows the selected input when the condition is active.
- When the condition becomes inactive, the output holds its previous state.
- The Yosys synthesis schematic confirms the presence of a D latch.

**Screenshot – Lab 1**
<img width="1877" height="908" alt="incompp_if" src="https://github.com/user-attachments/assets/6d63b372-d2ee-4d57-811f-0891d79e2b3a" />


### Lab 2 – Multiple Incomplete If Conditions

**Objective:** To examine latch inference when multiple conditional branches do not cover every possible input combination.

This experiment uses an `if-else if` structure to select between two data inputs.

The first condition determines whether the first input is selected. If it is false, the second condition is evaluated.

When both conditions are false, the output is not updated.

**Observation:**

- The first condition has priority over the second.
- The second input is selected only when the first condition is false.
- When neither condition is active, the output retains its previous value.
- Synthesis produces selection logic along with a latch.

**Screenshot – Lab 2**

<img width="1866" height="915" alt="incomp_if2" src="https://github.com/user-attachments/assets/722052b8-b472-4471-bba4-dab0e11bf230" />


### Lab 3 – Complete and Incomplete Case Statements

**Objective:** To compare the hardware generated by complete and incomplete case statements.

A complete case statement provides an assignment for every possible selection condition, usually through a `default` branch.

An incomplete case statement does not specify an assignment for every possible condition.

**Observation:**

- The complete case statement describes combinational selection logic.
- The incomplete case statement leaves some conditions without assignments.
- For unspecified conditions, the output must retain its previous value.
- Yosys infers a latch for the incomplete implementation.

This experiment highlights the importance of covering all possible input combinations in combinational logic.

**Screenshot – Lab 3**

<img width="1807" height="932" alt="incomp_case" src="https://github.com/user-attachments/assets/19772c32-11af-42d0-9321-9f17d5a679f0" />

----------------

<img width="1881" height="932" alt="comp_case" src="https://github.com/user-attachments/assets/df3acb7a-d652-457f-b1ff-1446c06df87a" />



### Lab 4 – Partial Assignments in Case Statements

**Objective:** To understand why a latch can be inferred even when a `default` branch is present.

In this experiment, multiple outputs are assigned within a case statement. However, one of the outputs is not assigned in every branch.

Even though a default condition exists, the missing assignment in another branch causes the affected output to retain its previous value.

**Observation:**

- An output assigned in every branch is implemented using combinational logic.
- An output with a missing assignment requires storage.
- Yosys identifies the missing assignment and infers a latch for the affected output.

**Key point:** Providing a `default` branch alone does not guarantee latch-free logic. Every output must be assigned in every branch.

**Screenshot – Lab 4**

<img width="1876" height="941" alt="partial_case_assign" src="https://github.com/user-attachments/assets/dbeb8974-9a86-498b-8f38-9da07ceb38ad" />


### Lab 5 – Overlapping Case Conditions

**Objective:** To investigate the effect of overlapping wildcard patterns on RTL simulation and synthesized hardware.

This experiment explores the use of overlapping conditions in case statements, particularly when wildcard matching is introduced using `casez`.

When two case patterns match the same input combination, the intended selection behavior may become ambiguous.

**Observation:**

- Wildcard patterns can match multiple input combinations.
- Overlapping conditions can create differences between simulation and synthesized implementations.
- The RTL and post-synthesis waveforms can be compared to identify behavioral differences.
- Properly defined, mutually exclusive case conditions help prevent unexpected synthesis results.

**Screenshot – Lab 5**

<img width="1902" height="927" alt="bad_case" src="https://github.com/user-attachments/assets/19e69a53-9135-437a-ac20-fc2412970762" />


### Lab 6 – MUX Implementation Using For Loop

**Objective:** To implement a multiplexer using a procedural `for` loop and understand how the loop is converted into hardware.

A procedural `for` loop is written inside an `always` block. It allows repetitive operations to be described without manually writing every selection condition.

For example, a four-input MUX can be implemented by combining the inputs into a vector and comparing the loop index with the select signal.

**Observation:**

- The loop iterates through the input indices.
- The matching index determines which input is assigned to the output.
- During synthesis, the loop is unrolled into hardware.
- A latch can be inferred if the output is not assigned a default value before the loop.

For combinational logic, a default assignment should be provided before the loop to ensure that the output is always assigned.

**Screenshot – Lab 6**

<img width="1910" height="922" alt="mux_generate" src="https://github.com/user-attachments/assets/04ff862b-e7fc-4ae0-9aea-94d47df3fdf0" />


### Lab 7 – DEMUX Using Loop

**Objective:** To implement a demultiplexer using a loop-based RTL description.

A demultiplexer directs a single input to one of several outputs depending on the select signal.

Using a loop, the repeated output selection operations can be described systematically rather than writing separate assignments for every output.

**Observation:**

- The select input determines the destination output.
- Only the selected output receives the input data.
- The remaining outputs retain their defined default values.
- Yosys converts the loop-based description into the corresponding selection and routing logic.

**Screenshot – Lab 7**

<img width="1916" height="942" alt="Demux_degenerate" src="https://github.com/user-attachments/assets/803f0edf-07d8-4f11-922d-095150aa9790" />


### Lab 8 – DEMUX Using Case Statement

**Objective:** To implement a demultiplexer using a `case` statement and compare it with the loop-based implementation.

The same demultiplexer functionality can be described using individual case items corresponding to different select values.

Each case branch assigns the input to the appropriate output while the other outputs are assigned their default values.

**Observation:**

- The select signal determines which output receives the input.
- The implementation produces the same intended functionality as the loop-based DEMUX.
- RTL and post-synthesis simulation can be compared to verify functional equivalence.
- The Yosys schematic represents the synthesized demultiplexer logic.

This experiment demonstrates that different RTL coding styles can describe the same hardware functionality.

**Screenshot – Lab 8**

<img width="1891" height="936" alt="demux_case" src="https://github.com/user-attachments/assets/0d4ab191-754e-43f8-b6d2-9a0d20c4b362" />


### Lab 9 – Generate For Loop

**Objective:** To understand how generate loops are used to replicate hardware structures.

A `generate for` loop is different from a procedural `for` loop. It is used to create multiple hardware instances during elaboration rather than describing repeated procedural operations.

A `genvar` variable is used as the index for generating the required hardware structures.

Example:

```verilog
genvar i;

generate
    for (i = 0; i < N; i = i + 1)
    begin
        // Repeated hardware instantiation
    end
endgenerate
```

Generate constructs are useful for:

- Repeated module instantiations.
- Multi-bit arithmetic circuits.
- Full-adder chains.
- Parameterized hardware designs.
- Repetitive combinational structures.

A conditional generate statement can also be used to instantiate hardware depending on a condition evaluated during elaboration.

**Observation:**

The generate loop expands into individual hardware instances. Unlike a procedural loop, it is not implemented as a runtime counter.



### Lab 10 – Ripple Carry Adder Using Generate

**Objective:** To design a Ripple Carry Adder using a generate loop and understand the cascading carry structure.

A Ripple Carry Adder (RCA) is constructed by connecting multiple full-adder stages.

Each full adder receives two input bits and a carry-in. Its carry-out is connected to the carry-in of the next stage.

A generate loop makes it possible to replicate the full-adder module for each bit position.

**Working principle:**

- The first full-adder stage receives the initial carry input.
- Each stage performs addition of its corresponding input bits.
- The carry generated by one stage is passed to the next stage.
- The final stage produces the most significant carry output.

For two unsigned `n`-bit operands, the result may require `n+1` bits to accommodate the final carry.

**Observation:**

- RTL simulation verifies the addition functionality.
- Post-synthesis simulation checks the synthesized implementation.
- The Yosys schematic shows the repeated full-adder structure.
- The carry propagates from one stage to the next, demonstrating the operation of a Ripple Carry Adder.

**Screenshot – Lab 10**

<img width="1871" height="960" alt="rca" src="https://github.com/user-attachments/assets/a1ade164-310d-4698-a69d-71d7bcbc00e2" />


---

## 7. Key Learnings

1. Conditional statements can infer cascaded MUX structures depending on the priority of the conditions.
2. Incomplete assignments in combinational blocks result in latch inference because the previous output value must be retained.
3. Complete case statements help describe combinational logic without unintended storage elements.
4. Every output must receive an assignment in every possible branch to avoid unintended latch inference.
5. Overlapping wildcard patterns can cause unexpected behavior and RTL-to-synthesis mismatches.
6. Procedural `for` loops are unrolled into hardware during synthesis and do not necessarily create hardware counters.
7. Generate loops are used to replicate hardware structures during elaboration.
8. MUX and DEMUX functionality can be described using different RTL coding approaches.
9. Ripple Carry Adders can be constructed efficiently by replicating full-adder modules using generate constructs.
10. Proper RTL coding practices are essential for obtaining the intended synthesized hardware.

## 8. Tools Used

| Tool | Purpose |
|---|---|
| Icarus Verilog | RTL simulation |
| Yosys | RTL synthesis and hardware visualization |
| GTKWave | Simulation waveform analysis |
| SKY130 Standard Cell Library | Standard-cell mapping and gate-level simulation |

---

## Conclusion

Day 5 provided an understanding of how Verilog coding styles influence hardware synthesis. Through the experiments, I explored conditional logic, latch inference, case statements, procedural loops, and generate constructs.

The comparison of RTL simulation, post-synthesis simulation, and Yosys schematics helped demonstrate how the written RTL description is translated into actual hardware structures.

These concepts are important for writing efficient, predictable, and synthesis-friendly RTL designs.
