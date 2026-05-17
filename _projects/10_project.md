---
layout: page
title: ReCopilot — Reverse Engineering Copilot in Binary Analysis
description: <i class="fa-solid fa-brain"></i> An expert LLM for binary analysis tasks including function name recovery, variable type inference, struct recovery, decompilation, and code summarization.
img: assets/img/pr_llm2/5.PNG
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
</style>

## <i class="fa-solid fa-graduation-cap" style="color: #3498db;"></i> Research & Project Background

Binary analysis sits at the heart of cybersecurity work — from malware detection to vulnerability discovery — yet it remains one of the most labor-intensive tasks in the field. When source code is compiled into a binary and its debug symbols are stripped, all the meaningful context that makes code readable vanishes: function names become `sub_1909`, variables become `a1` and `v3`, and data structures lose their identity entirely. Traditional decompilers like IDA Pro and Ghidra can lift machine code back into C-like pseudo-code, but they cannot recover these lost symbols.

This project presents **ReCopilot**, an expert Large Language Model purpose-built for binary analysis. Rather than prompting a general-purpose LLM with binary code and hoping for the best, ReCopilot is trained from the ground up on domain-specific data — learning the deep mappings between stripped pseudo-code, source code, and natural language. The result is a compact 7B-parameter model that is deployable on a local machine and outperforms much larger general-purpose models on binary analysis tasks.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <span class="tech-badge bg-llm">Expert LLM</span>
        <span class="tech-badge bg-cpt">Continued Pretraining</span>
        <span class="tech-badge bg-sft">Supervised Fine-Tuning</span>
        <span class="tech-badge bg-dpo">DPO Alignment</span>
        <span class="tech-badge bg-ctx">Context Enhancement</span>
        <span class="tech-badge bg-re">Binary Reverse Engineering</span>
    </div>
</div>

---

## <i class="fa-solid fa-layer-group" style="color: #e67e22;"></i> The Core Problem: Semantic Gap in Decompiled Code

When a binary is stripped before release, the debug symbols are removed for reasons ranging from reducing file size to hiding implementation details. What remains is decompiled pseudo-code full of placeholder names. Consider a real-world AES encryption function: in source code it reads clearly as `AES_CBC_encrypt_buffer` with named structs (`AES_ctx`), typed fields (`RoundKey`, `Iv`), and meaningful call sites (`XorWithIv`, `Cipher`). After stripping and decompilation by IDA Pro, the same function becomes `sub_1909` with arguments `a1`, `a2`, `a3` and local variables `v3`, `v7` — leaving analysts to reconstruct intent purely from memory offsets and arithmetic patterns.

<div class="row mt-4">
    <div class="col-sm-12 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/pr_llm2/5.PNG" title="Pseudo Code vs Source Code" class="img-fluid rounded z-depth-1 shadow-lg" style="width: 100%; height: auto;" %}
    </div>
</div>
<div class="caption">
    <strong>Figure 1:</strong> The same AES CBC encryption function — decompiled pseudo-code (left) vs. original source code (right). All symbolic information is erased: function names, struct types, variable names, and developer comments vanish entirely after stripping.
</div>

The three levels of representation this project addresses are:

* **Source Code (Baseline):** Clean, developer-authored logic with full context — function names, comments, type definitions, and struct layouts.
* **Decompiled Pseudo-Code (with symbols):** Output from IDA Pro or Ghidra when debug info is present — readable, though verbose.
* **Stripped Pseudo-Code (the hard case):** The same decompiler output with all symbolic information removed, leaving only generic placeholders and raw offsets.

---

## <i class="fa-solid fa-database" style="color: #27ae60;"></i> Dataset Construction

One of the most significant contributions of this work is a large-scale binary dataset built entirely from scratch, since no adequate public dataset existed for this domain.

### Raw Data Collection

Three parallel pipelines were used to collect binary functions at scale:

**Compile from Scratch** pulls open-source packages from repositories like Arch Linux, compiles them with controlled compiler flags, and extracts binary functions alongside their source counterparts using DWARF debug information. This pipeline alone produced tens of millions of binary functions.

**Off-the-shelf Software Artifacts** harvests release packages, debug symbol packages, and source packages from Ubuntu and Debian repositories — covering a much wider range of real-world compilation environments than any controlled pipeline can replicate.

