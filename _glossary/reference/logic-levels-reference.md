---
title: Logic Levels — Calculations Reference
category: reference
slug: logic-levels-reference
description: Full derivation of every formula used in the Logic Levels calculator — CMOS voltage thresholds, FO4 delay, critical path depth, logical effort, AOI/OAI gate types, gate count, and power density, with links to primary sources.
---

# Logic Levels — Calculations Reference

This page documents every formula used in the [Logic Levels calculator](/tools/logic-levels/).
Each section covers the physical meaning, the formula, the assumptions made, and the primary sources.

---

## 1. CMOS Voltage Thresholds

A CMOS gate defines four voltage thresholds that separate valid logic states from the undefined *forbidden zone*.

| Symbol | Name | Formula | Meaning |
|--------|------|---------|---------|
| **VOH** | Output HIGH voltage (min) | `0.9 × VDD` | Lowest voltage a gate guarantees as a valid HIGH output |
| **VOL** | Output LOW voltage (max)  | `0.1 × VDD` | Highest voltage a gate guarantees as a valid LOW output |
| **VIH** | Input HIGH voltage (min)  | `0.7 × VDD` | Lowest voltage a gate will reliably interpret as HIGH |
| **VIL** | Input LOW voltage (max)   | `0.3 × VDD` | Highest voltage a gate will reliably interpret as LOW |

### Noise Margins

The gap between output guarantee and input requirement is the *noise margin* — the amount of signal degradation the interconnect can introduce without causing a logic error.

```
NMH  =  VOH − VIH  =  0.9·VDD − 0.7·VDD  =  0.2·VDD
NML  =  VIL − VOL  =  0.3·VDD − 0.1·VDD  =  0.2·VDD
```

The 0.9 / 0.7 / 0.3 / 0.1 ratios are standard CMOS approximations derived from the
symmetric NMOS/PMOS switching point at `VDD/2`. Actual silicon values vary by
process corner (slow/typical/fast) and temperature, but these fractions are a
reliable first-order model for all planar and FinFET nodes.

> **Why 0.7 × VDD for VIH?**
> The CMOS transfer curve has maximum gain around `VDD/2`. Inputs above `0.7·VDD`
> are deep in the NMOS-dominant region where the output is solidly LOW, so
> `0.7·VDD` is a conservative but practical threshold.

**Sources**

- Weste & Harris, *CMOS VLSI Design* 4th ed., §1.5 — Static CMOS Voltage Transfer Characteristics
- Rabaey, Chandrakasan & Nikolic, *Digital Integrated Circuits* 2nd ed., §5.2
- IEEE Std 91A-1991 — Logic Symbol Standards (threshold definitions)

---

## 2. Gate Delay and the FO4 Metric

### Intrinsic gate delay (`t_gate`)

The intrinsic (unloaded) gate delay is the propagation delay of a minimum-sized inverter driving no external load. It scales roughly as:

```
t_gate  ≈  (C_gate × VDD) / I_on
```

where `C_gate` is the gate capacitance and `I_on` is the on-state drive current.
As nodes shrink, `C_gate` falls faster than `I_on` improves, so `t_gate` shrinks
— but with diminishing returns below 7 nm.

The values in the calculator are empirical consensus figures from published IEDM
and ISSCC papers, shown below alongside their sources.

### Fanout-of-4 delay (`t_FO4`)

A single unloaded inverter delay (`t_gate`) is not a useful design metric because
real gates always drive something. The **fanout-of-4 (FO4) delay** standardises
the load: it is the delay of one inverter driving four identical inverters.

```
         ┌───┐
    ─────┤ 1 ├──┬──► INV
         └───┘  ├──► INV    ← 4 identical loads
                ├──► INV
                └──► INV

  t_FO4  ≈  4 × t_gate   (empirical, varies ±20% by process)
```

FO4 is the standard timing unit in microarchitecture research because:

