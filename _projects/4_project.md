---
layout: page
title: Security Assessment of Software Protection in Diagnostic Tools
description: <i class="fa-solid fa-shield-halved"></i> Assessing protection layers in automotive diagnostic software using x32dbg and anti-tamper analysis.
img: assets/img/x32dbg.PNG
importance: 4
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

Defensive Analysis of the Protection of ECU Programming Tools: This research focuses on the technical evaluation of the software-protection mechanisms implemented in automotive diagnostic tools, specifically a commercial ECU programming tool, as research and development through lawful, authorised corporate and personal processes. The project analyzes how these tools defend against reverse engineering and unauthorized tampering, and what a stronger design would look like. It reports protection categories only; no bypass methods, patches or cracked software are published.

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

### 2. Memory Integrity & VirtualProtect
A critical part of the analysis involved monitoring the Windows `VirtualProtect` API calls. This mechanism is used to dynamically change memory page permissions (e.g., flipping from PAGE_READWRITE to PAGE_EXECUTE_READ) to prevent code injection and live memory dumping during runtime.

### 3. Dynamic Analysis with x32dbg
Using **x32dbg**, I analysed how the software's protection layers behave at runtime. By utilizing hardware breakpoints and tracing API calls, I observed how the software protects sensitive operations like communication with the hardware dongle, in order to assess the strength of those protections.

<div class="row justify-content-sm-center">
    <div class="col-sm-10 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/x32dbg.PNG" title="Dynamic Analysis in x32dbg" class="img-fluid rounded z-depth-1 shadow-lg" %}
    </div>
</div>
<div class="caption text-danger">
    <strong>Debugging Session:</strong> Monitoring register states to observe how the software's integrity check behaves.
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

<div class="alert alert-warning">
  <i class="fa-solid fa-triangle-exclamation"></i> <strong>Research Use Only.</strong>
  This is research and development carried out through lawful, authorised corporate and personal processes, on bench hardware, for defensive security and education. It is not intended for circumventing licensing or copy protection of commercial software, and no bypass instructions are provided. Reverse engineering of commercial software may be restricted by licence terms and by anti-circumvention laws (for example in the US, EU and Canada).
</div>
