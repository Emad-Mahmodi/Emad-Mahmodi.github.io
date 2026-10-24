---
layout: page
title: Advanced Reverse Engineering & Hardware Interfacing of a Multi-Core Automotive ECU
description: <i class="fa-solid fa-microchip"></i> A defensive study of multi-core firmware architecture, SecurityAccess (UDS 0x27) robustness, and live CAN-Bus monitoring for ECU diagnostics.
img: assets/img/ecu-disassembly.png
importance: 7
category: work
---

<style>
  .tech-badge {
    display: inline-block;
    padding: 2px 10px;
    margin: 2px;
    border-radius: 15px;
    font-size: 0.85em;
    font-weight: bold;
    color: white;
  }
  .bg-multicore { background-color: #e67e22; }
  .bg-security { background-color: #c0392b; }
  .bg-hardware { background-color: #34495e; }
</style>

## <i class="fa-solid fa-shield-halved" style="color: #c0392b;"></i> Project Overview

This project is a defensive security assessment of the **multi-core automotive ECU**, carried out on a bench unit as research and development through lawful, authorised corporate and personal processes. By combining firmware reverse engineering with physical hardware interfacing, I mapped the internal logic of the controller, identified design weaknesses that a vendor would want to harden, and established a framework for live diagnostic monitoring. The goal is understanding and protection — no keys, algorithms, firmware images or exploit code are published on this page.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <span class="tech-badge bg-multicore">Multi-core Architecture</span>
        <span class="tech-badge bg-security">SecurityAccess Analysis</span>
        <span class="tech-badge bg-hardware">CAN-Bus Interfacing</span>
    </div>
</div>

---

## <i class="fa-solid fa-gears" style="color: #3498db;"></i> Firmware Reverse Engineering

### 1. Multi-core Architecture Analysis
The ECU utilizes a high-performance multi-core processor (Core 0, 1, and 2). By analyzing references like `in_CORE_ID`, I mapped the distribution of tasks across cores, focusing on how the system balances real-time engine management with diagnostic interrupts.

### 2. Memory-Mapped I/O & Register Tracking
The analysis of decompiled C code revealed direct hardware interactions via memory-mapped addresses (e.g., `DAT_f0036100`). This enabled the identification of critical registers used for sensor acquisition and actuator triggering.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/ecu-disassembly.png" title="Decompiled Code Logic" class="img-fluid rounded z-depth-1 shadow-lg" style="width: 100%; height: 300px; object-fit: cover;" %}
    </div>
</div>
<div class="caption">
    Analyzing decompiled logic to understand the interaction between software variables and hardware registers.
</div>

---

## <i class="fa-solid fa-link" style="color: #34495e;"></i> Hardware Interfacing & CAN Monitoring

A key part of this project was the transition from static analysis to live verification. Using a specialized bench setup, I established a **CAN-Bus** connection to the ECU to monitor its operational behavior.

### Live Diagnostic Monitoring
- **Read/Write Operations:** Intercepting data streams during flash memory operations to analyze the bootloader's behavior.
- **Protocol Analysis:** Monitoring the UDS (Unified Diagnostic Services) stack to identify proprietary Service IDs (SIDs) and their corresponding response patterns.
- **Functionality Verification:** Using the bench setup to simulate vehicle states and observe how the decompiled logic reacts in a real-time hardware environment.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/ecu-bench-setup-1.jpg" title="ECU Bench Setup" class="img-fluid rounded z-depth-1 shadow-lg" style="width: 50%; height: 200; object-fit: cover;" %}
    </div>
</div>
<div class="caption">
    <strong>Hardware Bench:</strong> Establishing CAN-Bus communication for live monitoring and diagnostic verification of the ECU.
</div>

---

## <i class="fa-solid fa-key" style="color: #f1c40f;"></i> Security & Integrity Research

### SecurityAccess Robustness Review (Service 0x27)
I studied the structure of the **Seed/Key challenge-response** in the firmware: the use of bitwise operations (XORs, shifts) and lookup tables, the quality of the seed, and the protections around it. The aim was to judge how well the mechanism resists misuse and to derive hardening recommendations (stronger seed entropy, a keyed cryptographic MAC instead of a fixed transformation, attempt limiting and lock-out delays). Details of the algorithm are deliberately not published.

### Integrity & Checksum Analysis
The research also covered the ECU's self-protection mechanisms. I identified and analyzed the routines (such as custom CRC-32 routines like `FUN_000044c0`) responsible for verifying firmware integrity, in order to evaluate how well the ECU protects itself against unintended or malicious modification of its calibration.

---

## <i class="fa-solid fa-list-check" style="color: #27ae60;"></i> Key Accomplishments

* <i class="fa-solid fa-check text-success"></i> **Architecture Mapping:** Full analysis of multi-core load balancing.
* <i class="fa-solid fa-check text-success"></i> **Security Assessment:** Evaluated the SecurityAccess (Service 0x27) design and derived hardening recommendations.
* <i class="fa-solid fa-check text-success"></i> **Hardware Setup:** Bench acquisition of the firmware image for static analysis and live CAN monitoring.
* <i class="fa-solid fa-check text-success"></i> **Integrity Analysis:** Identification of firmware validation and Checksum routines.

<div class="alert alert-info">
  <i class="fa-solid fa-code"></i> <strong>Methodology Note:</strong> This project highlights the synergy between static binary analysis (Ghidra/IDA) and dynamic hardware monitoring (CAN Analyzers) to achieve a full understanding of proprietary embedded systems.
</div>

<div class="alert alert-warning">
  <i class="fa-solid fa-triangle-exclamation"></i> <strong>Research Use Only.</strong>
  This work is research and development carried out through lawful, authorised corporate and personal processes, on bench hardware, for defensive security and education. It is not intended for circumventing access control, altering emissions, mileage or safety functions, or modifying any vehicle without the owner's permission, which may be unlawful (for example under US, EU and Canadian law).
</div>