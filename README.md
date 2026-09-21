# 4-Bit Universal Shift Register Using Logic Gates

> **Structural gate-level design, simulation, and verification of a 4-bit Universal Shift Register (USR) in Logisim 2.7.1.**

This project implements a **4-bit Universal Shift Register** using basic logic-gate structures and four edge-triggered D flip-flops. The design supports **Hold, Shift Right, Shift Left, and Parallel Load** operations selected by **S1 and S0**.

The project is based directly on the accompanying presentation, **“Design and Simulation of a 4-Bit Universal Shift Register Using Logic Gates”**, and follows its terminology, routing rules, Boolean expression, simulation workflow, examples, troubleshooting notes, and timing discussion.

---

## 1. Project Objectives

- Design a fully functional **4-bit Universal Shift Register (USR)**.
- Avoid high-level pre-built macro components for the control logic.
- Build the steering logic from **AND, OR, and NOT gates**.
- Use exactly **four edge-triggered D flip-flops** as storage elements.
- Use one **global synchronized clock** and a common **Master Reset**.
- Verify the register through interactive structural simulation in Logisim.

### Tool / Design Scope

| Item | Specification |
|---|---|
| Simulation environment | Logisim / Logisim-Evolution layout style |
| Main logic | AND, OR, NOT gates |
| Storage | 4 × edge-triggered D flip-flops |
| Control | S1, S0 |
| Clocking | Single global clock |
| Reset | Common Master Reset |
| Register width | 4 bits |

---

## 2. What Is a Universal Shift Register?

A **Universal Shift Register (USR)** is a bidirectional sequential circuit that can store and manipulate binary data. It combines:

- serial shifting,
- parallel loading,
- serial-in/parallel-out behavior,
- parallel-in/serial-out behavior.

The selected operation changes on the clock edge according to **S1, S0**.

### Operating Modes

| S1 | S0 | Mode | Function |
|---:|---:|---|---|
| 0 | 0 | **Hold** | Keep the current contents |
| 0 | 1 | **Shift Right** | Shift bits from Q3 toward Q0 |
| 1 | 0 | **Shift Left** | Shift bits from Q0 toward Q3 |
| 1 | 1 | **Parallel Load** | Load I3–I0 simultaneously |

---

## 3. System Truth Table & Routing Rules

| S1 | S0 | Operation | Data Steering Path |
|---:|---:|---|---|
| 0 | 0 | Hold / No Change | Qn → Dn |
| 0 | 1 | Shift Right | Qn+1 → Dn; Serial input enters Stage 3 |
| 1 | 0 | Shift Left | Qn−1 → Dn; Serial input enters Stage 0 |
| 1 | 1 | Parallel Load | In → Dn |

---

## 4. Gate-Level 4×1 Multiplexer

Each register stage uses a custom **`MUX_4x1`** sub-circuit. It selects one of four possible data sources for the D input.

### Construction

The multiplexer is built from:

- **2 NOT gates** — generate S1' and S0'
- **4 three-input AND gates** — decode Y0, Y1, Y2, Y3
- **1 four-input OR gate** — combines the four decoded paths

### Boolean Expression

```text
Out = (Y0 · S1' · S0')
    + (Y1 · S1' · S0)
    + (Y2 · S1 · S0')
    + (Y3 · S1 · S0)
```

This gate-level implementation is the core steering mechanism of the register.

---

## 5. Inter-Stage Signal Routing

Each of the four MUX stages receives four possible inputs.

| Stage | Y0 — Hold | Y1 — Shift Right | Y2 — Shift Left | Y3 — Parallel |
|---|---|---|---|---|
| **Stage 3 (MSB)** | Q3 | Serial_In_Right | Q2 | I3 |
| **Stage 2** | Q2 | Q3 | Q1 | I2 |
| **Stage 1** | Q1 | Q2 | Q0 | I1 |
| **Stage 0 (LSB)** | Q0 | Q1 | Serial_In_Left | I0 |

The selected MUX output connects directly to the **D input** of its corresponding flip-flop.

---

## 6. Logisim Assembly

### Main Circuit

Instantiate:

- **4 × `MUX_4x1`**
- **4 × edge-triggered D flip-flops**

Connect every MUX output to the corresponding D input.

### Control Routing

- Connect **S1** to all four MUXes.
- Connect **S0** to all four MUXes.
- Connect all flip-flop clock inputs to one **global CLK**.
- Distribute the common **Master Reset** as required.

### Interactive Simulation

