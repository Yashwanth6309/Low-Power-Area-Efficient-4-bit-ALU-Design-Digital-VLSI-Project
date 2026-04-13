# Low Power Area Efficient ALU with Low Power Full Adder

> A 4-bit Arithmetic Logic Unit (ALU) designed for minimal power consumption and area efficiency using XNOR-based full adder logic — targeted at embedded systems and low-power digital processors.

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [ALU Operations & Control Signals](#alu-operations--control-signals)
- [Full Adder Design](#full-adder-design)
- [Two's Complement Subtraction](#twos-complement-subtraction)
- [Simulation & Verification](#simulation--verification)
- [Results](#results)
- [Tools Used](#tools-used)
- [Project Structure](#project-structure)
- [How to Run](#how-to-run)
- [Key Learnings](#key-learnings)
- [Author](#author)

---

## Overview

This project presents the design and implementation of a **4-bit low-power ALU** capable of performing arithmetic and logical operations. The primary innovation lies in replacing the conventional full adder with an **XNOR-based full adder**, significantly reducing switching activity and dynamic power consumption.

**Project Duration:** January 2023 – July 2023

---

## Features

- 4-bit ALU supporting 6 operations
- XNOR-based full adder with ~50% power reduction vs. standard designs
- 3-bit control signal interface (`s0`, `s1`, `s2`) for operation selection
- Two's complement subtraction logic
- Fully simulated and verified across all 256 input combinations (16 × 16)
- Optimized for area efficiency without performance trade-off

---

## Architecture

```
        ┌─────────────────────────────────────┐
        │            4-bit ALU                │
        │                                     │
 A[3:0]─►                                     │
        │    ┌────────────┐                   ├──► Result[3:0]
 B[3:0]─►    │  Operation │                   │
        │    │  Selector  │                   ├──► Carry Out
s0,s1,s2►    └────────────┘                   │
        │         │                           ├──► Zero Flag
        │    ┌────▼──────────────────────┐    │
        │    │  XNOR-based Full Adder   │    │
        │    │  (4-bit ripple carry)     │    │
        │    └───────────────────────────┘    │
        └─────────────────────────────────────┘
```

The ALU is built around a ripple-carry chain of 4 XNOR-based full adder cells. A multiplexer controlled by `s0`, `s1`, `s2` routes the correct operand and carry-in to the adder or directly passes the logical operation result.

---

## ALU Operations & Control Signals

| s2 | s1 | s0 | Operation | Description                     |
|----|----|----|-----------|----------------------------------|
|  0 |  0 |  0 | ADD       | A + B                            |
|  0 |  0 |  1 | SUB       | A - B (via two's complement)     |
|  0 |  1 |  0 | AND       | A AND B (bitwise)                |
|  0 |  1 |  1 | OR        | A OR B (bitwise)                 |
|  1 |  0 |  0 | XOR       | A XOR B (bitwise)                |
|  1 |  0 |  1 | XNOR      | A XNOR B (bitwise)               |

---

## Full Adder Design

### Traditional Full Adder
Standard full adders are built using AND, OR, and XOR gates. This leads to high switching activity, especially under frequently toggling inputs, increasing dynamic power.

**Standard equations:**
```
Sum   = A ⊕ B ⊕ Cin
Carry = (A · B) + (Cin · (A ⊕ B))
```

### XNOR-Based Full Adder (Proposed Design)
The redesigned full adder leverages XNOR gates to reduce the number of transitions, thereby lowering switching power.

**Optimized equations:**
```
Sum   = (A ⊙ B) ⊙ Cin        ; using XNOR for intermediate
Carry = derived using fewer gate transitions
```

**Why XNOR?**
- XNOR produces `1` when both inputs are equal (common in real data patterns)
- Fewer transitions → lower switching capacitance charged/discharged per cycle
- Results in approximately **50% power reduction** compared to the standard design

---

## Two's Complement Subtraction

Subtraction (`A - B`) is implemented using the two's complement method:

```
A - B  =  A + (~B) + 1
```

When `s0 = 1` (SUB mode):
- Input B is bitwise inverted before entering the adder
- Carry-in (`Cin`) is forced to `1`
- The adder computes `A + (~B) + 1`, which equals `A - B`

This eliminates the need for a dedicated subtractor circuit, saving area.

---

## Simulation & Verification

All operations were verified using a hardware simulation environment.

### Test Coverage
- **Inputs:** All 4-bit combinations → 16 values for A × 16 values for B = **256 test vectors**
- **Operations tested:** ADD, SUB, AND, OR, XOR, XNOR (each verified across all 256 input pairs)

### Verification Checklist

| Test Category        | Status |
|----------------------|--------|
| Addition correctness | ✅ Pass |
| Subtraction (2's complement) | ✅ Pass |
| AND / OR / XOR / XNOR | ✅ Pass |
| Carry propagation    | ✅ Pass |
| Overflow detection   | ✅ Pass |
| Power measurement    | ✅ Measured |
| Area estimation      | ✅ Measured |

---

## Results

| Metric              | Standard ALU | This Design  | Improvement     |
|---------------------|-------------|--------------|-----------------|
| Power Consumption   | Baseline    | ~50% lower   | ~50% reduction  |
| Gate Count (Area)   | Higher      | Reduced      | Area efficient  |
| Operating Speed     | Baseline    | Comparable   | No degradation  |
| Operations Supported| 6           | 6            | Same coverage   |

---

## Tools Used

| Tool / Technology     | Purpose                            |
|-----------------------|------------------------------------|
| HDL (Verilog / VHDL)  | Hardware description & design      |
| Logic Simulation Tool | Functional verification            |
| Schematic Editor      | Gate-level circuit design          |
| Waveform Viewer       | Output signal analysis             |
| Power Analysis Tool   | Dynamic power measurement          |

> *(Specify exact tools if applicable — e.g., ModelSim, Xilinx Vivado, Cadence, Synopsys)*

---

## Project Structure

```
low-power-alu/
│
├── src/
│   ├── full_adder.v          # XNOR-based full adder module
│   ├── alu_4bit.v            # Top-level 4-bit ALU module
│   └── control_unit.v        # Operation selector logic
│
├── testbench/
│   ├── tb_full_adder.v       # Testbench for full adder
│   └── tb_alu_4bit.v         # Testbench for ALU (all 256 combinations)
│
├── sim/
│   └── waveforms/            # Simulation output waveforms
│
├── docs/
│   ├── schematic.png         # Gate-level schematic
│   └── power_report.txt      # Power analysis report
│
└── README.md
```

---

## How to Run

### Prerequisites
- Any HDL simulator (ModelSim, Icarus Verilog, Vivado, etc.)

### Steps

```bash
# Clone the repository
git clone https://github.com/your-username/low-power-alu.git
cd low-power-alu

# Compile the design (Icarus Verilog example)
iverilog -o alu_sim src/full_adder.v src/alu_4bit.v testbench/tb_alu_4bit.v

# Run simulation
vvp alu_sim

# View waveforms (if using GTKWave)
gtkwave dump.vcd
```

---

## Key Learnings

- Digital logic design and gate-level optimization
- Low-power design techniques (switching activity reduction)
- XNOR-based circuit restructuring
- Control signal design for multi-function units
- Hardware verification methodology using exhaustive test vectors
- Area-power-performance trade-off analysis in ALU design

---

## Author

**[Your Name]**
B.Tech / M.Tech — Electronics / VLSI / Computer Engineering
[Your College Name] | [Year]

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue)](https://linkedin.com/in/your-profile)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-black)](https://github.com/your-username)

---

> *This project was completed as part of academic research into low-power VLSI design. Feel free to fork, reference, or build upon it.*
