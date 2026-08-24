---
layout: page
title: Binary-Analysis LLM Copilot — Studying ReCopilot & a Lightweight RAG Baseline
description: <i class="fa-solid fa-brain"></i> A study of ReCopilot (Chen et al., 2025) — an expert LLM for binary analysis — together with my own independent, lightweight retrieval-augmented (RAG) reimplementation for function-name recovery.
img: assets/img/binrag/pipeline.png
importance: 10
category: work
published: true
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
  .bg-llm   { background-color: #34495e; }
  .bg-rag   { background-color: #8e44ad; }
  .bg-re    { background-color: #2c3e50; }
  .bg-cpt   { background-color: #1a6b4a; }
  .bg-sft   { background-color: #c0392b; }
  .bg-dpo   { background-color: #d35400; }
  .bg-ctx   { background-color: #16a085; }
  .attrib-box {
    border-left: 4px solid #8e44ad;
    background: rgba(142, 68, 173, 0.07);
    padding: 12px 18px;
    border-radius: 6px;
    margin: 18px 0;
  }
</style>

<div class="attrib-box">
<strong>What this page is.</strong> <em>ReCopilot</em> is published research by
<strong>Guoqiang Chen, Huiqi Sun, et&nbsp;al. (QI-ANXIN Technology Research Institute)</strong>
— it is <strong>not my work</strong>. The first half of this page is my study and
summary of their system (the foundation I build on). The second half,
<a href="#my-contribution">My contribution</a>, is an <strong>independent,
lightweight reimplementation I wrote myself</strong> — a retrieval-augmented
(RAG) baseline for binary function-name recovery that borrows ReCopilot's core
<em>idea</em> (feeding an LLM static-analysis context) but uses <strong>no model
training</strong>. Code: <a href="https://github.com/Emad-Mahmodi/binary-rag-copilot">github.com/Emad-Mahmodi/binary-rag-copilot</a>.
All figures in the background section are reproduced from the ReCopilot paper and
are credited as such.
</div>

## <i class="fa-solid fa-graduation-cap" style="color: #3498db;"></i> Motivation

Binary analysis sits at the heart of cybersecurity work — from malware detection to vulnerability discovery — yet it remains one of the most labor-intensive tasks in the field. When source code is compiled and its debug symbols are stripped, all the meaningful context that makes code readable vanishes: function names become `sub_1909`, variables become `a1` and `v3`, and data structures lose their identity. Decompilers like IDA Pro and Ghidra lift machine code back into C-like pseudo-code, but they cannot recover these lost symbols.

This is my day-to-day domain (automotive ECU reverse engineering), so I studied the current state of the art for applying LLMs to it — **ReCopilot** — in depth, and then built my own lightweight baseline to understand how much of the job is achievable *without* the heavy machinery ReCopilot uses.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <span class="tech-badge bg-llm">LLM for Binary Analysis</span>
        <span class="tech-badge bg-rag">RAG (my baseline)</span>
        <span class="tech-badge bg-ctx">Static-Analysis Context</span>
        <span class="tech-badge bg-re">Binary Reverse Engineering</span>
    </div>
</div>

---

# <i class="fa-solid fa-book" style="color: #8e44ad;"></i> Part 1 — Background: How ReCopilot Works (Chen et al., 2025)

*The following summarizes the published ReCopilot system. It is their contribution,
not mine; I include it because my baseline builds on its central idea.*

## <i class="fa-solid fa-layer-group" style="color: #e67e22;"></i> The Core Problem: Semantic Gap in Decompiled Code

When a binary is stripped, debug symbols are removed. What remains is decompiled pseudo-code full of placeholder names. A real AES encryption function that reads clearly in source as `AES_CBC_encrypt_buffer` — with named structs (`AES_ctx`) and typed fields (`RoundKey`, `Iv`) — becomes `sub_1909` with arguments `a1`, `a2`, `a3` after stripping and decompilation, leaving analysts to reconstruct intent from offsets and arithmetic alone.

<div class="row mt-4">
    <div class="col-sm-12 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/pr_llm2/5.PNG" title="Pseudo Code vs Source Code" class="img-fluid rounded z-depth-1 shadow-lg" style="width: 100%; height: auto;" %}
    </div>
</div>
<div class="caption">
    <strong>Figure 1 (from the ReCopilot paper, Chen et al., 2025):</strong> The same AES CBC encryption function — decompiled pseudo-code (left) vs. original source (right). All symbolic information is erased after stripping.
</div>

ReCopilot targets three levels of representation: clean **source code**, **decompiled pseudo-code with symbols**, and the hard case — **stripped pseudo-code** with only generic placeholders and raw offsets.

## <i class="fa-solid fa-database" style="color: #27ae60;"></i> Dataset Construction

A major part of the ReCopilot work is a large-scale dataset built from scratch, since no adequate public one existed. Three pipelines collect binary functions at scale: **compile-from-scratch** (open-source packages compiled with controlled flags, functions extracted via DWARF), **off-the-shelf artifacts** (release + debug + source packages from Ubuntu/Debian), and **CompileAgent** (an LLM-driven build agent for ad-hoc GitHub projects). In total, **over 100 million binary functions from 11,000+ projects**, then sanitized and MinHash-deduplicated.

Each pretraining sample pairs three views of the same function — stripped pseudo-code, symbolized pseudo-code, and source with a natural-language comment — with the segments **randomly shuffled** to force bidirectional learning. The final pretraining corpus is **36 billion tokens**.

<div class="row mt-4 justify-content-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/pr_llm2/7.PNG" title="Pretraining Data Format and Inner-Shuffling" class="img-fluid rounded z-depth-1 shadow-lg" style="width: 100%; height: auto;" %}
    </div>
</div>
<div class="caption">
    <strong>Figure 2 (from the ReCopilot paper):</strong> Each pretraining sample contains three representations of the same function; inner-shuffling randomizes segment order to enforce bidirectional learning.
</div>

For task fine-tuning, ReCopilot uses a **generator–discriminator framework** to synthesize Chain-of-Thought examples: a Generator writes reasoning *without* seeing ground truth; a Discriminator judges it for correctness and consistency. Passing examples become SFT data; failed-then-fixed ones become DPO chosen/rejected pairs. The final SFT set covers **14 binary-analysis tasks** (name recovery, signature/variable/struct recovery, algorithm identification, summarization, decompilation improvement, and more).

<div class="row mt-4">
    <div class="col-sm-12 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/pr_llm2/9.PNG" title="Generator-Discriminator Framework" class="img-fluid rounded z-depth-1 shadow-lg" style="width: 100%; height: auto;" %}
    </div>
</div>
<div class="caption">
    <strong>Figure 3 (from the ReCopilot paper):</strong> The generator–discriminator loop for automated CoT dataset construction; failed generations are recycled as DPO pairs.
</div>

## <i class="fa-solid fa-gears" style="color: #8e44ad;"></i> Training Strategy

ReCopilot is trained in three stages on top of **Qwen2.5-Coder-7B**: **Continued Pretraining (CPT)** to inject binary-domain knowledge, **Supervised Fine-Tuning (SFT)** to follow structured instructions and emit JSON with CoT reasoning, and **Direct Preference Optimization (DPO)** to improve format compliance and reasoning coherence.

<div class="row mt-4">
    <div class="col-sm-12 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/pr_llm2/6.PNG" title="Training Pipeline" class="img-fluid rounded z-depth-1 shadow-lg" style="width: 100%; height: auto;" %}
    </div>
</div>
<div class="caption">
    <strong>Figure 4 (from the ReCopilot paper):</strong> The three-stage training pipeline — CPT → SFT → DPO.
</div>

## <i class="fa-solid fa-diagram-project" style="color: #16a085;"></i> Context Enhancement

ReCopilot's most transferable idea — and the one my baseline borrows — is that a single function rarely tells the whole story, so it augments the model with **static-analysis context** at inference time: a bidirectional call-graph traversal selects informative caller/callee functions (ranked by an informativeness score over names, string density, and callee names), and a custom data-flow engine injects alias annotations directly into the prompt.

<div class="row mt-4 justify-content-center">
    <div class="col-sm-9 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/pr_llm2/2.PNG" title="Informative Score Formula" class="img-fluid rounded z-depth-1 shadow-lg" style="width: 100%; height: auto;" %}
    </div>
</div>
<div class="caption">
    <strong>Figure 5 (from the ReCopilot paper):</strong> The informativeness score S(f) ranking context functions by name, string-literal density, and named-callee ratio.
</div>

## <i class="fa-solid fa-chart-line" style="color: #e74c3c;"></i> ReCopilot's Reported Results

On a from-scratch, full-binary benchmark (six tasks; leakage-free private test data), ReCopilot reportedly **outperforms baseline tools by ~13% on average**, and its 7B expert model reaches performance comparable to far larger general models (e.g. DeepSeek-V3, 671B) — supporting the thesis that **domain-specific training beats raw scale** for this task. Ablations credit CPT, DPO (format compliance), and data-flow context (struct tasks) as the main contributors.

---

<a name="my-contribution"></a>

# <i class="fa-solid fa-wrench" style="color: #2ecc71;"></i> Part 2 — My Contribution: A Lightweight RAG Baseline

Everything below is my own work: [**github.com/Emad-Mahmodi/binary-rag-copilot**](https://github.com/Emad-Mahmodi/binary-rag-copilot).

ReCopilot answers "how good can an LLM get at binary analysis if you *train* one properly?" I wanted the complementary question:

> **How much of function-name recovery is achievable with the context idea alone — no training, just retrieval + prompting of an off-the-shelf model?**

That question needs a baseline, so I built one.

## <i class="fa-solid fa-sitemap" style="color: #3498db;"></i> Method

```
 symbol-bearing binary
        │  objdump / llvm-objdump  (tool-independent PLT→GOT→symbol resolution)
        ▼
 per-function context ── callees ── callers ── disassembly     ← RETRIEVAL
        ▼
 hide the real name → build a context-augmented prompt          ← AUGMENT
        ▼
 LLM backend proposes a name  (mock | OpenAI-compatible | Ollama)   ← GENERATE
        ▼
 score vs. the real name — exact / token-F1 / semantic-hit      ← EVALUATE
```

Ground truth is free: a **symbol-bearing** binary already knows each function's real name. I hide it, ask the pipeline to recover it from context, and score the guess. The call-context extraction reuses the call-graph tooling from my [GPU graph-analytics project](https://github.com/Emad-Mahmodi/gpu-graph-study), and the disassembly parser resolves PLT calls to real symbols across **both** GNU `objdump` and LLVM `llvm-objdump`.

<div class="row mt-4">
    <div class="col-sm-12 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/binrag/pipeline.png" title="My RAG pipeline for function-name recovery" class="img-fluid rounded z-depth-1 shadow-lg" style="width: 100%; height: auto;" %}
    </div>
</div>
<div class="caption">
    <strong>Figure 6 — my own work:</strong> The four-stage pipeline in
    <a href="https://github.com/Emad-Mahmodi/binary-rag-copilot">binary-rag-copilot</a>.
    No model training: retrieve call context → augment a prompt → let a pluggable LLM
    backend propose a name → score against the real symbol. Every stage maps to a
    module in the repo (<code>src/rag/</code>, <code>src/llm/</code>, <code>src/eval/</code>).
</div>

## <i class="fa-solid fa-scale-balanced" style="color: #e67e22;"></i> How it differs from ReCopilot

| | **ReCopilot** (Chen et al.) | **My baseline** |
|---|---|---|
| Core idea | LLM + static-analysis context | same idea |
| How the model is built | CPT + SFT + DPO on **36B tokens** | **no training** — retrieval + prompting |
| Compute to reproduce | multi-GPU, weeks, private data | a laptop; default backend needs no GPU |
| Tasks | 14 | function-name recovery (extensible) |
| Runs offline, no API key | — | yes (mock backend) |
| Ceiling on quality | high (purpose-built model) | bounded by the general LLM plugged in |

**Honest positioning:** ReCopilot is the stronger *system*; mine is the cheap, fully-reproducible *baseline* that isolates how far context + prompting alone can go. Its pipeline can even serve ReCopilot's own base model (Qwen2.5-Coder via Ollama) to measure exactly that.

## <i class="fa-solid fa-flask" style="color: #9b59b6;"></i> Results (proof-of-concept)

Five methods scored on the same functions of a small symbol-bearing demo binary. The default backend is a transparent **call-pattern heuristic** — not an LLM — serving as an honest *lower bound* any real model should beat:

| Method | token-F1 | semantic-hit |
|---|:--:|:--:|
| majority-callee | 0.00 | 0.00 |
| **heuristic (calls)** | **0.36** | **0.60** |
| nearest-neighbor (RAG retrieval) | 0.00 | 0.00 |
| **llm (mock lower-bound)** | **0.36** | **0.60** |
| llm (Qwen2.5-Coder, Ollama) | *plug in to fill* | *plug in to fill* |

<div class="row mt-4">
    <div class="col-sm-12 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/binrag/comparison.png" title="Method comparison on the demo binaries" class="img-fluid rounded z-depth-1 shadow-lg" style="width: 100%; height: auto;" %}
    </div>
</div>
<div class="caption">
    <strong>Figure 7 — my own work:</strong> Head-to-head of the naming methods,
    produced by <code>experiments/compare_methods.py</code> in
    <a href="https://github.com/Emad-Mahmodi/binary-rag-copilot">binary-rag-copilot</a>.
    The call-pattern heuristic recovers 60% of names semantically; the retrieval
    baseline collapses under domain mismatch — motivating the LLM row.
</div>

Example recoveries from call context alone: `read_whole_file → read_file`, `print_banner → print_message`, `compare_names → compare_strings`. Run it yourself: `python experiments/run_name_recovery.py --binary examples/demo` — the full command set is in the [repo README](https://github.com/Emad-Mahmodi/binary-rag-copilot#quick-start).

<div class="attrib-box">
<strong>Deliberately honest caveat.</strong> These are tiny demo binaries (≈5 functions),
so the numbers illustrate the pipeline and the differences between methods — they
are <em>not</em> a benchmark verdict, and they are <em>not</em> comparable to
ReCopilot's large-scale evaluation. The interesting finding is qualitative: the
retrieval baseline fails when the corpus doesn't cover the test binary's call
patterns, which is exactly why a model that <em>reads</em> the disassembly is
needed — motivating the LLM row.
</div>

## <i class="fa-solid fa-person-digging" style="color: #f39c12;"></i> Work in Progress

Three extensions are actively underway to turn this proof-of-concept into a meaningful study:

<div class="row mt-2">
    <div class="col-sm mt-3 mt-md-0">
        <span class="tech-badge" style="background-color:#f39c12;">🚧 In Progress</span>
    </div>
</div>

1. **Plug in a real LLM.** Wiring **Qwen2.5-Coder** (ReCopilot's own base family) through **Ollama on my local NVIDIA T600 GPU**, to fill the LLM row of the comparison table with a genuine number — so the claim becomes concrete: *"context + prompting alone reaches X% semantic accuracy, versus the 60% call-pattern lower bound."*
2. **Add a second task.** Extending the pipeline from function-name recovery to a second ReCopilot task — **variable-type inference** or **one-line function summarization** — moving from one task to two on the same context-retrieval backbone.
3. **Evaluate on a larger corpus.** Running the study over **many symbol-bearing binaries** instead of the demo pair, so the metrics become statistically meaningful rather than illustrative. The harness already accepts multiple corpus/test binaries (`--corpus a b c --test d`).

Progress is tracked in the [repository roadmap](https://github.com/Emad-Mahmodi/binary-rag-copilot#roadmap).

---

## <i class="fa-solid fa-scroll" style="color: #95a5a6;"></i> Attribution & Reference

The system studied in Part 1 is **not my work**. Full credit to its authors:

> **ReCopilot: Reverse Engineering Copilot in Binary Analysis**  
> Guoqiang Chen, Huiqi Sun, Daguang Liu, Zhiqi Wang, Qiang Wang, Bin Yin, Lu Liu, Lingyun Ying  
> QI-ANXIN Technology Research Institute, Beijing, China  
> arXiv:2505.16366 — May 2025 · [project code](https://github.com/XingTuLab/recopilot)

My own independent baseline (Part 2): [**github.com/Emad-Mahmodi/binary-rag-copilot**](https://github.com/Emad-Mahmodi/binary-rag-copilot). All figures above are reproduced from the ReCopilot paper for the purpose of study and are credited accordingly.
