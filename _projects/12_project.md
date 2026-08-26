---
layout: page
title: GPU-Accelerated Graph Analytics for Irregular Security Graphs
description: <i class="fa-solid fa-microchip"></i> A reproducible empirical study of GPU acceleration strategies (BFS, PageRank, WCC, Louvain) on skewed, power-law graphs — from a sequential C++ baseline through OpenMP to optimized and load-balanced CUDA, with every speedup attributed to one named technique.
img: assets/img/gpugraph/cuda_t4.png
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

Many problems in my field produce graphs with the *same* difficult shape: **power-law, highly imbalanced degree distributions.** A binary's call-graph has a handful of hot helper functions called from everywhere while most functions are called once; a network-flow graph for intrusion detection has a few scanning hosts touching thousands of peers while most hosts touch a handful. On a GPU, this imbalance is exactly what wrecks performance — one thread grinds through a hub with tens of thousands of edges while its 31 warp-mates sit idle.

This project is a **reproducible empirical study** — not a claim of a new algorithm — of how the standard GPU-acceleration techniques perform on these irregular security graphs. Four classic graph algorithms are implemented at every optimization tier, so each speedup is attributable to a *specific* technique, and every GPU number is reported against **both** a single CPU core **and** a full 8-core CPU — never just one core.

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

Every algorithm is implemented at each tier, so a speedup can be traced to one technique rather than a rewrite. The sequential tier doubles as a **correctness oracle** — every faster tier (including all GPU kernels, on a 254-level high-diameter case) is diffed against it in CI.

| Algorithm | T0 · Seq C++ | T1 · OpenMP | T2 · Naïve CUDA | T3 · Optimized | T4 · Load-balanced |
|---|:--:|:--:|:--:|:--:|:--:|
| BFS | ✅ | ✅ | ✅ | ✅ | ⬜ |
| PageRank | ✅ | ✅ | ✅ | ✅ | ✅ |
| Connected Components (WCC) | ✅ | ✅ | ✅ | ✅ | ⬜ |
| Community Detection (Louvain) | ✅ | ⬜ | ⬜ | ⬜ | — |

* **T0 Sequential** — the correctness oracle.
* **T1 OpenMP** — multi-core CPU baseline (8 threads).
* **T2 Naïve CUDA** — one thread per vertex, CSR in global memory: the honest starting point.
* **T3 Optimized CUDA** — block-level parallel reductions, precomputed inverse degree, frontier / active-set traversal.
* **T4 Load-balanced** — degree-bucketed scheduling (thread / warp / block per vertex), documented as an *established* technique (Gunrock/Merrill), not a novel contribution.

---

## <i class="fa-solid fa-gauge-high" style="color: #e74c3c;"></i> Headline Result — the PageRank optimization ladder

The clearest story in the study. Naïve GPU PageRank *loses* — its two per-iteration reductions are single-address atomics that all N threads fight over. Each optimization removes one named bottleneck, and the speedups compound. Measured on an **NVIDIA T600** (entry-level, 4 GB, Turing) vs an **Intel i7-11850H** (8-core), on a power-law RMAT graph (2²⁰ vertices, 33.5M edges, max degree 138,626), median of 10 runs.

<div class="row mt-4">
    <div class="col-sm-12 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/gpugraph/cuda_t4.png" title="PageRank optimization ladder T0–T4" class="img-fluid rounded z-depth-1 shadow-lg" style="width: 100%; height: auto;" %}
    </div>
</div>
<div class="caption">
    <strong>Figure 1:</strong> PageRank across tiers. Naïve CUDA (T2) loses to the CPU (0.48×). <strong>T3 (parallel reduction)</strong> makes it 4.0× faster than naïve; <strong>T4 (degree-bucketing)</strong> takes it to 5.16× over naïve and 2.46× over a single core — closing to within 15% of the 8-core CPU (dashed line). Every result is bit-for-bit / within-tolerance identical to the oracle.
</div>

