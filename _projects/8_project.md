---
layout: page
title: Reverse Engineering of MSE2 Engine Control Unit
description: <i class="fa-solid fa-motorcycle"></i> Full-stack analysis of Bajaj MSE2 ECU, from hardware interfacing to line-by-line binary de-obfuscation and logic reconstruction.
img: assets/img/Bajaj-1.webp
importance: 8
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
  .bg-boot { background-color: #2980b9; }
  .bg-fault { background-color: #c0392b; }
  .bg-crypto { background-color: #8e44ad; }
</style>

## <i class="fa-solid fa-gauge-high" style="color: #d35400;"></i> Project Overview

The **MSE2 ECU** is the central brain of many modern motorcycles, including the Bajaj Pulsar series. This project represents a complete **Security & Functional Audit** of this controller. By extracting and analyzing the firmware, I reconstructed the logic behind its safety systems, diagnostic protocols, and security handshakes.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <span class="tech-badge bg-boot">Bootloader Analysis</span>
        <span class="tech-badge bg-fault">DTC Logic</span>
        <span class="tech-badge bg-crypto">Security Access (0x27)</span>
    </div>
</div>

---

## <i class="fa-solid fa-microchip" style="color: #2c3e50;"></i> Hardware Context

The research began with the physical ECU hardware. Establishing a stable bench connection was essential for capturing the firmware dump and monitoring live diagnostic traffic.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/mse2.jpg" title="MSE2 ECU Hardware" class="img-fluid rounded z-depth-1 shadow-lg" style="width: 100%; height: 350px; object-fit: cover;" %}
    </div>
</div>
<div class="caption">
    <strong>The Target:</strong> MSE2 ECU hardware environment used for live monitoring and firmware acquisition.
</div>

---

## <i class="fa-solid fa-code-branch" style="color: #16a085;"></i> Deep Logic Extraction (Line-by-Line Analysis)

Using advanced Disassemblers, the binary artifacts were translated into structured C code. The following core subsystems were identified and analyzed:

### 1. The Boot & Initialization "Brain" (`sub_22D0`)
This function serves as the mastermind for ECU startup.
- **Clock Configuration:** Tuning the system to **80MHz** with a high-precision margin (error < 0.33%).
- **Protocol Setup:** Initializing the **CAN network** at 500kbps and verifying memory integrity via specific markers like `0xDEADBEEF`.

### 2. Cryptographic Security Access (`sub_23D2C`)
I performed a line-by-line de-obfuscation of the **UDS Service 0x27** handler.
- **Seed-to-Key Transformation:** Reconstructed the mathematical model used by the ECU to generate dynamic security keys.
- **Algorithm Analysis:** Mapping the sequential XORs and bit-shifting operations used to protect the ECU's flash memory from unauthorized access.



### 3. Fault Management & System Health (`sub_2B2C`)
Identified as the **Fault Manager**, this routine acts as the ECU's internal doctor:
- **DTC Generation:** Monitoring sensors and incrementing error counters.
- **Threshold Logic:** Deciding when a transient glitch should be logged as a permanent **Diagnostic Trouble Code (DTC)**.
- **Fail-Safe Modes:** Activating protective routines (Fault Reactions) to prevent engine damage during sensor failures.

---

## <i class="fa-solid fa-terminal" style="color: #27ae60;"></i> Key Technical Accomplishments

* <i class="fa-solid fa-check text-success"></i> **Binary Reconstruction:** Deciphered the logic for `0x2E` (WriteData) and `0x3D` (WriteMemory) services.
* <i class="fa-solid fa-check text-success"></i> **Architecture Mapping:** Full understanding of the PowerPC-based execution flow and register settings.
* <i class="fa-solid fa-check text-success"></i> **Security Auditing:** Identification of vulnerabilities in the Seed/Key exchange process.

<div class="alert alert-info">
  <i class="fa-solid fa-lightbulb"></i> <strong>Developer Insight:</strong> This level of analysis is the prerequisite for developing custom tuning tools, flash downloaders, and security patches for automotive embedded systems.
</div>