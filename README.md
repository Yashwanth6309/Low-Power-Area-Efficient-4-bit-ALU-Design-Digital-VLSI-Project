# Low Power Area-Efficient 4-bit ALU with XNOR-Based Full Adder

![Domain](https://img.shields.io/badge/Domain-VLSI%20%2F%20Digital%20Logic-7C3AED)
![Language](https://img.shields.io/badge/Tool-Hardware%20Simulation-0D9488)
![Status](https://img.shields.io/badge/Status-Completed-16A34A)
![Duration](https://img.shields.io/badge/Duration-Jan%202023%20–%20Jul%202023-EA580C)

---

## What Is This Project?

This project is the design and verification of a **4-bit Arithmetic Logic Unit (ALU)** — the core component inside every processor that performs math and logic operations.

The goal was not just to build a working ALU, but to make it **low power** and **area efficient** — two critical requirements in embedded systems, IoT devices, and battery-powered hardware where every milliwatt and every gate counts.

The key innovation: replacing the traditional full adder design (which wastes energy through unnecessary gate switching) with an **XNOR-based full adder** that achieves the same results with ~50% less power consumption.

---

## The Problem Being Solved

In digital hardware, every gate that switches state (0→1 or 1→0) consumes power. A traditional full adder uses a chain of AND → OR → XOR gates. This chain has high **switching activity** — many gates toggle for every computation, even when they don't need to.

In embedded systems, this is a real problem:
- Battery-powered devices drain faster
- Heat increases, requiring more cooling
- Chip area grows with more gates

**This project solves it by redesigning the full adder at the gate level** using XNOR-based logic that reduces transitions and cuts power by approximately 50%.

---

## System Architecture

```
                    ┌──────────────────────────────────────┐
                    │           3-bit Control Word          │
                    │         s2   s1   s0                  │
                    └──────────┬───┬───┬───────────────────┘
                               │   │   │
         ┌─────────────────────▼───▼───▼─────────────────────┐
         │                                                     │
4-bit A ─►                  4-bit ALU                         ├─► 4-bit Result
         │                                                     │
4-bit B ─►   [ XNOR-Based Full Adder Core × 4 ]              ├─► Carry Out
         │   [ Operation Control Logic        ]               │
         │                                                     │
         └─────────────────────────────────────────────────────┘
```

The ALU takes:
- **Input A** — 4-bit number (0 to 15)
- **Input B** — 4-bit number (0 to 15)
- **Control signals s0, s1, s2** — 3 bits that select which operation to perform

And produces:
- **4-bit Result** — the output of the selected operation
- **Carry Out** — overflow bit for addition

---

## Supported Operations

| s2 | s1 | s0 | Operation | Description |
|:--:|:--:|:--:|-----------|-------------|
| 0  | 0  | 0  | **ADD**   | A + B (binary addition) |
| 0  | 0  | 1  | **SUB**   | A − B (two's complement subtraction) |
| 0  | 1  | 0  | **AND**   | Bitwise AND |
| 0  | 1  | 1  | **OR**    | Bitwise OR |
| 1  | 0  | 0  | **XOR**   | Bitwise XOR |
| 1  | 0  | 1  | **XNOR**  | Bitwise XNOR |

---

## Core Design: XNOR-Based Full Adder

### What is a Full Adder?

A full adder adds three 1-bit inputs (A, B, Carry-In) and produces a Sum and a Carry-Out. Four of these are chained together to build a 4-bit adder — the core of the ADD and SUB operations.

### Traditional Design vs This Design

**Traditional full adder:**
```
Sum   = A XOR B XOR Cin
Carry = (A AND B) OR (B AND Cin) OR (A AND Cin)

Gates used: AND × 3, OR × 2, XOR × 2  → 7 gates, high switching activity
```

**XNOR-based full adder (this project):**
```
P     = A XNOR B           ← intermediate signal (reused)
Sum   = P XNOR Cin
Carry = MUX(A, Cin, P)     ← multiplexer selects based on P

Gates used: XNOR × 2, MUX × 1  → 3 gates, low switching activity
```

The XNOR gate has a natural property: it produces a HIGH output when both inputs are equal (both 0 or both 1). This means it switches far less frequently than XOR under typical data patterns, which is where the power saving comes from.

### Power Reduction Mechanism

| Factor | Traditional | XNOR-Based |
|--------|-------------|------------|
| Gate count per adder | 7 | 3 |
| Switching transitions (typical) | High | ~50% lower |
| Power consumption | Baseline | ~50% reduction |
| Logic depth (speed) | Same | Same |

---

## Subtraction: Two's Complement

Instead of building a separate subtraction circuit (which would double the hardware), subtraction reuses the same adder with a clever trick:

```
A − B  =  A + (~B) + 1
```

Steps:
1. Invert all bits of B (bitwise NOT)
2. Add 1 (by setting Carry-In = 1)
3. Add A normally

This is **two's complement arithmetic** — the standard technique used in all modern processors. The result is correct signed subtraction using the exact same adder hardware, controlled entirely by the `s0` signal.

---

## Control Logic

Three control bits (s0, s1, s2) route inputs through a multiplexer to select the operation:

```
s2=0, s1=0 → arithmetic mode  →  s0=0: ADD,  s0=1: SUB
s2=0, s1=1 → logic mode A     →  s0=0: AND,  s0=1: OR
s2=1, s1=0 → logic mode B     →  s0=0: XOR,  s0=1: XNOR
```

This is implemented with minimal gate overhead — the control signals directly steer MUXes at the input stage rather than switching entire functional units on and off.

---

## Verification Methodology

Every operation was verified exhaustively. With 4-bit inputs:
- Input A: 16 possible values (0000 to 1111)
- Input B: 16 possible values (0000 to 1111)
- Combinations per operation: **16 × 16 = 256**
- Total test cases across 6 operations: **256 × 6 = 1,536**

### What Was Checked

For each test case:
- **Functional correctness** — does the output match the expected result?
- **Carry-out** — is the overflow bit correct for ADD/SUB?
- **Timing** — does the output settle before the next clock edge?
- **Power** — switching activity measured and compared vs baseline

### Simulation Waveform (Conceptual)

```
Time →    T0    T1    T2    T3    T4    T5
─────────────────────────────────────────────────
A         0011  0110  1010  1111  0101  1100
B         0101  0011  0110  0001  1010  0011
Control   ADD   ADD   SUB   AND   OR    XOR
─────────────────────────────────────────────────
Result    1000  1001  0100  0001  1111  1111
Carry     0     0     0     0     0     0
─────────────────────────────────────────────────
```

All 1,536 test cases passed with no logic errors.

---

## Results Summary

| Metric | Result |
|--------|--------|
| Power reduction | ~50% vs traditional full adder |
| Operations implemented | 6 (ADD, SUB, AND, OR, XOR, XNOR) |
| Input width | 4 bits (expandable) |
| Control overhead | 3 bits (8 possible operation slots) |
| Subtraction method | Two's complement (no extra hardware) |
| Test cases verified | 1,536 (256 per operation × 6) |
| Logic errors found | 0 |

---

## Design Decisions and Trade-offs

### Why XNOR instead of XOR?
XOR switches on every input change where the bits differ. XNOR naturally stays HIGH when inputs are equal — which is statistically more common in typical data patterns. This means fewer transitions and less dynamic power.

### Why two's complement for subtraction?
A dedicated subtractor would require ~7 extra gates per bit stage (28 gates for 4 bits). Two's complement reuses the existing adder with just a signal inversion and Carry-In = 1. Same result, zero extra area.

### Why 3-bit control instead of 2-bit?
2 bits supports only 4 operations. 3 bits gives 8 slots — the 6 implemented operations plus 2 reserved for future expansion (e.g., NAND, NOR) without redesigning the control logic.

### Why 4-bit width?
4 bits is the smallest practical word size for demonstrating all ALU behaviours including carry propagation and overflow. The design is directly scalable to 8, 16, or 32 bits by cascading additional full adder stages.

---

## Skills and Concepts Demonstrated

- **Digital logic design** — gate-level schematic design and optimization
- **Low-power design techniques** — switching activity analysis, XNOR-based logic
- **Arithmetic circuits** — full adder design, carry propagation, two's complement
- **Control architecture** — multiplexer-based operation selection
- **Hardware verification** — exhaustive test case generation and simulation
- **Embedded systems thinking** — area and power constraints as first-class design goals

---

## Project Timeline

| Phase | Period | Work Done |
|-------|--------|-----------|
| Research & design | Jan – Feb 2023 | Studied traditional full adder designs, identified XNOR optimization |
| Core implementation | Mar – Apr 2023 | Designed XNOR adder, built ALU control logic |
| Simulation & testing | May – Jun 2023 | Generated all 1,536 test cases, verified outputs |
| Optimization & wrap-up | Jul 2023 | Power/area measurement, documentation |

---

## Author

**[Your Name]**  
B.E. / B.Tech in Electronics / VLSI / Computer Engineering  
[LinkedIn](https://linkedin.com/in/yourprofile) · [Email](mailto:your@email.com)

---

*This project was completed as part of academic research in low-power VLSI and digital circuit design. Jan 2023 – Jul 2023.*
