# Low Power Area-Efficient ALU with Low Power Full Adder

![VLSI](https://img.shields.io/badge/Domain-VLSI%20Design-blue)
![Type](https://img.shields.io/badge/Type-Digital%20Logic-teal)
![Tool](https://img.shields.io/badge/Tool-Hardware%20Simulation-orange)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

A 4-bit Arithmetic Logic Unit (ALU) designed with a focus on **low power consumption** and **area efficiency**, using an XNOR-based full adder to reduce switching activity by approximately 50% compared to conventional designs.

---

## Overview

This project presents the design, simulation, and verification of a **4-bit ALU** capable of performing six operations controlled by a 3-bit select signal. The core innovation is replacing the traditional full adder with an XNOR-based design that significantly reduces power usage while maintaining correctness and speed.

---

## Features

- **6 operations**: ADD, SUB, AND, OR, XOR, XNOR
- **3-bit control signals** (s0, s1, s2) to select operations
- **~50% power reduction** over traditional full adder designs
- **Two's complement** subtraction — no separate subtractor circuit needed
- Fully verified across all 256 input combinations (4-bit × 4-bit)

---

## Operations Table

| s2 | s1 | s0 | Operation |
|----|----|----|-----------|
|  0 |  0 |  0 | ADD       |
|  0 |  0 |  1 | SUB       |
|  0 |  1 |  0 | AND       |
|  0 |  1 |  1 | OR        |
|  1 |  0 |  0 | XOR       |
|  1 |  0 |  1 | XNOR      |

---

## Architecture

```
         ┌─────────────────────────────────────┐
4-bit A ──►                                     │
         │   XNOR-Based Full Adder Core         ├──► 4-bit Output
4-bit B ──►   + Operation Control Logic         │
         │                                     ├──► Carry Out
s0,s1,s2──►   (3-bit select)                   │
         └─────────────────────────────────────┘
```

---

## Design Methodology

### Step 1 — XNOR-Based Full Adder
Traditional full adders rely heavily on AND/OR/XOR gate chains, causing high switching activity. This design uses **XNOR logic** to reduce transitions, cutting power consumption by ~50%.

### Step 2 — Control Signal Architecture
A 3-bit control word (s0, s1, s2) selects the active operation. Subtraction is handled using **two's complement** arithmetic, eliminating a dedicated subtractor and saving area.

### Step 3 — Verification
All 16 × 16 = 256 input combinations tested for each of the 6 operations. Simulated using a hardware description and simulation tool. Power, area, and timing metrics measured and optimized.

---

## Key Results

| Metric          | Traditional Design | This Design        |
|-----------------|--------------------|--------------------|
| Power usage     | Baseline           | ~50% reduction     |
| Operations      | Basic arithmetic   | 6 (arith + logic)  |
| Subtraction     | Separate circuit   | Two's complement   |
| Verification    | —                  | All 256 combos     |

---

## Tools & Technologies

- Digital logic design (gate-level)
- Hardware simulation (functional + timing verification)
- Two's complement arithmetic
- XNOR-based low-power logic design
- Control signal design

---

## Skills Demonstrated

- Digital circuit design and optimization
- Low-power design techniques
- Hardware verification and test case generation
- Embedded system design principles
- Control logic architecture

---

## Project Timeline

**Jan 2023 – Jul 2023**

---

## Author

**[Your Name]**  
[LinkedIn](https://linkedin.com/in/yourprofile) | [Email](mailto:your@email.com)

---

*Designed as part of academic/personal research in low-power VLSI and digital logic design.*
