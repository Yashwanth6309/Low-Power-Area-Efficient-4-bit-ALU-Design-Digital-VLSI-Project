# Low Power Area Efficient ALU with XNOR-Based Full Adder

A 4-bit ALU designed from scratch with a custom XNOR-based full adder that achieves ~50% lower power consumption than conventional designs. Built for embedded and low-power processor applications.

---

## What I Built

A 4-bit ALU that supports 6 operations — ADD, SUB, AND, OR, XOR, XNOR — controlled by a 3-bit select signal (`s0`, `s1`, `s2`). The core innovation is a redesigned full adder that replaces the standard AND/OR/XOR gate topology with XNOR-based logic, reducing switching transitions and cutting dynamic power roughly in half.

---

## The Full Adder Redesign

The standard full adder has high switching activity because XOR and AND gates toggle frequently even for small input changes. I redesigned the full adder using XNOR as the primary gate, which naturally produces fewer output transitions for typical data patterns — when inputs are equal, XNOR outputs 1 without switching.

This change at the full adder cell level propagated power savings across all 4 bits of the ALU, since both addition and subtraction run through the same adder chain.

**Power reduction achieved: ~50% vs conventional full adder design.**

---

## Control Signal Design

I designed a 3-bit control interface to select operations without adding extra hardware for each function:

| s2 | s1 | s0 | Operation |
|----|----|----|-----------|
|  0 |  0 |  0 | ADD       |
|  0 |  0 |  1 | SUB       |
|  0 |  1 |  0 | AND       |
|  0 |  1 |  1 | OR        |
|  1 |  0 |  0 | XOR       |
|  1 |  0 |  1 | XNOR      |

For **subtraction**, I reused the adder by inverting B and forcing carry-in to 1 (two's complement), so no separate subtractor hardware was needed — keeping area minimal.

For **logic operations** (AND, OR, XOR, XNOR), the control signals bypass the adder and route directly to the output mux.

---

## Architecture

```
 A[3:0] ──────────────────────────────────────┐
                                              ▼
                                    ┌──────────────────┐
 s2, s1, s0 ──► Operation Select ──►   Output Mux      ──► Result[3:0]
                        │           └──────────────────┘
                        │                    ▲
                 ┌──────▼──────┐             │
 B[3:0] ──►     │  B Inverter │    ┌─────────────────────┐
          (SUB) └──────┬──────┘    │  4-bit Ripple Carry  │
                       └──────────►│  (XNOR Full Adders)  │──► Carry Out
 Cin (forced 1 for SUB) ──────────►└─────────────────────┘
```

4 XNOR-based full adder cells chained in ripple-carry configuration. Each cell shares the same optimized gate structure — no special-casing per bit.

---

## Verification

Tested all 256 input combinations (16 possible values for A × 16 for B) across all 6 operations in simulation. Every output was checked against the expected result to confirm correctness before measuring power and area.

| Metric                    | Result              |
|---------------------------|---------------------|
| Operations verified       | 6 / 6               |
| Input combinations tested | 256                 |
| Power vs standard design  | ~50% lower          |
| Area                      | Reduced (no separate subtractor) |
| Speed                     | No degradation      |

---

## Project Structure

```
low-power-alu/
├── src/
│   ├── full_adder.v        # XNOR-based 1-bit full adder cell
│   ├── alu_4bit.v          # Top-level 4-bit ALU
│   └── control_unit.v      # 3-bit operation selector + output mux
├── testbench/
│   ├── tb_full_adder.v     # Full adder cell tests
│   └── tb_alu_4bit.v       # All 256-combination ALU tests
├── sim/
│   └── waveforms/          # Simulation output dumps
└── docs/
    ├── schematic.png
    └── power_report.txt
```

---

## How to Run

```bash
git clone https://github.com/your-username/low-power-alu.git
cd low-power-alu

# Compile (Icarus Verilog)
iverilog -o alu_sim src/full_adder.v src/alu_4bit.v testbench/tb_alu_4bit.v

# Simulate
vvp alu_sim

# View waveforms
gtkwave dump.vcd
```

---

## Tools

- HDL: Verilog / VHDL *(update to match what you used)*
- Simulator: ModelSim / Icarus Verilog *(update)*
- Waveform viewer: GTKWave / Vivado *(update)*
- Power analysis: *(add tool name if applicable)*

---

## Author

**[Your Name]** — [College], [Department], [Year]  
[LinkedIn](https://linkedin.com/in/your-profile) · [GitHub](https://github.com/your-username)
