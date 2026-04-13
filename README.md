# Low Power Area Efficient ALU with Low Power Full Adder

> A 4-bit Arithmetic Logic Unit (ALU) designed with XNOR-based full adder logic to achieve ~50% reduction in power consumption compared to conventional designs.

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [ALU Operations](#alu-operations)
- [Full Adder Design](#full-adder-design)
- [Control Signal Truth Table](#control-signal-truth-table)
- [Two's Complement Subtraction](#twos-complement-subtraction)
- [Verification & Testing](#verification--testing)
- [Results](#results)
- [Tools Used](#tools-used)
- [Project Timeline](#project-timeline)
- [What I Learned](#what-i-learned)

---

## Overview

This project presents the design and verification of a **4-bit low-power ALU** optimized for embedded systems and small processors where energy efficiency and silicon area are critical constraints.

The key innovation is replacing the conventional AND/OR/XOR-based full adder with an **XNOR-based full adder**, which significantly reduces switching activity — the primary source of dynamic power consumption in CMOS digital circuits.

The ALU supports **6 operations** (ADD, SUB, AND, OR, XOR, XNOR) selectable via a 3-bit control signal, making it compact, flexible, and suitable for integration into low-power processor datapaths.

---

## Features

- 4-bit datapath with 6 selectable operations
- XNOR-based full adder — reduces switching transitions, cuts dynamic power by ~50%
- 3-bit control signal (s0, s1, s2) for operation selection
- Two's complement subtraction using the existing adder hardware
- Exhaustive simulation across all 256 input combinations (16 × 16)
- Verified for correctness, power, area, and timing

---

## Architecture

```
         ┌────────────────────────────────────┐
  A[3:0] │                                    │
─────────►                                    ├──► Result[3:0]
         │         4-bit ALU                  │
  B[3:0] │                                    ├──► Carry_Out
─────────►                                    │
         │                                    ├──► Zero Flag
s2,s1,s0 │                                    │
─────────►                                    │
         └────────────────────────────────────┘
```

The ALU consists of:
- **4 XNOR-based full adder cells** chained as a ripple-carry adder
- **Logic operation units** (AND, OR, XOR, XNOR) operating in parallel on inputs A and B
- **A multiplexer** controlled by s0, s1, s2 to select and route the appropriate result to the output
- **B-input inversion + carry-in** for two's complement subtraction

---

## ALU Operations

| Operation | Description                       | Control (s2 s1 s0) |
|-----------|-----------------------------------|--------------------|
| ADD       | A + B                             | 0 0 0              |
| SUB       | A − B (via two's complement)      | 0 0 1              |
| AND       | A AND B (bitwise)                 | 0 1 0              |
| OR        | A OR B (bitwise)                  | 0 1 1              |
| XOR       | A XOR B (bitwise)                 | 1 0 0              |
| XNOR      | A XNOR B (bitwise)                | 1 0 1              |

---

## Full Adder Design

### Conventional Design (Standard)

A traditional 1-bit full adder computes:

```
Sum   = A XOR B XOR Cin
Carry = (A AND B) OR (B AND Cin) OR (A AND Cin)
```

This uses a mix of AND, OR, and XOR gates with high gate count and frequent logic transitions, leading to elevated dynamic power consumption.

### Proposed Design (XNOR-Based)

The redesigned full adder restructures the sum and carry equations to maximize use of XNOR gates:

```
Sum   = (A XNOR B) XNOR Cin
Carry = derived from XNOR intermediate signals
```

**Why XNOR reduces power:**
- XNOR gates produce fewer signal transitions (0→1 or 1→0) for typical input patterns
- Fewer transitions → less charging/discharging of node capacitances
- Dynamic power: P = α · C · V² · f — reducing α (activity factor) directly reduces power
- Reusing intermediate XNOR signals for both Sum and Carry avoids redundant gate paths

This redesign achieves approximately **50% reduction in power consumption** compared to the conventional full adder at the same operating frequency.

---

## Control Signal Truth Table

The 3-bit control input (s2, s1, s0) drives a multiplexer that selects among the parallel functional units:

| s2 | s1 | s0 | Selected Operation |
|----|----|----|-------------------|
| 0  | 0  | 0  | ADD               |
| 0  | 0  | 1  | SUB               |
| 0  | 1  | 0  | AND               |
| 0  | 1  | 1  | OR                |
| 1  | 0  | 0  | XOR               |
| 1  | 0  | 1  | XNOR              |
| 1  | 1  | 0  | Reserved          |
| 1  | 1  | 1  | Reserved          |

All functional units operate in parallel; only the selected output is passed to the result bus. This avoids sequential reconfiguration and keeps the critical path delay uniform across operations.

---

## Two's Complement Subtraction

Subtraction (A − B) is implemented using two's complement arithmetic — no dedicated subtractor hardware is needed:

```
A − B  =  A + (NOT B) + 1
```

**Hardware implementation:**
- When s0 = 1 (SUB mode), each bit of B is inverted through XOR gates (XOR with s0 acts as a conditional inverter)
- The carry-in (Cin) of the least-significant full adder cell is forced to 1
- The existing ripple-carry adder computes the result

This reuse of adder hardware for subtraction significantly reduces area overhead compared to implementing a separate subtractor.

---

## Verification & Testing

### Test Coverage

- **Input space:** All combinations of 4-bit inputs A[3:0] and B[3:0] → 16 × 16 = **256 input pairs**
- **Operations tested:** All 6 operations for each input pair
- **Total test vectors:** 1,536 unique test cases

### Verification Methodology

1. **Functional simulation** — Each operation verified against expected outputs derived from Boolean equations
2. **Corner case testing** — Overflow detection (e.g., 0xF + 0x1), zero result (A − A), all-ones, all-zeros
3. **Power simulation** — Toggle activity measured across all input patterns to confirm reduced switching
4. **Timing analysis** — Critical path delay verified to meet target frequency
5. **Area analysis** — Gate count and layout area compared against conventional ALU baseline

### Sample Test Vectors

| A (4-bit) | B (4-bit) | Operation | Expected Output | Carry Out |
|-----------|-----------|-----------|-----------------|-----------|
| 0101      | 0011      | ADD       | 1000            | 0         |
| 1010      | 0110      | SUB       | 0100            | 1         |
| 1100      | 1010      | AND       | 1000            | —         |
| 0110      | 0011      | OR        | 0111            | —         |
| 1111      | 1010      | XOR       | 0101            | —         |
| 1001      | 1001      | XNOR      | 1111            | —         |

---

## Results

| Metric              | Conventional ALU | This Design       | Improvement     |
|---------------------|-----------------|-------------------|-----------------|
| Power Consumption   | Baseline         | ~50% of baseline  | ~50% reduction  |
| Full Adder Gate Count | Higher          | Lower (XNOR reuse)| Area efficient  |
| Supported Operations| 6               | 6                 | —               |
| Control Bits        | 3               | 3                 | —               |
| Input Width         | 4-bit           | 4-bit             | —               |
| Verification        | —               | 1,536 test vectors| Exhaustive      |

---

## Tools Used

| Tool / Technology      | Purpose                                      |
|------------------------|----------------------------------------------|
| Hardware Description Language (HDL) | RTL design of ALU and full adder |
| Digital Simulation Tool | Functional and timing simulation            |
| Logic Synthesis Tool   | Gate-level netlist generation and area report|
| Power Analysis Tool    | Dynamic power estimation via toggle activity |

---

## Project Timeline

**January 2023 – July 2023**

| Phase | Duration | Activity |
|-------|----------|----------|
| Literature Review | Jan – Feb 2023 | Studied conventional full adder designs and low-power techniques |
| Full Adder Design | Feb – Mar 2023 | Designed and simulated XNOR-based full adder |
| ALU Integration | Mar – May 2023 | Implemented 4-bit ALU with control logic and all 6 operations |
| Verification | May – Jun 2023 | Exhaustive simulation, power measurement, timing analysis |
| Optimization & Report | Jun – Jul 2023 | Refinement and documentation |

---

## What I Learned

- **Digital logic design** — Gate-level and RTL design for arithmetic and logic circuits
- **Low-power design techniques** — Minimizing switching activity as a power reduction strategy
- **Control signal architecture** — Using multiplexers and binary encoding for flexible operation selection
- **Hardware verification** — Writing exhaustive test benches and interpreting simulation results
- **Two's complement arithmetic** — Hardware-efficient subtraction by inverting and incrementing
- **Area-performance-power trade-offs** — Understanding how design choices ripple across multiple metrics simultaneously

---

## Author

**[Your Name]**  
B.Tech / M.Tech in Electronics / VLSI Design  
[Your Institution]  
[Your Email or LinkedIn]

---

## License

This project is for academic and educational purposes.  
Feel free to reference or build upon this work with attribution.