1. It is **technology-independent** — "15 FO4 critical path" means the same thing
   on 28 nm and 5 nm even though the absolute picoseconds differ.
2. It captures a **realistic load** — most internal nodes see 2–6 fanout.
3. It correlates well with **pipeline stage depth** as reported in academic literature.

The `t_FO4` values in the calculator use the empirical relation `t_FO4 ≈ 4 × t_gate`,
cross-referenced against published characterisation data.

**Sources**

- Sutherland, Sproull & Harris, *Logical Effort*, Chapter 1 — FO4 as canonical delay unit
- Hrishikesh et al., *"The Optimal Logic Depth Per Pipeline Stage Is 6 to 8 FO4 Inverter Delays"*, ISCA 2002
- Borkar & Chien, *"The Future of Microprocessors"*, CACM 2011

---

## 3. Critical Path Depth

The critical path is the longest timing arc from one flip-flop's output to another's
data input. Its depth in FO4 stages limits the maximum operating frequency.

### Why not use raw gate delays?

`t_gate` is the absolute minimum delay with no load and no wire. Using it gives an
optimistic upper bound that ignores:

| Overhead | Typical fraction of `T_clk` |
|----------|-----------------------------|
| Clock-to-Q delay (FF output) | ~10 % |
| Setup time (FF input) | ~15 % |
| Clock skew + jitter | ~8 % |
| Wire / buffer delay | ~7 % |
| **Total overhead** | **~40 %** |

Only ~60 % of the clock period is available for combinational logic.

### Formula used in the calculator

```
logic_budget  =  T_clk × 0.6
depth (FO4)   =  ⌊ logic_budget / t_FO4 ⌋
```

#### Example — 3 GHz on 7 nm

```
T_clk         =  1000 / 3.0  =  333 ps
logic_budget  =  333 × 0.6   =  200 ps
t_FO4 (7nm)   =  44 ps
depth         =  ⌊ 200 / 44 ⌋  =  4 FO4 stages
```

This matches published microarchitecture data: Apple M-series and AMD Zen cores
running 3–4 GHz on 5–7 nm report 4–8 FO4 stages per pipeline stage.

### Reference values

| Regime | FO4 stages | Example |
|--------|-----------|---------|
| < 8 FO4 | Aggressive, near f_max, hard timing closure | Intel i9 @ 5 GHz / 10 nm |
| 10–16 FO4 | Typical production design | Server CPU @ 2–3 GHz |
| 18–25 FO4 | Relaxed, well below f_max | Low-power microcontroller |
| > 25 FO4 | Extremely conservative | FPGA soft-core emulation |

**Sources**