**CompileAgent** uses an LLM-driven compilation agent to handle ad-hoc projects from GitHub repositories, automatically resolving dependency and build errors that would normally block automated collection.

In total, the raw dataset spans over 100 million binary functions collected from more than 11,000 projects — an order of magnitude larger than prior work in this space. After collection, the dataset undergoes sanitization (removing functions that are too short, too long, thunk/auxiliary, or missing ground truth) and deduplication using MinHash to reduce redundancy from code reuse across projects.

### Pretraining Data Format

Each pretraining sample pairs three representations of the same function: stripped pseudo-code, symbolized pseudo-code, and source code with a natural language comment. To prevent the model from learning a one-directional mapping, the three segments are **randomly shuffled** within each sample — forcing the model to learn genuine bidirectional relationships across all modalities.

<div class="row mt-4 justify-content-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/pr_llm2/7.PNG" title="Pretraining Data Format and Inner-Shuffling" class="img-fluid rounded z-depth-1 shadow-lg" style="width: 100%; height: auto;" %}
    </div>
</div>
<div class="caption">
    <strong>Figure 2:</strong> Each pretraining sample contains three representations of the same function (stripped pseudo-code, symbolized pseudo-code, and source code). Inner-shuffling randomizes segment order to enforce bidirectional learning across all modalities.
</div>

The final pretraining dataset contains 36 billion tokens, with binary-domain data making up the dominant share, supplemented by general code (C/C++/Python/Rust/Go/Shell) and curated natural language sources including Wikipedia, Stack Overflow posts, and security research papers.

### SFT Dataset via Generator-Discriminator Framework

To fine-tune the model on specific binary analysis tasks, a novel **generator-discriminator framework** was developed to automatically synthesize Chain-of-Thought (CoT) training examples. The key challenge: existing reasoning LLMs are not trained on binary code, so distilling their reasoning directly produces incorrect thought processes. And if ground-truth source code is provided in the prompt, any reasoning model simply cites it — making the result useless as training data.

The solution is a two-LLM loop: a **Generator** produces a CoT for each example *without* seeing the ground truth, guided only by expert-written step descriptions per task. A **Discriminator** then judges the CoT for correctness, consistency, helpfulness, and purity. Passing examples become training data. Failed examples, when retried successfully, form chosen/rejected pairs for DPO.

<div class="row mt-4">
    <div class="col-sm-12 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/pr_llm2/9.PNG" title="Generator-Discriminator Framework" class="img-fluid rounded z-depth-1 shadow-lg" style="width: 100%; height: auto;" %}
    </div>
</div>
<div class="caption">
    <strong>Figure 3:</strong> The generator-discriminator loop for automated CoT dataset construction. The dashed loop enables stepwise Super-CoT generation. Failed generations are recycled as DPO chosen/rejected pairs rather than discarded — making the process nearly waste-free.
</div>

A **Super-CoT** variant extends this loop stepwise — generating reasoning one step at a time, each conditioned on all previous steps — producing reasoning chains roughly ten times longer than the standard process. This is enabled by the constrained nature of binary analysis tasks, where each reasoning step can be precisely defined by domain experts.

The final SFT dataset covers **14 distinct binary analysis tasks**, including function name recovery, full function signature recovery, variable and argument recovery, struct recovery, algorithm identification, category identification, code summarization (brief and detailed, in English and Chinese), full function analysis, and decompilation improvement.

---

## <i class="fa-solid fa-gears" style="color: #8e44ad;"></i> Training Strategy

ReCopilot is trained in three sequential stages on top of Qwen2.5-Coder-7B as the base model:

<div class="row mt-4">
    <div class="col-sm-12 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/pr_llm2/6.PNG" title="Training Pipeline" class="img-fluid rounded z-depth-1 shadow-lg" style="width: 100%; height: auto;" %}
    </div>
</div>
<div class="caption">
    <strong>Figure 4:</strong> The three-stage training pipeline — Continued Pretraining (CPT), Supervised Fine-Tuning (SFT), and Direct Preference Optimization (DPO) — producing intermediate checkpoints evaluated on both binary-domain and general benchmarks.
</div>

**Continued Pretraining (CPT)** injects binary domain knowledge into the base model. General-purpose LLMs see almost no decompiled pseudo-code during their original training, and what little exists online typically lacks corresponding source code for alignment. CPT closes this fundamental gap before any task-specific learning begins.

