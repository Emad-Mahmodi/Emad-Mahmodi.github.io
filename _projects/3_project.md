---
layout: page
title: PISAD ECU Full Simulator
description: >
  <i class="fa-solid fa-microchip"></i>
  A high-fidelity PowerPC (PPC) emulator for full ECU firmware analysis,
  automotive security research, and Seed/Key reverse engineering.
img: assets/img/reverse.png
importance: 1
category: work
---

<style>
  /* ── Icon color palette ─────────────────────────────────── */
  .icon-hardware  { color: #5dade2; }
  .icon-security  { color: #e74c3c; }
  .icon-analysis  { color: #2ecc71; }
  .icon-workflow  { color: #f39c12; }

  /* ── Section divider ────────────────────────────────────── */
  .section-divider {
    border: none;
    border-top: 2px solid #2e4057;
    margin: 2.5rem 0;
  }

  /* ── Feature grid cards ─────────────────────────────────── */
  .feature-card {
    background: #1a2535;
    border-left: 4px solid #5dade2;
    border-radius: 6px;
    padding: 1rem 1.25rem;
    margin-bottom: 1rem;
  }
  .feature-card h5 { margin-top: 0; color: #5dade2; }
  .feature-card p  { color: #d0dce8 !important; font-size: 0.95rem; line-height: 1.6; }
  .feature-card code { background: #0d1b2a; color: #7ec8e3; padding: 1px 5px; border-radius: 3px; }

  /* ── Badge pill ─────────────────────────────────────────── */
  .badge-tech {
    display: inline-block;
    background: #2e4057;
    color: #a8d8ea;
    border-radius: 999px;
    padding: 2px 10px;
    font-size: 0.82rem;
    margin: 2px 2px;
  }

  /* ── Workflow step number ───────────────────────────────── */
  .step-num {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    width: 2rem;
    height: 2rem;
    background: #f39c12;
    color: #fff;
    border-radius: 50%;
    font-weight: 700;
    margin-right: .6rem;
    flex-shrink: 0;
  }
  .step-row { display: flex; align-items: flex-start; margin-bottom: .8rem; }
</style>

---

## <i class="fa-solid fa-info-circle icon-hardware"></i> Introduction

This project is a **dedicated simulation environment for PISAD-based ECUs** running on the **PowerPC (PPC)** architecture — the processor family found in a wide range of automotive engine control units including ME17, Changan, and Lamari variants.

Unlike generic emulators, this tool is purpose-built for **automotive firmware analysis**. It allows researchers to:

- Load a **full ECU binary dump** and run it in an isolated virtual environment.
- Interact with the simulated processor at a hardware-abstracted level — including registers, memory, and peripheral I/O.
- Inject custom **analogue signals**, **CAN bus messages**, and **diagnostic commands** (UDS / KWP2000) without touching physical hardware.

<div class="alert alert-info">
  <strong><i class="fa-solid fa-lightbulb"></i> The Core Challenge:</strong>
  Reverse-engineering automotive ECUs typically means bypassing hardware security under real operating conditions — a risky and time-consuming process.
  This simulator provides a safe <strong>sandbox</strong> where every byte of memory and every register transition is observable and controllable, enabling researchers to locate and analyse <strong>authentication algorithms (SID 0x27)</strong> without ever touching the physical ECU.
</div>

<span class="badge-tech">PowerPC e200 / MPC5xx</span>
<span class="badge-tech">UDS (ISO 14229)</span>
<span class="badge-tech">KWP2000 (ISO 14230)</span>
<span class="badge-tech">CAN 2.0A / 2.0B</span>
<span class="badge-tech">Seed/Key (SID 0x27)</span>
<span class="badge-tech">Flash Memory Emulation</span>

<hr class="section-divider">

## <i class="fa-solid fa-cogs icon-analysis"></i> Technical Features & Interface

### 1. Assembly View & Disassembly Panel

The central execution window provides a **scrollable disassembly** of the loaded firmware. Each row shows:

| Column | Description |
|--------|-------------|
| Address | 32-bit linear address within the emulated address space |
| Hex Opcode | Raw 4-byte PPC instruction word |
| Mnemonic | Decoded PowerPC assembly (e.g. `mfmsr`, `ori`, `mtspr`, `stw`) |
| Breakpoints | Red/white dot per row — click to toggle a hardware breakpoint |

The highlighted yellow row always tracks the **current Program Counter (PC)**. Two breakpoints shown at `0x00000040` and `0x00000044` illustrate mid-routine interception of the security seed generation sequence.

<div class="row mt-3">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid
       loading="eager"
       path="assets/img/PISAD/1.PNG"
       title="Assembly View — PPC disassembly with breakpoints"
       class="img-fluid rounded z-depth-1 shadow-lg" %}
  </div>
</div>
<div class="caption text-info">
  <i class="fa-solid fa-code"></i>
  <strong>Assembly View:</strong> Live PPC disassembly with the PC at <code>0x00000004</code> and two active breakpoints (red dots) at <code>0x00000040</code> / <code>0x00000044</code>.
  The right panel lists analogue variable names with their ECU memory addresses and current raw values.
</div>

---

### 2. Register & Variable Management (Analogue Panel)

Before or during execution, every **GPR (R0–R31)**, **SPR (LR, SP, CTR, CR)**, and software-level variable can be **read and overridden** via the left-hand register sidebar and the right-hand analogue variable list.

The analogue panel maps **ECU software labels** (e.g. `DEV_U1L_T_Fuel`, `DEV_T_Intake_Air`, `DEV_T_Coolant_InterCooler`) to their RAM addresses (e.g. `102e078`, `102c980`) and allows injecting arbitrary values before execution — effectively simulating any sensor reading the firmware would see.

<div class="row mt-3">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid
       loading="eager"
       path="assets/img/PISAD/3.PNG"
       title="Analogues Input Panel"
       class="img-fluid rounded z-depth-1 shadow-lg" %}
  </div>
</div>
<div class="caption text-success">
  <i class="fa-solid fa-pen-to-square"></i>
  <strong>Analogue / Variable Management:</strong> PI inputs (PI1–PI12), digital inputs (DI1–DI8),
  ITS signals, and 2.5 V reference rails can all be pre-set before running the firmware, simulating arbitrary vehicle states.
</div>

---

### 3. CAN Bus Simulation

The **CAN tab** exposes three independent CAN channels (CAN A, CAN B, CAN C), each with a configurable frame table:

- **Index** — frame sequence number
- **ID** — 11-bit or 29-bit arbitration identifier
- **Length** — DLC (0–8 bytes)
- **Data 0–7** — payload bytes, editable per frame

This allows the researcher to replay real-world bus captures or craft custom diagnostic frames (e.g. `0x7DF` broadcast requests for SID 0x27) and observe how the ECU firmware reacts without any physical CAN hardware.

<div class="row mt-3">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid
       loading="eager"
       path="assets/img/PISAD/4.PNG"
       title="CAN Bus Simulation Channels"
       class="img-fluid rounded z-depth-1 shadow-lg" %}
  </div>
</div>
<div class="caption text-warning">
  <i class="fa-solid fa-network-wired"></i>
  <strong>CAN Bus Panel:</strong> Three independent CAN channels (A / B / C) each support custom frame injection.
  The right-hand variable monitor simultaneously shows live ECU internal values such as
  <code>DEV_DBR_MCR_Torque</code>, <code>DEV_BOI_Norm</code>, and <code>DEV_Pause_Pilot_Injection</code>.
</div>

---

### 4. HexView — Raw Flash Memory Inspector

The **HexView** tab renders the loaded firmware as a classic hex dump, with:

- **Region selector** (`Flash_AB`, internal RAM, etc.)
- **GoTo address** for instant navigation
- **Data-type overlay** — toggle between `int8`, `uint16`, `float32`, and more to interpret raw bytes on the fly
- **Binary representation** shown alongside hex for bitfield analysis

<div class="row mt-3">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid
       loading="eager"
       path="assets/img/PISAD/2.PNG"
       title="HexView — Flash Memory Dump"
       class="img-fluid rounded z-depth-1 shadow-lg" %}
  </div>
</div>
<div class="caption text-danger">
  <i class="fa-solid fa-magnifying-glass"></i>
  <strong>HexView:</strong> Raw firmware bytes of <code>Flash_AB</code> displayed from address <code>0x00000000</code>.
  The data-type selector (<code>int8</code> shown) and binary panel (right) allow rapid identification of constants,
  calibration tables, and cryptographic seeds embedded in the ECU flash.
</div>

<hr class="section-divider">

## <i class="fa-solid fa-route icon-workflow"></i> Reverse Engineering Workflow

<div class="alert alert-warning">
  <i class="fa-solid fa-key"></i> <strong>Key Insight:</strong>
  By placing breakpoints at the exact offset where the ECU's security access routine begins
  (typically the function that responds to UDS SID <code>0x27</code> — <em>SecurityAccess</em>),
  you can pause execution at the moment the <strong>seed</strong> is loaded into a GPR,
  read it directly from the register panel, and step through the transformation logic
  to recover the full <strong>Seed → Key</strong> algorithm without any firmware decryption.
</div>

<div class="step-row"><span class="step-num">1</span>
  <div><strong>Load ECU Dump</strong> — Import the full binary flash image (<code>.bin</code> / <code>.hex</code> / <code>.s19</code>).
  The emulator maps Flash, RAM, and peripheral register regions according to the MPC5xx memory map.</div>
</div>
<div class="step-row"><span class="step-num">2</span>
  <div><strong>Set Analogue Conditions</strong> — Use the Analogues panel to pre-configure sensor inputs
  (coolant temp, fuel pressure, rail pressure, etc.) so the ECU firmware starts in a valid engine-running state
  rather than entering a safe-mode or limp-home branch.</div>
</div>
<div class="step-row"><span class="step-num">3</span>
  <div><strong>Inject Diagnostic Request</strong> — Via the CAN panel, send a UDS <code>0x27 01</code>
  (RequestSeed) frame. The firmware receives it through the emulated CAN controller peripheral and begins
  executing its security subroutine.</div>
</div>
<div class="step-row"><span class="step-num">4</span>
  <div><strong>Break at Seed Generation</strong> — With a breakpoint set at the seed-computation offset,
  execution pauses and all GPR values are frozen. Read R3/R4 (or whichever register holds the seed)
  directly from the left-hand register panel.</div>
</div>
<div class="step-row"><span class="step-num">5</span>
  <div><strong>Step & Trace the Algorithm</strong> — Single-step through the PPC instructions, observing
  each register mutation. The sequence of <code>ori</code>, <code>mtspr</code>, <code>li</code>, and shift
  operations reveals the exact mathematical transformation <strong>f(seed) = key</strong>.</div>
</div>
<div class="step-row"><span class="step-num">6</span>
  <div><strong>Validate & Export</strong> — Replay with different seeds to confirm the recovered algorithm,
  then export it as a standalone key-calculation script for use in diagnostic tools.</div>
</div>

<hr class="section-divider">

## <i class="fa-solid fa-shield-alt icon-security"></i> Key Capabilities

<div class="row">
  <div class="col-md-6">
    <div class="feature-card">
      <h5><i class="fa-solid fa-microchip"></i> Full PPC Instruction Emulation</h5>
      <p>Interprets the complete PowerPC embedded instruction set including
      <code>mfmsr/mtmsr</code>, <code>mfspr/mtspr</code>, supervisor calls,
      and all fixed-point, branch, and load/store operations.</p>
    </div>
  </div>
  <div class="col-md-6">
    <div class="feature-card">
      <h5><i class="fa-solid fa-network-wired icon-hardware"></i> UDS / KWP2000 Protocol Stack</h5>
      <p>Simulates full ISO 14229 (UDS) and ISO 14230 (KWP2000) transport layers over the emulated
      CAN controller, enabling end-to-end diagnostic session testing without physical hardware.</p>
    </div>
  </div>
  <div class="col-md-6">
    <div class="feature-card">
      <h5><i class="fa-solid fa-memory icon-analysis"></i> Multi-Region Memory Map</h5>
      <p>Separate Flash (AB), internal SRAM, and peripheral address spaces, each independently
      browsable via HexView with data-type overlays and binary display.</p>
    </div>
  </div>
  <div class="col-md-6">
    <div class="feature-card">
      <h5><i class="fa-solid fa-sliders icon-workflow"></i> Analogue & Digital I/O Injection</h5>
      <p>All physical sensor channels (PI, DI, ITS, PTI, PFI, LSI, KTY) and analogue references
      can be set to arbitrary values before or during execution, simulating any vehicle operating condition.</p>
    </div>
  </div>
  <div class="col-md-6">
    <div class="feature-card">
      <h5><i class="fa-solid fa-circle-dot icon-security"></i> Hardware Breakpoints</h5>
      <p>Per-instruction breakpoints set directly in the disassembly view. Execution pauses instantly,
      freezing all register and memory state for inspection.</p>
    </div>
  </div>
  <div class="col-md-6">
    <div class="feature-card">
      <h5><i class="fa-solid fa-car icon-hardware"></i> ECU Target Support</h5>
      <p>Validated against firmware dumps from <strong>ME17</strong>, <strong>Changan</strong>,
      and <strong>Lamari</strong> ECU families. Extendable to any PPC-based PISAD variant.</p>
    </div>
  </div>
</div>

<hr class="section-divider">

## <i class="fa-solid fa-flask icon-analysis"></i> Use Cases

| Use Case | How the Simulator Helps |
|----------|------------------------|
| **Seed/Key Algorithm Recovery** | Breakpoint at SID 0x27 handler → step through transformation → extract algorithm |
| **Calibration Table Analysis** | HexView + data-type overlay to locate and interpret lookup tables in flash |
| **Diagnostic Protocol Fuzzing** | Inject malformed UDS frames via CAN panel and observe firmware error handling |
| **Firmware Validation** | Run production firmware against edge-case sensor values without hardware risk |
| **Security Audit** | Enumerate all reachable code paths from external diagnostic sessions |

<hr class="section-divider">

## <i class="fa-solid fa-code icon-hardware"></i> Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│                  PISAD ECU Simulator                │
├──────────────┬──────────────┬────────────────────────┤
│  PPC Core    │  Memory Bus  │  Peripheral Emulation  │
│  ─────────── │  ─────────── │  ──────────────────    │
│  GPR R0–R31  │  Flash AB    │  CAN A / B / C         │
│  SPR (LR,CTR)│  SRAM        │  Analogue inputs       │
│  MSR / CR    │  Peripheral  │  Digital I/O           │
│  PC tracker  │  Registers   │  KTY / ITS sensors     │
├──────────────┴──────────────┴────────────────────────┤
│              Debugger / Control Layer                │
│  Breakpoints · Single-step · Register override ·     │
│  HexView · Disassembly · Variable monitor            │
├─────────────────────────────────────────────────────┤
│          Diagnostic Protocol Stack (UDS/KWP)         │
│  ISO 14229 · ISO 14230 · CAN frame inject/receive    │
└─────────────────────────────────────────────────────┘
```

<hr class="section-divider">

## <i class="fa-solid fa-circle-info icon-hardware"></i> Notes & Disclaimer

<div class="alert alert-danger">
  <i class="fa-solid fa-triangle-exclamation"></i> <strong>Research Use Only.</strong>
  This simulator is intended exclusively for <strong>authorised security research, education, and firmware validation</strong>.
  Using recovered authentication algorithms on vehicles or ECUs without the owner's explicit permission may violate
  computer fraud and vehicle security regulations in your jurisdiction.
  Always obtain proper authorisation before performing security assessments on automotive systems.
</div>