- Hrishikesh et al., *"The Optimal Logic Depth Per Pipeline Stage Is 6 to 8 FO4 Inverter Delays"*, ISCA 2002 — [ACM DL](https://dl.acm.org/doi/10.1145/545214.545225)
- Harris & Harris, *Digital Design and Computer Architecture* §3.5 — Timing analysis
- Weste & Harris, *CMOS VLSI Design* 4th ed., §4.4 — Logical effort and path delay

---

## 4. Gate Types, Logical Effort, and Critical Path Depth

### What is logical effort?

**Logical effort** is a dimensionless number that captures how hard a gate is to
drive compared to a standard inverter. An inverter has logical effort `g = 1` by
definition. A NAND2 has `g = 4/3` because its series NMOS stack needs wider
transistors to match inverter drive strength, loading its inputs more.

```
delay of gate type X at fanout h  =  g × h × t_p0  +  p
                                      ↑         ↑
                               logical   electrical
                                effort    effort
```

For a practical critical path estimate at fanout-of-4 (FO4), the delay per gate
simplifies to:

```
t_gate_type  =  g × t_gate      (t_gate = intrinsic inverter delay for the node)
```

Gates in the critical path given a logic budget `B`:

```
gates_in_path  =  ⌊ B / (g × t_gate) ⌋
               =  ⌊ (T_clk × 0.6) / (g × t_gate) ⌋
```

### Gate type reference table

| Gate | Function | Logical effort (g) | Transistors | Logic complexity |
|------|----------|-------------------|-------------|-----------------|
| INV    | ¬A                    | 1.00 | 2  | 1 level  |
| NAND2  | ¬(A·B)                | 1.33 | 4  | 1 level  |
| NAND3  | ¬(A·B·C)              | 1.67 | 6  | 1 level  |
| NOR2   | ¬(A+B)                | 1.67 | 4  | 1 level  |
| NOR3   | ¬(A+B+C)              | 2.33 | 6  | 1 level  |
| AOI21  | ¬(A·B + C)            | 1.50 | 6  | 2 levels |
| AOI22  | ¬(A·B + C·D)          | 2.00 | 8  | 2 levels |
| AOI211 | ¬(A·B + C + D)        | 1.67 | 8  | 3 levels |
| OAI21  | ¬((A+B)·C)            | 1.67 | 6  | 2 levels |
| OAI22  | ¬((A+B)·(C+D))        | 2.00 | 8  | 2 levels |

*Logic complexity* = number of AND/OR logic levels computed in a single physical gate stage.

### AOI and OAI gates

**AOI (And-Or-Invert)** and **OAI (Or-And-Invert)** are *complex gates* — they
implement multiple boolean operations in a single CMOS stage by merging the
pull-down network (NMOS) and pull-up network (PMOS) of what would otherwise be
two or three separate gates.

```
── AOI22: ¬(AB + CD) ───────────────────────────────────────
                          VDD
                           │
              ┌────────────┼────────────┐
             [P:A]       [P:C]        ...     ← PMOS pull-up
              │            │
             [P:B]       [P:D]             (parallel-series PMOS)
              └────────────┘
                           │ out
              ┌────────────┐
             [N:A]       [N:C]             ← NMOS pull-down
              │            │
             [N:B]       [N:D]             (series-parallel NMOS)
              └────────────┘
                           │
                          GND

  Equivalent circuit: inverter + NAND2 + NAND2 + OR  →  1 gate stage
```

**Why use complex gates?**

| Plain gates (3 stages) | Complex gate (1 stage) |
|------------------------|------------------------|
| AND2 → AND2 → NOR2     | AOI22                  |
| 3 × propagation delays | 1 × (2.0 × t_gate)     |
| 3 × VDD transitions    | 1 × VDD transition      |

A single AOI22 stage is ~33 % faster than the equivalent three-gate chain because
it avoids two intermediate node transitions and the glitch power they generate.

### Gates vs logic levels

The calculator distinguishes two quantities:

- **Gates in critical path** — the number of physical gate *instances* from
  flip-flop output to flip-flop input. Fewer gates = faster, simpler netlist.
- **Logic levels** = gates × complexity — the number of AND/OR *operations*
  those gates collectively compute. This is the true measure of algorithmic depth.

```
Example — 3 GHz / 7 nm, logic budget = 200 ps, t_gate = 11 ps:

  NAND2  (g=1.33):  gates = ⌊200 / (1.33×11)⌋ = 13   levels = 13×1 = 13
  AOI22  (g=2.00):  gates = ⌊200 / (2.00×11)⌋ = 9    levels = 9×2  = 18

  AOI22 uses fewer gate stages but computes more logic per stage.
  It is preferred when boolean functions map naturally to sum-of-products form.
```

### Why NAND2 is the normalisation baseline

NAND2 is chosen as the reference gate because:

1. It has the **lowest logical effort** among useful two-input gates (inverter is
   lower but computes no new function).
2. Standard cell libraries express area and power in **NAND2-equivalent** units.
3. Most synthesis tools map combinational logic to NAND/NOR trees by default.

**Sources**

- Sutherland, Sproull & Harris, *Logical Effort: Designing Fast CMOS Circuits*,
  Morgan Kaufmann, 1999 — the definitive text on logical effort
- Weste & Harris, *CMOS VLSI Design* 4th ed., §4.3 — Complex gates, §4.4 — Logical effort
- Rabaey et al., *Digital Integrated Circuits* 2nd ed., §6.3 — AOI/OAI gate design
- Sheehan, B., *"Logical Effort Tutorial"*, EE Times, 2003

---

## 5. Gate Count

### Total transistors

```
transistors  =  density (MT/mm²) × 10⁶  ×  die_area (mm²)
```

Transistor density is published in MT/mm² (million transistors per mm²).
Multiplying by 10⁶ converts to transistors/mm², then by die area gives a total count.

### NAND2-equivalent gates

A NAND2 gate requires 4 transistors (2 NMOS in series, 2 PMOS in parallel).
It is the standard normalisation unit for logic density comparisons.

```
gates (NAND2-eq)  =  transistors / 4
```

Real chips mix gate types — some use 6-transistor flip-flops, 8-transistor AOI cells,
etc. The ÷4 approximation gives a useful order-of-magnitude figure.

### Parallel pipeline width

If the total gate count is `G` and the critical path depth is `D` FO4 stages, the
die could in principle support that many independent sequential pipelines:

```
pipelines  ≈  G / D
```

This is a theoretical maximum assuming perfectly balanced, independent workloads
with no routing or power constraints.

### Active gates per cycle and switching rate

```
active_per_cycle  =  G × α
switching_per_sec =  active_per_cycle × f
```

where `α` is the **activity factor** — the fraction of gates that switch (0→1 or 1→0)
on a given clock edge. Typical values:

| Workload | α |
|----------|---|
| Static / mostly idle logic | 0.05 – 0.10 |
| Typical CPU pipeline | 0.20 – 0.35 |
| GPU shader core (heavy ALU) | 0.40 – 0.60 |
| Worst-case (all gates toggle) | 1.0 |

**Sources**

- ITRS 2015 Roadmap Tables, *Process Integration, Devices, and Structures* chapter
- Shalf, *"The future of computing beyond Moore's Law"*, Phil. Trans. R. Soc. A, 2020 — [DOI](https://doi.org/10.1098/rsta.2019.0061)
- TSMC Technology Symposium papers (2016–2023) — density figures per node

---

## 6. Power Density

### Dynamic (switching) power

Each time a gate switches, it charges or discharges its load capacitance `C_load`
through the supply rail, dissipating energy `½ C V²` per transition.

```
P_dyn  =  α · C_load · VDD² · f
```

In density terms (mW/mm²), `C_load` is absorbed into an empirical reference
power density `P_ref` measured at α = 0.3, f = 1 GHz for each node:

```
P_dyn (mW/mm²)  =  P_ref  ×  (α / 0.3)  ×  (f / 1 GHz)
```

The `VDD²` scaling is already baked into `P_ref` per node, so the formula
correctly reflects lower dynamic power at reduced supply voltages without
needing an explicit VDD² term.

### Leakage (static) power

Sub-threshold and gate-oxide leakage produce a constant current regardless of
switching activity. Leakage grows with transistor count and worsens at elevated
temperature; it does **not** scale with frequency.

```
P_leak (mW/mm²)  ≈  constant per node (temperature-independent approximation)
```

The reference leakage figures in the calculator are empirical estimates derived
from published chip measurements and ITRS projections. They increase with node
scaling because higher transistor density packs more leaky junctions per mm².

### Total power

```
P_total  =  P_dyn + P_leak
```

Real chip power budgets also include:
- **Clock tree power** (~20–30 % of dynamic)
- **Memory array leakage** (SRAM bit-cells)
- **I/O and pad ring power**

These are omitted here; the calculator gives logic-core power density only.

**Sources**

- Weste & Harris, *CMOS VLSI Design* 4th ed., §5.1 — Dynamic and static power
- Rabaey et al., *Digital Integrated Circuits* 2nd ed., §5.5 — Power dissipation
- Borkar, *"Design Challenges of Technology Scaling"*, IEEE Micro, 1999 — [IEEE](https://ieeexplore.ieee.org/document/782838)
- TSMC 7 nm / 5 nm / 3 nm Technology Overview briefs

---

## 7. Process Node Data

The per-node values (VDD, gate delay, FO4, f_max, transistor density, gate pitch)
are consensus figures compiled from the following primary sources.

| Source | What it provides |
|--------|-----------------|
| IEDM proceedings (annual) | Gate delay, density, VDD per node at introduction |
| ISSCC proceedings (annual) | f_max and power figures from real chip tape-outs |
| ITRS / IRDS Roadmap | Projected scaling trajectory for pitch, VDD, density |
| IEEE Spectrum, *"Transistor count and Moore's Law"* | Historical density cross-check |
| WikiChip node pages | Synthesised per-node summaries with citations |

Key caveats:

- **"N nm" is a marketing label**, not a physical gate length. Since ~28 nm, node
  names have decoupled from actual gate lengths. TSMC 7 nm and Intel 10 nm have
  similar real dimensions.
- **f_max is process-limited**, not design-limited. A chip running at 3 GHz on 7 nm
  is not at f_max; it trades speed for power/area efficiency.
- Density figures count **total transistors**, including SRAM cells (which are
  denser than logic cells). Logic density alone is typically 40–60 % of the
  published headline density.

**Primary references**

- Waldrop, M. M., *"The chips are down for Moore's law"*, Nature 530, 144–147 (2016) — [DOI](https://doi.org/10.1038/530144a)
- Shalf, J., *"The future of computing beyond Moore's Law"*, Phil. Trans. R. Soc. A (2020) — [DOI](https://doi.org/10.1098/rsta.2019.0061)
- IRDS 2022 Roadmap, *More Moore* chapter — [irds.ieee.org](https://irds.ieee.org/editions/2022)
- WikiChip process node comparisons — [en.wikichip.org/wiki/technology_node](https://en.wikichip.org/wiki/technology_node)

---

## 8. Quick Formula Card

```
── Voltage thresholds (CMOS) ────────────────────────────────
  VOH = 0.90 · VDD       VOL = 0.10 · VDD
  VIH = 0.70 · VDD       VIL = 0.30 · VDD
  NMH = VOH − VIH        NML = VIL − VOL  (both = 0.20 · VDD)

── Timing ───────────────────────────────────────────────────
  T_clk (ps)      = 1000 / f (GHz)
  logic_budget    = T_clk × 0.6
  depth (FO4)     = ⌊ logic_budget / t_FO4 ⌋
  t_FO4           ≈ 4 × t_gate

── Gate type critical path ───────────────────────────────────
  logic_budget     = T_clk × 0.6
  t_gate_type      = g × t_gate
  gates_in_path    = ⌊ logic_budget / t_gate_type ⌋
  logic_levels     = gates_in_path × complexity

  Logical effort (g) by gate type:
    INV    1.00   NAND2  1.33   NAND3  1.67
    NOR2   1.67   NOR3   2.33
    AOI21  1.50   AOI22  2.00   AOI211 1.67
    OAI21  1.67   OAI22  2.00

  Logic complexity (AND/OR levels per stage):
    INV/NAND/NOR  → 1     AOI21/OAI21 → 2
    AOI22/OAI22   → 2     AOI211      → 3

── Gate count ───────────────────────────────────────────────
  transistors     = density (MT/mm²) × 10⁶ × area (mm²)
  gates (NAND2)   = transistors / 4
  pipelines       = gates / depth
  active/cycle    = gates × α
  switch/sec      = active/cycle × f

── Power ────────────────────────────────────────────────────
  P_dyn  = P_ref × (α / 0.3) × (f / 1 GHz)   [mW/mm²]
  P_leak = constant per node                    [mW/mm²]
  P_tot  = P_dyn + P_leak
```
