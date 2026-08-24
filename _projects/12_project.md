---
layout: page
title: GPU-Accelerated Graph Analytics for Irregular Security Graphs
description: <i class="fa-solid fa-microchip"></i> A reproducible empirical study of GPU acceleration strategies (BFS, PageRank, WCC, Louvain) on skewed, power-law graphs — from a sequential C++ baseline through OpenMP to CUDA.
img: assets/img/gpugraph/scaling.png
importance: 12
category: work
published: true
---

<style>
  .tech-badge {
    display: inline-block; padding: 2px 10px; margin: 2px; border-radius: 15px;
    font-size: 0.85em; font-weight: bold; color: white;
  }
  .bg-cpp { background-color: #00599C; }
  .bg-cuda { background-color: #76b900; }
  .bg-hpc { background-color: #c0392b; }
  .bg-algo { background-color: #34495e; }
  .bg-sec { background-color: #2c3e50; }
</style>

## <i class="fa-solid fa-graduation-cap" style="color: #3498db;"></i> Project Background

Many problems in my field produce graphs with the *same* difficult shape: **power-law, highly imbalanced degree distributions.** A binary's call-graph has a handful of hot helper functions called from everywhere while most functions are called once; a network-flow graph for intrusion detection has a few scanning hosts touching thousands of peers while most hosts touch a handful. On a GPU, this imbalance is exactly what wrecks performance — one thread grinds through a million-edge hub while its 31 warp-mates sit idle.

This project is a **reproducible empirical study** — not a claim of a new algorithm — of how the standard GPU-acceleration techniques perform on these irregular security graphs, implementing four classic graph algorithms at every optimization tier so each speedup is attributable to a *specific* technique.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <span class="tech-badge bg-cpp">C++17</span>
        <span class="tech-badge bg-cuda">CUDA</span>
        <span class="tech-badge bg-hpc">OpenMP / HPC</span>
        <span class="tech-badge bg-algo">Graph Algorithms</span>
        <span class="tech-badge bg-sec">Binary / Network Security</span>
    </div>
</div>

<p class="mt-3">
  <i class="fa-brands fa-github"></i>
  <strong>Code:</strong>
  <a href="https://github.com/Emad-Mahmodi/gpu-graph-study">github.com/Emad-Mahmodi/gpu-graph-study</a>
</p>

---

## <i class="fa-solid fa-layer-group" style="color: #e67e22;"></i> Algorithms × Optimization Tiers

Every algorithm is implemented at each tier, so a speedup can be traced to one technique rather than a rewrite. The sequential tier doubles as a **correctness oracle** — every faster tier is diffed against it.

| Algorithm | T0 · Seq C++ | T1 · OpenMP | T2 · Naïve CUDA | T3 · Optimized | T4 · Advanced |
|---|:--:|:--:|:--:|:--:|:--:|
| BFS | ✅ | ✅ | 🚧 | ⬜ | ⬜ |
| PageRank | ✅ | ✅ | ⬜ | ⬜ | ⬜ |
| Connected Components | ✅ | ✅ | ⬜ | ⬜ | ⬜ |
| Community Detection (Louvain) | ✅ | ⬜ | ⬜ | ⬜ | ⬜ |

* **T0 Sequential** — the correctness oracle.
* **T1 OpenMP** — multi-core baseline (GPU speedups are reported against *this*, never a single core).
* **T2–T3 CUDA** — naïve one-thread-per-vertex, then shared-memory frontiers, memory coalescing, warp primitives.
* **T4 Advanced** — degree-bucketed load balancing (thread/warp/block per vertex), documented as an *established* technique (Gunrock/Merrill), not a novel contribution.

---

## <i class="fa-solid fa-chart-line" style="color: #e74c3c;"></i> Results — OpenMP Thread Scaling

Speedup of the OpenMP tier over the sequential baseline on a power-law RMAT graph (2²⁰ vertices, ~16.8M edges), measured on an Intel Core i7-11850H (8 physical cores / 16 threads), median of 10 runs.

<div class="row mt-4">
    <div class="col-sm-12 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/gpugraph/scaling.png" title="OpenMP thread-scaling on a power-law graph" class="img-fluid rounded z-depth-1 shadow-lg" style="width: 100%; height: auto;" %}
    </div>
</div>
<div class="caption">
    <strong>Figure 1:</strong> Thread scaling per algorithm. WCC scales best (6×, no per-vertex synchronization); BFS scales worst (2.5×, atomic frontier + level barriers); the 8→16 plateau reflects the CPU's 8 physical cores. These honest differences motivate the GPU tiers.
</div>

What the numbers say — and *why* is the point of the study:

* **WCC scales best (6×)** — label propagation needs no synchronization, so it is bandwidth-bound only.
* **BFS scales worst (2.5×)** — the atomic frontier claim and the per-level barrier serialize work; this is precisely the imbalance the GPU tiers are built to attack.
* **8→16 threads barely moves** — the CPU has 8 physical cores; the rest is hyper-threading. Textbook, and exactly why a GPU is the next step.

---

## <i class="fa-solid fa-diagram-project" style="color: #16a085;"></i> Domain Bridge — Real Binary Graphs

The analytics engine is **domain-agnostic**: it consumes a graph in CSR form and never parses a binary or a packet. A companion extractor turns a real symbol-bearing binary into a call-graph (functions → vertices, `call` instructions → edges) that the same engine analyzes — so one codebase spans both binary-analysis and network-security graphs. This ties the systems/HPC work directly to my reverse-engineering background.

---

## <i class="fa-solid fa-flask" style="color: #9b59b6;"></i> Engineering & Reproducibility

Built in raw C++/CUDA with CMake (MSVC and GCC), the project ships a correctness test suite (CI-gated), a deterministic RMAT generator, a median-of-N timing harness, and scripts that regenerate every figure. GPU speedups are reported against the OpenMP baseline, as a geometric mean across datasets, with host↔device transfer separated from kernel time — the reproducibility protocol drawn from recent GPU-graph survey work.

<p class="mt-3">
  Full methodology, roadmap, and results:
  <a href="https://github.com/Emad-Mahmodi/gpu-graph-study">github.com/Emad-Mahmodi/gpu-graph-study</a>
</p>