**Supervised Fine-Tuning (SFT)** adapts the pretrained model to follow structured binary analysis instructions and output predictions in a consistent JSON format. The SFT data includes CoT reasoning, enabling the model to think through binary code step-by-step before committing to a prediction — analogous to the test-time scaling behavior of o1 and DeepSeek-R1.

**Direct Preference Optimization (DPO)** uses chosen/rejected pairs harvested during dataset construction to improve format consistency and reasoning coherence. Unlike RLHF, DPO requires no reward model or human labeling. DPO substantially improves the rate of syntactically valid, parseable JSON output — critical for seamless integration into decompilation tools.

### Input-Output Template

Every binary analysis task follows a unified structured prompt template, making the system modular and easily extensible to new tasks without architectural changes:

<div class="row mt-4 justify-content-center">
    <div class="col-sm-6 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/pr_llm2/8.PNG" title="Input-Output Template" class="img-fluid rounded z-depth-1 shadow-lg" style="width: 100%; height: auto;" %}
    </div>
</div>
<div class="caption">
    <strong>Figure 5:</strong> The unified input-output template for all binary analysis tasks in ReCopilot. Red sections form the model input (target function, context functions, call chains, data flow, task tag); purple sections are the model output (chain-of-thought reasoning followed by structured JSON prediction).
</div>

The template accepts: ① the target binary function in pseudo-code, ② context functions from the call graph, ③ call chain relationships, ④ variable data flow annotations, and ⑤ a task tag specifying what to recover. The model outputs ⑥ a step-by-step reasoning process followed by ⑦ a structured JSON prediction.

---

## <i class="fa-solid fa-diagram-project" style="color: #16a085;"></i> Context Enhancement

A single binary function rarely tells the whole story. Understanding what a variable *is* often requires following it through the functions that call it and the functions it calls. ReCopilot addresses this with two static analysis techniques applied at inference time.

### Call Chain Analysis

Starting from the target function, a bidirectional breadth-first search traverses the call graph — identifying both callers and callees up to a configurable depth. Functions closer to the target are placed near the end of the prompt to leverage the model's stronger attention to recent context.

To avoid exceeding the context window, an **informative score** ranks candidate context functions by three signals: whether they carry a meaningful function name, how densely they contain string literals, and whether their callees have meaningful names. Only the top-k most informative functions are included.

<div class="row mt-4 justify-content-center">
    <div class="col-sm-9 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/pr_llm2/2.PNG" title="Informative Score Formula" class="img-fluid rounded z-depth-1 shadow-lg" style="width: 100%; height: auto;" %}
    </div>
</div>
<div class="caption">
    <strong>Figure 6:</strong> The informative score S(f) for ranking context functions. It combines three signals: presence of a meaningful function name N(f), string literal density scaled by β=25, and the average named-function ratio across callees.
</div>

### Data Flow Analysis

For tasks like variable type and struct inference, the model needs to understand how a variable flows through the program — what it is passed to, how it is modified, and what aliases it acquires across function boundaries. Rather than relying on heavy-weight tools like Joern or Semgrep (which have no native support for IDA-style pseudo-code), a custom lightweight data flow engine was implemented directly on the AST of decompiled pseudo-code.

The engine propagates alias relationships using five categories of inference rules:

<div class="row mt-4">
    <div class="col-sm-12 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/pr_llm2/3.PNG" title="Data Flow Inference Rules" class="img-fluid rounded z-depth-1 shadow-lg" style="width: 100%; height: auto;" %}
    </div>
</div>
<div class="caption">
    <strong>Figure 7:</strong> Inference rules for the custom data flow analysis engine — covering definition, expression, assignment, callee call, and caller propagation statements. Rules are written in programming-like notation rather than formal Hoare logic for readability.
</div>

Alias annotations are injected directly into the prompt — e.g., `// alias: __int64 a2 == a1` — so the model can immediately see how a variable in a callee corresponds to the variable being analyzed in the target function, without having to perform the trace itself:

<div class="row mt-4">
    <div class="col-sm-12 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/pr_llm2/1.PNG" title="Context Enhancement Prompt Example" class="img-fluid rounded z-depth-1 shadow-lg" style="width: 100%; height: auto;" %}
    </div>
</div>
<div class="caption">
    <strong>Figure 8:</strong> A complete context-enhanced prompt for argument recovery on <code>sub_1909@a1</code>. Yellow highlights show the alias annotations injected by the data flow engine — informing the model that <code>a2</code> in <code>sub_14F7</code> is a direct alias of the traced variable <code>a1</code>.