| PageRank tier | Median time | vs 1 core | vs naïve GPU |
|---|--:|:--:|:--:|
| CPU 1-core (T0) | 1863.8 ms | 1.00× | — |
| **CPU 8-core (T1)** | **662.2 ms** | 2.81× | — |
| Naïve CUDA (T2) | 3917.5 ms | 0.48× | 1.00× |
| Optimized CUDA (T3) | 978.4 ms | 1.90× | 4.00× |
| **Load-balanced CUDA (T4)** | **758.7 ms** | **2.46×** | **5.16×** |

Two techniques, each attributable: **T3 = parallel reduction** (fixes the write/reduce), **T4 = degree-bucketed scheduling** (fixes the read/gather). The third bucket level — a whole *block* per extreme hub — was implemented and measured to give **no gain** on this graph (a documented point of diminishing returns), because RMAT's power-law tail holds too few very-high-degree vertices for it to matter.

---

## <i class="fa-solid fa-code-branch" style="color: #16a085;"></i> Key Finding — the optimal strategy depends on graph diameter

For traversal (BFS, WCC), the T3 "only launch active vertices" optimizations (frontier worklist, active-set label propagation) are **not** unconditional wins. Whether they pay off depends on the graph's diameter — so I measured both regimes: a shallow power-law RMAT graph (BFS depth 4) and a high-diameter 2D grid (depth 2046).

<div class="row mt-4">
    <div class="col-sm-12 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/gpugraph/crossover.png" title="Diameter crossover for BFS and WCC" class="img-fluid rounded z-depth-1 shadow-lg" style="width: 100%; height: auto;" %}
    </div>
</div>
<div class="caption">
    <strong>Figure 2:</strong> The frontier/active-set optimization <em>loses</em> on the shallow graph (its bookkeeping is pure overhead when the frontier is nearly all of N) but <em>wins 2.4×</em> on the high-diameter grid. On high-diameter graphs the GPU also becomes launch-bound — thousands of kernel launches over tiny per-level work — and the CPU wins outright. Same code, opposite verdict, decided by diameter.
</div>

---

## <i class="fa-solid fa-scale-balanced" style="color: #9b59b6;"></i> An honest bottom line

On *this* hardware — an entry-level 4 GB T600 against a strong 8-core mobile CPU — the **multi-core CPU is the sensible default on these mid-size graphs**; the GPU optimizations are large and real *as techniques* but the T600 does not overtake the CPU at this scale. That is exactly the kind of candid, hardware-grounded result a tiered study exists to produce, rather than a single flattering "GPU vs one core" number. The transferable contributions are the technique attribution (each speedup tied to one named optimization), the diameter-crossover finding, and the launch-bound characterization — all reproducible from committed CSVs and figure scripts.

---

## <i class="fa-solid fa-diagram-project" style="color: #2980b9;"></i> Domain Bridge — Real Binary Graphs

The analytics engine is **domain-agnostic**: it consumes a graph in CSR form and never parses a binary or a packet. A companion extractor turns a real symbol-bearing binary into a call-graph (functions → vertices, `call` instructions → edges) that the same engine analyzes — so one codebase spans both binary-analysis and network-security graphs, tying the systems/HPC work directly to my reverse-engineering background.

---

## <i class="fa-solid fa-flask" style="color: #f39c12;"></i> Engineering & Reproducibility

Built in raw C++/CUDA with CMake (MSVC and GCC, CPU-only builds fall back gracefully with no GPU), the project ships a CI-gated correctness suite (GPU kernels validated against the sequential oracle, including a high-diameter case), a deterministic RMAT generator, a 2D-grid generator for the diameter study, a median-of-N timing harness, committed result CSVs, and Python scripts that regenerate every figure from one dataset. GPU timings are end-to-end and reported against both CPU baselines.

<p class="mt-3">
  Full methodology, tier-by-tier code, roadmap, and results:
  <a href="https://github.com/Emad-Mahmodi/gpu-graph-study">github.com/Emad-Mahmodi/gpu-graph-study</a>
</p>
