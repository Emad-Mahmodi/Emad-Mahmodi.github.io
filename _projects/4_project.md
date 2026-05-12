---
# published: false
layout: page
title: Vulnerability ECU Tools
description: A technical audit of automotive diagnostic software layers, focusing on anti-debugging techniques, V-table obfuscation, and runtime memory protection using x32dbg.
img: assets/img/images.jfif
importance: 4
category: work



---
layout: page
title: Security Auditing of Diagnostic Ecosystems
description: <i class="fa-solid fa-shield-halved"></i> Auditing protection layers in automotive software (TNM) using x32dbg and anti-tamper analysis.
img: assets/img/images.jfif
importance: 2
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
  .bg-x32 { background-color: #34495e; }
  .bg-virtual { background-color: #8e44ad; }
  .bg-obfuscation { background-color: #2c3e50; }
</style>

## <i class="fa-solid fa-microscope" style="color: #3498db;"></i> Project Overview

Reverse Engineering and Vulnerability Analysis of ECU Programming Tools: This research focuses on the technical evaluation of security mechanisms implemented in automotive diagnostic tools, specifically the **TNM** software. The project analyzes how these tools defend against reverse engineering and unauthorized tampering through advanced software protection layers.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <span class="tech-badge bg-x32">x32dbg Analysis</span>
        <span class="tech-badge bg-virtual">Virtual Functions</span>
        <span class="tech-badge bg-obfuscation">Memory Protection</span>
        <span class="tech-badge bg-x32">Anti-Debugging</span>
    </div>
</div>

---

## <i class="fa-solid fa-user-shield" style="color: #e74c3c;"></i> Security Mechanisms Evaluated

### 1. Virtual Functions & V-Table Obfuscation
The application utilizes complex `Virtual Functions` to decouple the logic from the binary's static structure. By obfuscating the **vtable**, the software makes it significantly harder for static analysis tools to reconstruct the execution flow and call graphs.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/tnm.PNG" title="TNM Diagnostic Interface" class="img-fluid rounded z-depth-1 shadow-lg" %}
    </div>
</div>
<div class="caption">
    The TNM Diagnostic interface, used as a case study for evaluating commercial software protection.
</div>

### 2. Memory Integrity & VirtualProtect
A critical part of the analysis involved monitoring the Windows `VirtualProtect` API calls. This mechanism is used to dynamically change memory page permissions (e.g., flipping from PAGE_READWRITE to PAGE_EXECUTE_READ) to prevent code injection and live memory dumping during runtime.

### 3. Dynamic Analysis with x32dbg
Using **x32dbg**, I conducted a deep-dive analysis of the software's decryption routines. By utilizing hardware breakpoints and tracing API calls, I investigated how the software handles sensitive operations like communication with the hardware dongle.

<div class="row justify-content-sm-center">
    <div class="col-sm-10 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/x32dbg.PNG" title="Dynamic Analysis in x32dbg" class="img-fluid rounded z-depth-1 shadow-lg" %}
    </div>
</div>
<div class="caption text-danger">
    <strong>Debugging Session:</strong> Monitoring register states and identifying conditional jumps during the software's integrity check.
</div>

---

## <i class="fa-solid fa-list-check" style="color: #2ecc71;"></i> Technical Comparison

The following table summarizes the observed protection layers and their effectiveness against standard reverse engineering workflows:

| Protection Layer | Complexity | Analysis Tool | Observed Status |
| :--- | :--- | :--- | :--- |
| **VirtualProtect** | Medium | x32dbg | Active (Dynamic) |
| **V-Table Hooking** | High | Scylla / x32dbg | Identified |
| **Anti-Debugging** | Very High | ScyllaHide | Multiple Layers |
| **Function Virtualization** | High | x32dbg / IDA | Implemented |

---

## <i class="fa-solid fa-diagram-project" style="color: #f39c12;"></i> Proposed Security Framework

To enhance the resilience of automotive diagnostic software, a multi-layered defense strategy is proposed. This includes implementing **Polymorphic Code** and **Dynamic Integrity Verification** to ensure that critical logic cannot be easily bypassed via patching.

<div class="row">
    <div class="col-sm-12 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/security_flow.png" title="Proposed Security Flow" class="img-fluid rounded z-depth-1 shadow-lg" %}
    </div>
</div>
<div class="caption">
    Conceptual diagram of an enhanced, multi-layered software protection architecture.
</div>