</div>

---

## <i class="fa-solid fa-chart-line" style="color: #e74c3c;"></i> Benchmark & Evaluation

### Benchmark Design

A multi-task binary analysis benchmark was constructed from the ground up, covering six evaluation tasks: **function name recovery**, **variable name recovery**, **variable type recovery**, **struct recovery**, **binary code summarization**, and **decompilation quality**.

<div class="row mt-4">
    <div class="col-sm-12 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/pr_llm2/4.PNG" title="Benchmark Pipeline" class="img-fluid rounded z-depth-1 shadow-lg" style="width: 100%; height: auto;" %}
    </div>
</div>
<div class="caption">
    <strong>Figure 9:</strong> The end-to-end benchmark pipeline. Full binary files are processed through IDA Pro's runtime environment; predictions are extracted from <code>.idb</code> files and evaluated by six independent evaluators — one per task.
</div>

Unlike prior benchmarks that operate at function level, this benchmark takes full binary files as input — providing realistic context to all evaluated tools. The test set spans cryptography, networking, multimedia, compression, databases, and system utilities, compiled for both Linux and Windows. All test data originates from private compilation environments to eliminate training leakage.

Evaluation metrics are matched carefully to each task: ROUGE for name recovery, Precision/Recall/F1 for struct boundary prediction, LLM-as-judge (semantic coverage, accuracy, misleading content, readability) for summarization, and CodeBLEU for decompilation quality.

### Key Results

ReCopilot outperforms all baseline tools across nearly every task — including tools backed by much larger general-purpose LLMs — with an average lead of **13%** over the next-best method. It is the only LLM-based tool that supports variable type recovery and struct recovery; prior tools left both tasks entirely unaddressed.

On the model comparison, ReCopilot's 7B expert model achieves performance comparable to DeepSeek-V3 (671B) and holds a meaningful advantage over DeepSeek-R1 on most tasks, despite being nearly 100× smaller. This validates the core thesis: **domain-specific training yields far greater efficiency than simply scaling model size**.

The ablation studies confirm each component's contribution:
* **CPT** is essential — skipping it measurably weakens performance, particularly on tasks requiring deep semantic inference.
* **DPO** dramatically improves format compliance, increasing the rate of valid JSON output and enabling reliable tool integration.
* **Data flow analysis** provides the largest single boost for struct-related tasks, where variable propagation context is critical for inferring memory layouts.
* **Super-CoT**, while promising, is currently limited by dataset scale; expanding it is a clear priority for future iterations.

One expected trade-off: domain-specific CPT causes some degradation on general benchmarks. The model trained without CPT shows much smaller general-capability loss while still achieving strong binary analysis performance — a useful operating point for deployment scenarios where general capability must be preserved.

---

## <i class="fa-solid fa-lightbulb" style="color: #f39c12;"></i> Discussion & Future Directions

ReCopilot demonstrates that a small, locally-deployable expert LLM can match or exceed frontier general-purpose models on specialized security tasks. Several directions are planned:

**Reinforcement Learning** is the most promising path for improving reasoning consistency. The current SFT approach gives the model reasoning capability, but it struggles with very long chains-of-thought. On-policy RL would allow the model to develop robust test-time scaling organically.

**Disassembly Support** is the most impactful near-term expansion. Some CPU architectures lack mature decompilers, forcing analysts to work directly with assembly. Extending ReCopilot to handle disassembly would cover these practical gaps.

**Language Expansion** beyond C/C++ is necessary as Go and Rust binaries become increasingly prevalent in real-world targets — requiring new data pipelines for these compiled languages.

**Agentic Mode** represents the longer-term vision: a ReCopilot agent that can autonomously plan multi-step analysis, invoke tools, cross-reference findings, and produce structured reports — moving toward a genuine automated reverse engineering system.

---

## <i class="fa-solid fa-scroll" style="color: #95a5a6;"></i> Publication

> **ReCopilot: Reverse Engineering Copilot in Binary Analysis**  
> Guoqiang Chen, Huiqi Sun, Daguang Liu, Zhiqi Wang, Qiang Wang, Bin Yin, Lu Liu, Lingyun Ying  
> QI-ANXIN Technology Research Institute, Beijing, China  
> arXiv:2505.16366 — May 2025
