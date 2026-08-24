---
layout: page
title: Cross-Domain Generalization of Anomaly Detection on Network-Intrusion Data
description: <i class="fa-solid fa-shield-halved"></i> A PyTorch reimplementation of my drift-aware, uncertainty-weighted online ensemble (ESWA 2020), evaluated under real distribution shift — trained on one attack distribution, tested on another.
img: assets/img/idsgen/generalization.png
importance: 13
category: work
published: true
---

<style>
  .tech-badge {
    display: inline-block; padding: 2px 10px; margin: 2px; border-radius: 15px;
    font-size: 0.85em; font-weight: bold; color: white;
  }
  .bg-ml { background-color: #ee4c2c; }
  .bg-sec { background-color: #2c3e50; }
  .bg-gen { background-color: #8e44ad; }
  .bg-stream { background-color: #16a085; }
</style>

## <i class="fa-solid fa-graduation-cap" style="color: #3498db;"></i> Project Background

An intrusion detector that scores beautifully on its own test set can collapse the moment the traffic shifts — new attack types, a different network, a different population. The number that matters is therefore not in-domain accuracy but the **generalization gap**: how much performance drops under distribution shift. This is the same failure mode a clinical-AI model faces moving between patient populations — here posed on network-security data, where it can be measured on a real, canonical benchmark.

This project reimplements the drift-aware method from my *Expert Systems with Applications* (2020) paper in PyTorch and evaluates it the honest way: **external validation across attack distributions.**

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <span class="tech-badge bg-ml">PyTorch</span>
        <span class="tech-badge bg-stream">Online / Streaming ML</span>
        <span class="tech-badge bg-gen">Domain Shift / Generalization</span>
        <span class="tech-badge bg-sec">Intrusion Detection</span>
    </div>
</div>

<p class="mt-3">
  <i class="fa-brands fa-github"></i>
  <strong>Code:</strong>
  <a href="https://github.com/Emad-Mahmodi/adaptive-ids-generalization">github.com/Emad-Mahmodi/adaptive-ids-generalization</a>
</p>

---

## <i class="fa-solid fa-gears" style="color: #8e44ad;"></i> Method — Uncertainty-Weighted Online Ensemble

Several online linear learners (Perceptron, Passive-Aggressive I/II, a Confidence-Weighted learner) process the traffic stream one sample at a time. Their predictions are fused with weights **inversely proportional to each learner's prediction-error variance** — a learner making stable, low-variance errors is trusted; an erratic one is automatically down-weighted. This is what makes the method *drift-aware* without any explicit drift detector, and it is the PyTorch port of the paper's original fusion contribution.

---

## <i class="fa-solid fa-chart-line" style="color: #e74c3c;"></i> Results — Generalization Under Distribution Shift

NSL-KDD ships two official splits, `KDDTrain+` and `KDDTest+`, that form a built-in distribution shift: the test set deliberately contains attack types under-represented or absent in training. Training on one and testing on the other is genuine external validation, not a random split.

<div class="row mt-4">
    <div class="col-sm-12 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/idsgen/generalization.png" title="Generalization gap on NSL-KDD" class="img-fluid rounded z-depth-1 shadow-lg" style="width: 100%; height: auto;" %}
    </div>
</div>
<div class="caption">
    <strong>Figure 1:</strong> Every metric drops from in-domain (held-out KDDTrain+) to cross-domain (shifted KDDTest+). F1 falls from 0.97 to 0.76 — a 21-point generalization gap. An in-domain number alone would badly overstate real-world performance.
</div>

The headline finding: a detector that reads as near-perfect on its own distribution (**F1 0.97**) drops to **F1 0.76** under a realistic attack-distribution shift. Reporting only the in-domain score would be misleading — which is exactly why external validation across distributions is the honest way to evaluate a detector. The uncertainty-weighted ensemble also edges out every individual base learner on the shifted set.

---

## <i class="fa-solid fa-diagram-project" style="color: #16a085;"></i> Why This Framing Matters

The methodology — measuring and reporting the in-domain → cross-domain gap — is domain-transferable. The same question ("does a model trained on one population still work on another?") underlies digital pathology and medical imaging, where models must hold up across hospitals and patient groups. Demonstrating it end-to-end on security data I know well is a concrete, reproducible bridge into that space.

<p class="mt-3">
  Framework, datasets, and full results:
  <a href="https://github.com/Emad-Mahmodi/adaptive-ids-generalization">github.com/Emad-Mahmodi/adaptive-ids-generalization</a>
</p>

---

## <i class="fa-solid fa-scroll" style="color: #95a5a6;"></i> Based On

> **A drift-aware adaptive method based on minimum uncertainty for anomaly detection in social networking**  
> E. Mahmodi, H. Sadoghi Yazdi, A. Ghaemi Bafghi — *Expert Systems with Applications*, 2020  
> [doi:10.1016/j.eswa.2020.113881](https://doi.org/10.1016/j.eswa.2020.113881) · original MATLAB code: [Emad-Mahmodi/AdaptiveLarning](https://github.com/Emad-Mahmodi/AdaptiveLarning)
