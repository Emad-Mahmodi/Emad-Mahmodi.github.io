---
layout: page
title: Advanced Reverse Engineering & Hardware Interfacing of Lamari DU7Z ECU
description: <i class="fa-solid fa-microchip"></i> A comprehensive study on multi-core firmware architecture, security access bypassing, and live CAN-Bus monitoring for ECU diagnostics.
img: assets/img/lamari-ima-frontside-2.jfif
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

This project represents a complete end-to-end security audit of the **Lamari DU7Z ECU**. By combining deep-level firmware reverse engineering with physical hardware interfacing, I successfully mapped the internal logic of the controller, identified security vulnerabilities, and established a framework for live diagnostic monitoring.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <span class="tech-badge bg-multicore">Multi-core Architecture</span>
        <span class="tech-badge bg-security">Seed/Key Extraction</span>
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
        {% include figure.liquid loading="eager" path="assets/img/lamari2.PNG" title="Decompiled Code Logic" class="img-fluid rounded z-depth-1 shadow-lg" style="width: 100%; height: 300px; object-fit: cover;" %}
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
        {% include figure.liquid loading="eager" path="assets/img/lamari3.jpg" title="ECU Bench Setup" class="img-fluid rounded z-depth-1 shadow-lg" style="width: 50%; height: 200; object-fit: cover;" %}
    </div>
</div>
<div class="caption">
    <strong>Hardware Bench:</strong> Establishing CAN-Bus communication for live monitoring and diagnostic verification of the ECU.
</div>

---

## <i class="fa-solid fa-key" style="color: #f1c40f;"></i> Security & Integrity Research

### Seed/Key Algorithm Reconstruction (Service 27)
I successfully reverse-engineered the **Seed/Key transformation** logic. By isolating complex bitwise operations (XORs, Shifts) and substitution tables in the firmware, I reconstructed the algorithm required to bypass security levels in the UDS protocol.

### Integrity & Checksum Analysis
The research also covered the ECU's self-protection mechanisms. I identified and analyzed the algorithms (such as custom CRC-32 routines like `FUN_000044c0`) responsible for verifying firmware integrity and preventing unauthorized calibration modifications.

---

## <i class="fa-solid fa-list-check" style="color: #27ae60;"></i> Key Accomplishments

* <i class="fa-solid fa-check text-success"></i> **Architecture Mapping:** Full analysis of multi-core load balancing.
* <i class="fa-solid fa-check text-success"></i> **Security Bypass:** Extracted proprietary Seed/Key logic for Service 0x27.
* <i class="fa-solid fa-check text-success"></i> **Hardware Success:** Successful dump extraction and live CAN monitoring.
* <i class="fa-solid fa-check text-success"></i> **Integrity Analysis:** Identification of firmware validation and Checksum routines.

<div class="alert alert-info">
  <i class="fa-solid fa-code"></i> <strong>Methodology Note:</strong> This project highlights the synergy between static binary analysis (Ghidra/IDA) and dynamic hardware monitoring (CAN Analyzers) to achieve a full understanding of proprietary embedded systems.
</div>