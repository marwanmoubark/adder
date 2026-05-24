# adder
# 65 nm CMOS Perceptron — Digital IC Design Project

**Author:** Marwan Mohamed Mobarak Soliman  
**Technology:** 65 nm CMOS  
**Tool:** Cadence Virtuoso (ADE, DRC/LVS, PEX)

---

## Overview

This project presents the full custom design of a **4-bit × 2-bit perceptron datapath** implemented in a 65 nm CMOS process. Every cell is hand-sized at the transistor level — no standard-cell library is used. The perceptron computes the multiply-accumulate (MAC) operation:

```
Y = X1·W1 + X2·W2
```

The design covers six engineering objectives: architecture, transistor sizing, schematic simulation, physical layout, DRC/LVS verification, and post-layout PEX characterization.

---

## Architecture

```
 X1[3:0] ──┐                     ┌──────────────┐
            ├── Multiplier 1 ──▶ │              │
 W1[1:0] ──┘                     │  4-bit CLA   │──▶ Y[3:0], Cout
                                  │              │
 X2[3:0] ──┐                     │   (Adder)    │
            ├── Multiplier 2 ──▶ │              │
 W2[1:0] ──┘                     └──────────────┘
```

### Blocks

| Block | Description |
|---|---|
| **Multiplier** | Transmission-gate 2:1 MUX array; computes 4-bit partial products X·W |
| **Adder (CLA)** | 4-bit Carry Lookahead Adder with custom G/P, carry, and sum stages |
| **Perceptron** | Top-level integration of two Multipliers feeding one CLA |

---

## Common Design Parameters

| Parameter | Symbol | Value |
|---|---|---|
| Process technology | — | 65 nm CMOS |
| Channel length (all devices) | L | 65 nm |
| Basic Inverter (PMOS / NMOS) | Wp / Wn | 300 nm / 120 nm |
| Inverter Driver (PMOS / NMOS) | Wp / Wn | 1.2 µm / 500 nm |
| Transmission Gate (PMOS / NMOS) | Wp / Wn | 390 nm / 200 nm |

---

## Block Details

### Multiplier Block

- Built from an array of custom **transmission-gate (TG) 2:1 MUX** cells.
- Each MUX contains two TGs and two inverter drivers, routed on Metal 1.
- PMOS/NMOS width ratio of **2.5:1** throughout, compensating for µn/µp mobility disparity.
- TGs are kept compact (390 nm / 200 nm) to minimize gate (Cgg) and junction (Cdd) parasitic capacitance.

### Adder Block — Carry Lookahead Architecture

The CLA eliminates the linear delay of ripple-carry by computing all carries in parallel:

- **Generate:** `Gi = Ai · Bi` — implemented as NAND + inverter (W = 240 nm core)
- **Propagate:** `Pi = Ai + Bi` — implemented as NOR + inverter (W = 240 nm core)
- **Carry Network (C1–C5):** Progressive stack sizing applied:
  - Carry 1–2: shallow stacks, widths 120–720 nm
  - Carry 3–5: deep stacks (up to 5 transistors), widths up to 1.2 µm
- **Sum Block:** 2:1 PMOS/NMOS ratio (480 nm / 240 nm core), output buffer scaled to 240 nm / 120 nm

### Transistor Sizing Principles

1. **Mobility compensation** — PMOS widened by ~2.5× relative to NMOS to equalize drive strength.
2. **Series-stack scaling** — N transistors in series are each sized N× wider than a unit reference to maintain equivalent drive strength (progressive stack sizing).

---

## Physical Implementation

- **Strategy:** Hand-drawn layout with a standardized cell-height architecture. VDD/VSS rails routed horizontally on higher metal layers, enabling seamless cell tiling.
- **Local routing:** Metal 1 for intra-cell connections; higher metals for global data/weight buses.
- **DRC:** Zero violations against the 65 nm foundry design rules.
- **LVS:** Perfect match — transistor count, sizes, net connectivity, and port assignments all confirmed.

---

## Parasitic Extraction and Post-Layout Results

After DRC/LVS sign-off, **Parasitic Extraction (PEX)** generated an annotated netlist with distributed RC networks for every routed signal. Post-layout transient simulations show:

| Criterion | Pre-Layout | Post-Layout (PEX) |
|---|---|---|
| Functional correctness | ✓ | ✓ |
| Interconnect parasitics | Not modeled | Distributed RC |
| Junction / well capacitance | Device models only | Extracted from layout |
| Rise/fall shape | Sharp, near-ideal | Slightly softened |
| Propagation delay | Baseline | Small, predictable increase |
| Verification status | ADE simulation passes | DRC clean, LVS match, PEX OK |

No functional regression was observed. Timing degradation is bounded and predictable.

---

## Performance Summary

### Delay

| Stage | Hand Analysis | Simulation | Error |
|---|---|---|---|
| Multiplier | 31.3 ps | 26.6 ps | 15% |
| Adder | 124.62 ps | 146.5 ps | 14.6% |
| **Total** | **156 ps** | **173.1 ps** | **9.5%** |

### Power (at 1 GHz, VDD = 1.2 V)

| Activity Factor (α) | Hand Analysis | Simulation | Error |
|---|---|---|---|
| α = 1.0 (worst case) | 288.03 µW | 90.89 µW | 216% |
| α = 0.5 (typical data) | 144.015 µW | 90.89 µW | 58.5% |
| α = 0.25 (random data) | 72 µW | 90.89 µW | 20% |

> Power discrepancy at α = 1 is expected — real circuits rarely switch every node every cycle. At α = 0.25 (empirically realistic for CMOS logic), hand analysis and simulation agree within 20%.

---

## Technology Device Parameters

| Parameter | NMOS | PMOS |
|---|---|---|
| Drain Capacitance | 211.55 aF | 240.44 aF |
| Gate Capacitance | 105.95 aF | 113.7 aF |
| Equivalent Resistance | 6490 Ω | 14012.4 Ω |

---

## Project Structure

```
/
├── Multiplier/
│   ├── schematic/       # TG-MUX array, inverter, inverter driver, TG cells
│   └── layout/          # Hand-drawn cell and top-level layouts
├── Adder/
│   ├── schematic/       # G/P blocks, Carry 1–5 blocks, Sum block
│   └── layout/
├── Perceptron/
│   ├── schematic/       # Top-level MAC integration
│   └── layout/
├── verification/
│   ├── drc/             # DRC results (0 violations)
│   ├── lvs/             # LVS results (perfect match)
│   └── pex/             # Extracted netlist + post-layout simulations
└── report/
    └── main.pdf         # Full project report
```

---

## References

- Weste & Harris — *CMOS VLSI Design: A Circuits and Systems Perspective*
- Cadence Virtuoso ADE / Assura DRC/LVS / PEX documentation
- Foundry 65 nm PDK design rule manual
