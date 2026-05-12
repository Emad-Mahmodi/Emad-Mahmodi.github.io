---
layout: page
title: Automotive Protocol Analyzer & Monitoring Tool
description: <i class="fa-solid fa-microscope"></i> A high-performance monitoring app built with Qt/QML and Python to analyze Saleae Logic exports for UDS and OBD-II diagnostics.
img: assets/img/software_monitoring.png
importance: 6
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
  .bg-qt { background-color: #41cd52; color: #000; } /* Qt Green */
  .bg-python { background-color: #3776ab; } /* Python Blue */
  .bg-uds { background-color: #c0392b; }
</style>

## <i class="fa-solid fa-terminal" style="color: #2980b9;"></i> Project Overview

Automotive communication analysis often involves dealing with massive datasets captured via hardware sniffers. This project is a custom-developed **Protocol Analyzer** that processes raw text exports from **Saleae Logic**. It seamlessly bridges hardware-level signal capture with high-level diagnostic interpretation, specifically focusing on **Service Identifiers (SID)** and **Diagnostic Trouble Codes (DTC)**.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <span class="tech-badge bg-qt">Qt / QML (UI)</span>
        <span class="tech-badge bg-python">C/C++, Python (Data Engine)</span>
        <span class="tech-badge bg-uds">UDS / OBD-II</span>
    </div>
</div>

---

## <i class="fa-solid fa-gears" style="color: #e67e22;"></i> System Architecture

The application leverages a hybrid architecture to ensure both visual fluidity and computational efficiency:

### 1. The Frontend: Qt/QML
The user interface is crafted using **QML**, providing a responsive and modern experience. It features real-time data filtering and customized views for different diagnostic sessions, making it easy to navigate through thousands of hex frames.

### 2. The Backend: Python Integration
C/C++ and Python serves as the analytical core. It ingests the Saleae text files and applies complex regex-based parsing to:
- Identify **UDS Request/Response** pairs.
- Categorize traffic based on **SID** (e.g., Session Control, Security Access).
- Extract and decode **DTCs** from the data stream.

---

## <i class="fa-solid fa-magnifying-glass-chart" style="color: #27ae60;"></i> Workflow: From Waves to Insights

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/salea.PNG" title="Saleae Logic Capture" class="img-fluid rounded z-depth-1 shadow-lg" style="width: 100%; height: 250px; object-fit: cover;" %}
    </div>
</div>
<div class="caption">
    <strong>Phase 1:</strong> Raw digital signal acquisition and initial protocol decoding using Saleae Logic hardware.
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/software_monitoring.PNG" title="Qt/QML Monitoring Tool" class="img-fluid rounded z-depth-1 shadow-lg" style="width: 100%; height: 350px; object-fit: cover;" %}
    </div>
</div>
<div class="caption">
    <strong>Phase 2:</strong> The custom monitoring tool in action, visualizing parsed diagnostic data through its Qt-based interface.
</div>

---

## <i class="fa-solid fa-layer-group" style="color: #8e44ad;"></i> Technical Highlights

* <i class="fa-solid fa-code-compare text-success"></i> **Cross-Technology Integration:** Utilizing **PySide/PyQt** to connect Python's data processing power with QML's UI flexibility.
* <i class="fa-solid fa-filter text-primary"></i> **Dynamic Filtering:** Users can filter logs by specific ECU IDs or Diagnostic SIDs in real-time.
* <i class="fa-solid fa-file-code text-warning"></i> **Automated DTC Mapping:** Built-in dictionary to resolve hex values into standard automotive fault descriptions.

<div class="alert alert-info">
  <i class="fa-solid fa-microchip"></i> <strong>Optimized for Efficiency:</strong> The Python engine is optimized to handle large CSV/Text exports (up to 1GB) without compromising the UI's responsiveness.
</div>

---