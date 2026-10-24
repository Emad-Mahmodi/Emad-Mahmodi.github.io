---
layout: page
title: Security Assessment of Diagnostic Scanner Ecosystems
description: <i class="fa-brands fa-bluetooth-b"></i> Defensive assessment of the security posture of Android-based OBD2 scanners and their Bluetooth VCI communications.
img: assets/img/ble-hci-capture.png
importance: 5
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

This project is a defensive security assessment of aftermarket Android-based **diagnostic scanner** ecosystems, performed on a bench setup as research and development through lawful, authorised corporate and personal processes. These devices consist of an Android-based application communicating with a Smart VCI (Vehicle Communication Interface) via Bluetooth. The research focuses on understanding the communication protocol and evaluating what vehicle data and proprietary content is exposed, so that weaknesses can be reported and hardened. No credentials, keys, decrypted databases or bypass methods are published.

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
Many such tools run on a modified Android environment. By analysing the **APK** files, I reviewed how the software handles firmware updates for the VCI and how it protects its diagnostic databases.
* **Tools Used:** JADX-GUI, Frida.
* **Focus:** Strength of the protection around the update channel, subscription verification and local diagnostic databases, and how it could be hardened.

### 2. Bluetooth VCI Sniffing & Interception
The communication between the tablet and the OBD2 connector is a critical point. Using **Ubertooth One** and **Wireshark** on test devices, I captured the traffic to identify how diagnostic commands (UDS/KWP2000) are encapsulated in Bluetooth packets.

<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/ble-hci-capture.png" title="Bluetooth Packet Capture" class="img-fluid rounded z-depth-1 shadow-lg" %}
    </div>
</div>
<div class="caption">
    Analyzing Bluetooth HCI logs to observe which OBD-II commands are exposed on the wireless link.
</div>

### 3. Data Extraction & Firmware Analysis
Diagnostic tools often download "Car Brands" packages. This project reviews how these packages are protected in transit and at rest, and how the tool interprets specialized ECU responses that are not part of standard OBD-II.

---

## <i class="fa-solid fa-flask" style="color: #9b59b6;"></i> Key Research Findings

<div class="alert alert-info">
  <strong><i class="fa-solid fa-triangle-exclamation"></i> Discovery:</strong> 
  Some BLE-based diagnostic connectors may lack strong pairing encryption, potentially allowing an unauthorized device to intercept vehicle data within range.
</div>

| Vulnerability Area | Risk Level | Impact |
| :--- | :--- | :--- |
| **BLE Pairing** | High | Eavesdropping on vehicle data (fix: LE Secure Connections pairing) |
| **Local Database** | Medium | Exposure of proprietary diagnostic data (fix: stronger encryption and key handling) |
| **APK Security** | Medium | Weak integrity and license checks (fix: attestation and tamper detection) |

---

## <i class="fa-solid fa-diagram-next" style="color: #f39c12;"></i> Workflow & Methodology

1.  **Reconnaissance:** Mapping the device's hardware and identifying open ports.
2.  **Traffic Analysis:** Monitoring the handshake between the VCI and the ECU.
3.  **Validation:** Checking whether the VCI accepts unauthenticated or malformed diagnostic requests, to recommend access control on the link.



---

<div class="alert alert-warning">
  <i class="fa-solid fa-triangle-exclamation"></i> <strong>Research Use Only.</strong>
  This is research and development carried out through lawful, authorised corporate and personal processes, on bench hardware, for defensive security and education. It is not intended for circumventing licensing or access controls, extracting or redistributing vendor content, or modifying any vehicle without the owner's permission, which may be unlawful (for example under US, EU and Canadian law).
</div>