Use Logisim's **Poke Tool (hand icon)** to change input values.

For clock evaluation, the presentation specifies:

```text
Ctrl + T
Simulate → Tick Half Cycle
```

---

## 7. Example: Parallel-In to Serial-Out (PISO)

### Load `1011`

Set:

```text
S1 S0 = 11
I3 I2 I1 I0 = 1 0 1 1
```

After one clock cycle:

```text
Q3 Q2 Q1 Q0 = 1011
```

### Shift Right

Change to:

```text
S1 S0 = 01
Serial_In_Right = 0
```

Using **Q0** as the serial output tap:

| Clock | Register State | Q0 Output |
|---:|---|---:|
| Initial | 1011 | — |
| 1 | 0101 | 1 |
| 2 | 0010 | 1 |
| 3 | 0001 | 0 |

The presented sequence is:

```text
1, 1, 0, 1
```

---

## 8. Example: Arithmetic Bit Shifts

### Shift Left — Multiplication by 2

Initial:

```text
Q = 0011₂ = 3
```

Set:

```text
S1 S0 = 10
Serial_In_Left = 0
```

After one clock:

```text
Q = 0110₂ = 6
```

Verification:

```text
3 × 2 = 6
```

### Shift Right — Integer Division by 2

Initial:

```text
Q = 1100₂ = 12
```

Set:

```text
S1 S0 = 01
Serial_In_Right = 0
```

After one clock:

```text
Q = 0110₂ = 6
```

Verification:

```text
12 ÷ 2 = 6
```

For unsigned binary values, right shifting truncates any remainder.

---

## 9. Troubleshooting

### Blue Wires / Oscillation

Check that:

- **S1 and S0 are not floating**.
- The Y0 feedback path is connected correctly.
- No illegal feedback/race path has been introduced.

### Red Wires / Short Circuits

Make sure:

- two gate outputs are **never tied directly together**,
- wires are connected only at intended junctions.

### Output Changes Without a Clock Tick

Check the asynchronous reset configuration and ensure the reset is at the required logic level during normal operation.

---

## 10. Timing & Performance

The presentation highlights three important timing concepts:

### Setup Time — `Tsu`

MUX input data must be stable **before the active clock edge**.

### Hold Time — `Th`

Data must remain stable for the required interval **after the clock transition**.

### Propagation Delay — `Tpd`

Delay accumulates through the:

```text
NOT → AND → OR
```

network. The total gate delay limits the maximum usable clock frequency:

```text
Fmax
```

---

## 11. Repository Contents

A typical repository layout for this project:

```text
.
├── README.md
├── Universal shift register.pptx
├── 4bit_universal_shift_register_logisim_2_7_1.circ
├── 4bit_universal_shift_register_version_B_2_7_1.circ
├── 4bit_universal_shift_register_version_C_2_7_1.circ
└── 4bit_universal_shift_register_version_D_2_7_1.circ
```

The `.circ` files are intended for **Logisim 2.7.1**.

---

## 12. Presentation-to-Project Map

This README condenses the supplied presentation slide by slide:

| Slide | Topic | Covered Here |
|---:|---|---|
| 1 | Project title & scope | Introduction |
| 2 | Objectives & technical constraints | Section 1 |
| 3 | USR theory & operating modes | Section 2 |
| 4 | Truth table & routing rules | Section 3 |
| 5 | Custom 4×1 MUX & SOP equation | Section 4 |
| 6 | Inter-stage routing grid | Section 5 |
| 7 | Logisim setup & simulation workflow | Section 6 |
| 8 | PISO example with `1011` | Section 7 |
| 9 | Arithmetic shift examples | Section 8 |
| 10 | Troubleshooting & timing limits | Sections 9–10 |
| 11 | No technical content shown | — |

---

## 13. Core Takeaway

The entire register can be understood as four repeated stages:

```text
     ┌───────────────┐
Y0 ─▶│               │
Y1 ─▶│   4×1 MUX     ├──▶ D ──▶ D-FF ──▶ Q
Y2 ─▶│  (Gate-Level) │
Y3 ─▶│               │
     └───────────────┘
          ▲     ▲
          S1    S0
```

Four such stages operate together under one clock, giving the system its four universal modes:

```text
00 → HOLD
01 → SHIFT RIGHT
10 → SHIFT LEFT
11 → PARALLEL LOAD
```

**Result:** a compact, structural, gate-level 4-bit Universal Shift Register that can be analyzed, simulated, and verified in Logisim.

