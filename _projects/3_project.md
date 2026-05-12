---
layout: page
title: ECU Emulatore
description: This project is a PISAD-based (PISAD PPC ECU Full Simulatore) full simulation operation for the complete ECU dump of vehicles with PowerPC (PPC) architecture.
img: assets/img/reverse.png
# redirect: https://unsplash.com
importance: 3
category: work


---
layout: page
title: PISAD ECU Full Simulator
description: <i class="fa-solid fa-microchip"></i> A high-level PowerPC (PPC) emulator for ECU firmware analysis & security reverse engineering.
img: assets/img/reverse.png
importance: 1
category: work
---

<style>
  .project-icon {
    margin-right: 10px;
  }
  .icon-hardware { color: #5dade2; } /* آبی روشن */
  .icon-security { color: #e74c3c; } /* قرمز */
  .icon-analysis { color: #2ecc71; } /* سبز */
  .icon-workflow { color: #f39c12; } /* نارنجی */
</style>

## <i class="fa-solid fa-info-circle project-icon icon-hardware"></i>Introduction

This project is a dedicated simulation environment for **PISAD-based ECUs** utilizing the **PowerPC (PPC)** architecture. Unlike generic emulators, this tool is optimized for automotive firmware, allowing researchers to load full ECU dumps and interact with the virtual processor at a hardware-abstracted level.

<div class="alert alert-info">
  <strong><i class="fa-solid fa-lightbulb"></i> The Challenge:</strong> 
  Reverse engineering automotive ECUs often requires bypassing hardware security. This simulator provides a safe <strong>sandbox</strong> to test authentication algorithms without risking physical hardware.
</div>

---

## <i class="fa-solid fa-cogs project-icon icon-analysis"></i>Technical Features & Interface

### 1. Register & Variable Management
This is the core interface for defining the initial state. Users can manually override or initialize specific processor registers and system variables to simulate different vehicle conditions or cheat security checks.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/registers_input.png" title="Register Input Interface" class="img-fluid rounded z-depth-1 shadow-lg" %}
    </div>
</div>
<div class="caption text-info">
    <i class="fa-solid fa-pen-to-square"></i> <strong>Register Management:</strong> Pre-defining initial states before execution.
</div>

### 2. Live CPU State Monitoring
During execution, the simulator provides a real-time view of the **PPC Registers (<code>GPRs</code>, <code>SPRs</code>)**. This is essential for tracing security-critical functions.

<div class="row justify-content-sm-center">
    <div class="col-sm-10 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/cpu_state.png" title="Current CPU State" class="img-fluid rounded z-depth-1 shadow-lg" %}
    </div>
</div>
<div class="caption text-danger">
    <i class="fa-solid fa-eye"></i> <strong>CPU State:</strong> Values of registers during a security access routine.
</div>

---

## <i class="fa-solid fa-route project-icon icon-workflow"></i>Reverse Engineering Workflow

<div class="alert alert-warning">
  <i class="fa-solid fa-key"></i> <strong>Key Insight:</strong> 
  By pausing execution at specific offsets (e.g., authentication starts), one can analyze constants used in the <strong>Seed/Key</strong> transformation (SID 0x27).
</div>

<div class="row">
    <div class="col-sm-6 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/memory_view.png" title="Memory View" class="img-fluid rounded z-depth-1" %}
        <div class="caption">Memory mapping of Flash.</div>
    </div>
    <div class="col-sm-6 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/variables_monitor.png" title="Variables Monitor" class="img-fluid rounded z-depth-1" %}
        <div class="caption">Monitoring software variables.</div>
    </div>
</div>

## <i class="fa-solid fa-shield-alt project-icon icon-security"></i>Key Capabilities

* <i class="fa-solid fa-check text-success"></i> <strong>Architecture:</strong> Full PowerPC (PPC) instruction set interpretation.
* <i class="fa-solid fa-network-wired text-primary"></i> <strong>Diagnostics:</strong> UDS/KWP2000 protocol simulation.
* <i class="fa-solid fa-brain text-warning"></i> <strong>Verification:</strong> Perfect for analyzing algorithms in ME17, Changan, or Lamari ECUs.

---