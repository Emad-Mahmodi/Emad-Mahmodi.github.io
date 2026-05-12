---
# published: false
layout: page
title:  Security Auditing & Data Extraction of Launch Diagnostic Systems
description: <i class="fa-brands fa-bluetooth-b"></i> Investigating vulnerabilities in Android-based OBD2 scanners and intercepting Bluetooth VCI communications.
img: assets/img/launch.jpg
importance: 5
category: work



---
layout: page
title: Security Auditing & Data Extraction of Launch Diagnostic Systems
description: <i class="fa-brands fa-bluetooth-b"></i> Investigating vulnerabilities in Android-based OBD2 scanners and intercepting Bluetooth VCI communications.
img: assets/img/launch_project.png
importance: 3
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
  .bg-android { background-color: #3ddc84; color: #000; }
  .bg-bt { background-color: #0082fc; }
  .bg-ecu { background-color: #f1c40f; color: #000; }
</style>

## <i class="fa-solid fa-microchip" style="color: #3ddc84;"></i> Project Overview

This project involves a deep-dive security assessment of **Launch (ThinkCar)** diagnostic ecosystems. These devices consist of an Android-based application communicating with a Smart VCI (Vehicle Communication Interface) via Bluetooth. The research focuses on reverse engineering the communication protocol and extracting sensitive ECU data.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <span class="tech-badge bg-android">Android Reversing</span>
        <span class="tech-badge bg-bt">Bluetooth Low Energy (BLE)</span>
        <span class="tech-badge bg-ecu">OBD-II Protocols</span>
    </div>
</div>

---

## <i class="fa-solid fa-shield-virus" style="color: #e74c3c;"></i> Attack Surface Analysis

The analysis is divided into three primary vectors to identify potential vulnerabilities:

### 1. Android Application Reverse Engineering
Most Launch tools run on a modified Android environment. By decompiling the **APK** files, I analyzed how the software handles firmware updates for the VCI and how it stores decrypted diagnostic databases.
* **Tools Used:** JADX-GUI, Frida.
* **Focus:** Logic flaws in the subscription verification and database decryption keys.

### 2. Bluetooth VCI Sniffing & Interception
The communication between the tablet and the OBD2 connector is a critical point. Using **Ubertooth One** and **Wireshark**, I captured the traffic to identify how diagnostic commands (UDS/KWP2000) are encapsulated in Bluetooth packets.

<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/launch_bluetooth.png" title="Bluetooth Packet Capture" class="img-fluid rounded z-depth-1 shadow-lg" %}
    </div>
</div>
<div class="caption">
    Analyzing Bluetooth HCI logs to intercept raw OBD-II commands sent to the vehicle.
</div>

### 3. Data Extraction & Firmware Analysis
Diagnostic tools often download "Car Brands" packages. This project explores the extraction of these packages to understand how the tool interprets specialized ECU responses that are not part of standard OBD-II.

---

## <i class="fa-solid fa-flask" style="color: #9b59b6;"></i> Key Research Findings

<div class="alert alert-info">
  <strong><i class="fa-solid fa-triangle-exclamation"></i> Discovery:</strong> 
  Many BLE-based diagnostic connectors lack strong pairing encryption, potentially allowing an unauthorized device to intercept vehicle data within range.
</div>

| Vulnerability Area | Risk Level | Impact |
| :--- | :--- | :--- |
| **BLE Pairing** | High | Unauthorized data sniffing |
| **Local Database** | Medium | Reverse engineering of car logic |
| **APK Security** | Medium | Bypassing license restrictions |

---

## <i class="fa-solid fa-diagram-next" style="color: #f39c12;"></i> Workflow & Methodology

1.  **Reconnaissance:** Mapping the device's hardware and identifying open ports.
2.  **Traffic Analysis:** Monitoring the handshake between the VCI and the ECU.
3.  **Exploitation:** Attempting to inject custom diagnostic PIDs to extract non-standard data.



<div class="row justify-content-sm-center">
    <div class="col-sm-auto mt-3 mt-md-0">
        {% include figure.liquid 
            path="assets/img/launch2.PNG" 
            title="Launch Diagnostic Tool" 
            class="img-fluid rounded z-depth-1 shadow-lg"
            style="width: 50px; height: 100px; object-fit: cover;" %}
    </div>
</div>

<div class="caption">
    The hardware setup: Launch VCI connected to an ECU bench for controlled testing.
</div>

---