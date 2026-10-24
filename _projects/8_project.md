---
layout: page
title: Reverse Engineering of an Automotive Engine Control Unit
description: <i class="fa-solid fa-microchip"></i> Defensive full-stack analysis of an engine ECU, from hardware interfacing to line-by-line binary analysis and logic understanding.
img: assets/img/ecu-bench-setup-2.jpg
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

The **engine control unit (ECU)** is the central brain of modern vehicles. This project is a **defensive security and functional assessment** of such a controller, performed on a bench unit as research and development through lawful, authorised corporate and personal processes. By analyzing the firmware, I studied the logic behind its safety systems, diagnostic protocols, and security handshakes in order to understand how they work and where they could be strengthened. No keys, algorithms, firmware images or exploit code are published on this page.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <span class="tech-badge bg-boot">Bootloader Analysis</span>
        <span class="tech-badge bg-fault">DTC Logic</span>
        <span class="tech-badge bg-crypto">Security Access (0x27)</span>
    </div>
</div>

---

## <i class="fa-solid fa-microchip" style="color: #2c3e50;"></i> Hardware Context

The research began with the physical ECU hardware. Establishing a stable bench connection was essential for acquiring the firmware image for static analysis and monitoring live diagnostic traffic.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/ecu-bench-setup-2.jpg" title="ECU Hardware" class="img-fluid rounded z-depth-1 shadow-lg" style="width: 100%; height: 350px; object-fit: cover;" %}
    </div>
</div>
<div class="caption">
    <strong>The Target:</strong> ECU hardware environment used for live monitoring and bench analysis.
</div>

---

## <i class="fa-solid fa-code-branch" style="color: #16a085;"></i> Deep Logic Extraction (Line-by-Line Analysis)

Using advanced Disassemblers, the binary artifacts were translated into structured C code. The following core subsystems were identified and analyzed:

### 1. The Boot & Initialization "Brain" (`sub_22D0`)
This function serves as the mastermind for ECU startup.
- **Clock Configuration:** Tuning the system to **80MHz** with a high-precision margin (error < 0.33%).
- **Protocol Setup:** Initializing the **CAN network** at 500kbps and verifying memory integrity via specific markers like `0xDEADBEEF`.

### 2. Cryptographic Security Access (`sub_23D2C`)
I performed a line-by-line analysis of the **UDS Service 0x27** (SecurityAccess) handler to assess its design.
- **Seed-to-Key Transformation:** Characterised the general structure of the transformation used by the ECU, in order to judge its strength.
- **Design Review:** Reviewing the XOR and bit-shifting operations that protect the ECU's flash memory from unauthorized access, and what a stronger design would look like. Algorithm details are deliberately not published.



### 3. Fault Management & System Health (`sub_2B2C`)
Identified as the **Fault Manager**, this routine acts as the ECU's internal doctor:
- **DTC Generation:** Monitoring sensors and incrementing error counters.
- **Threshold Logic:** Deciding when a transient glitch should be logged as a permanent **Diagnostic Trouble Code (DTC)**.
- **Fail-Safe Modes:** Activating protective routines (Fault Reactions) to prevent engine damage during sensor failures.

---

## <i class="fa-solid fa-terminal" style="color: #27ae60;"></i> Key Technical Accomplishments

* <i class="fa-solid fa-check text-success"></i> **Binary Analysis:** Studied the handling of the `0x2E` (WriteData) and `0x3D` (WriteMemory) services to understand which access checks protect them.
* <i class="fa-solid fa-check text-success"></i> **Architecture Mapping:** Full understanding of the PowerPC-based execution flow and register settings.
* <i class="fa-solid fa-check text-success"></i> **Security Assessment:** Identification of design weaknesses in the Seed/Key exchange, described at a conceptual level only, with hardening recommendations.

<div class="alert alert-info">
  <i class="fa-solid fa-lightbulb"></i> <strong>Defensive Insight:</strong> This level of analysis is what manufacturers and security teams need in order to verify access controls, harden the Seed/Key exchange, and design security patches for automotive embedded systems.
</div>

<div class="alert alert-warning">
  <i class="fa-solid fa-triangle-exclamation"></i> <strong>Research Use Only.</strong>
  This work is research and development carried out through lawful, authorised corporate and personal processes, on bench hardware, for defensive security and education. It is not intended for circumventing access control, altering emissions, mileage or safety functions, or modifying any vehicle without the owner's permission, which may be unlawful (for example under US, EU and Canadian law).
</div>