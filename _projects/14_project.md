---
layout: page
title: RL-Guided Fuzzing for Binary Vulnerability Detection — an Open Reimplementation and 13-Agent Study of VSGFuzz
description: >
  <i class="fa-solid fa-bug"></i>
  An end-to-end open reimplementation of VSGFuzz (Array, 2025) — graph-embedding
  vulnerability prediction, block-value seed scoring, and reinforcement-learned
  mutation scheduling — together with the controlled multi-seed benchmark the
  paper never ran: thirteen RL agents on one MDP, one reward, the same seeds.
img: assets/img/vsgfuzz/agent_ranking.png
importance: 14
category: work
published: true
---

<style>
  .tech-badge {
    display: inline-block; padding: 2px 10px; margin: 2px; border-radius: 15px;
    font-size: 0.85em; font-weight: bold; color: white;
  }
  .bg-py { background-color: #3776ab; }
  .bg-torch { background-color: #ee4c2c; }
  .bg-rl { background-color: #8e44ad; }
  .bg-fuzz { background-color: #c0392b; }
  .bg-bin { background-color: #2c3e50; }
  .bg-repro { background-color: #16a085; }
</style>

## <i class="fa-solid fa-graduation-cap" style="color: #3498db;"></i> Project Background

A 2025 *Array* paper proposes **VSGFuzz**: predict how vulnerable each function of a stripped binary is with a graph-embedding network, turn that prediction into a per-basic-block value, score every test case by the blocks it executes, and use that score as the **reward for a reinforcement learning agent that decides which mutation operator to apply next**. The paper selects DDPG, with DQN reported only as a comparison.

It is an appealing idea and it ships with a problem: **no artifact.** No source, no seeds, no binaries, no scoring scripts — and the pipeline it describes is built on IDA Pro, Intel Pin and VUzzer (Python 2). None of it can be re-run.

So I re-derived the whole method from the text with open tooling, and then asked the question the paper leaves open: **is DDPG actually the right algorithm for scheduling fuzzer mutations?** Thirteen agents — from a stateless bandit up to SAC — are compared on the *same* MDP, the *same* reward and the *same* seeds. The answer turned out to be an uncomfortable one, and it is the reason this project is worth reading.

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        <span class="tech-badge bg-py">Python</span>
        <span class="tech-badge bg-torch">PyTorch</span>
        <span class="tech-badge bg-rl">Reinforcement Learning</span>
        <span class="tech-badge bg-fuzz">Fuzzing / AFL++</span>
        <span class="tech-badge bg-bin">Binary Analysis / angr</span>
        <span class="tech-badge bg-repro">Reproducible Research</span>
    </div>
</div>

<p class="mt-3">
  <i class="fa-brands fa-github"></i>
  <strong>Code:</strong>
  <a href="https://github.com/Emad-Mahmodi/vsgfuzz-rl">github.com/Emad-Mahmodi/vsgfuzz-rl</a>
</p>

---

## <i class="fa-solid fa-diagram-project" style="color: #e67e22;"></i> What Was Reimplemented

Every numbered equation, algorithm and table of the paper is mapped to code, and every place the paper is ambiguous is documented with the reading implemented and the flag that switches to the alternative — **13 such ambiguities**, from a disagreement between Eq. (4) and Algorithm 1 to the fact that the paper never says how a continuous-control algorithm addresses fourteen discrete operators.

| Paper (Fig. 2) | This implementation |
|---|---|
| ACFG extraction with an IDA Pro plugin | `angr` backend, plus a synthetic generator with executable ACFG semantics |
| structure2vec vulnerability prediction (Alg. 1, Eq. 3–4) | PyTorch — **both** readings of the ambiguous update rule |
| Basic block value `BV = k·p_v + W(b)` (Eq. 5) | ✅ with error-handling-block detection and a clamp for unreachable blocks |
| Seed score `Σ BV + C·HugeScore` (Eq. 6) | ✅ plus four opt-in reward terms, all zero by default |
| 14 mutation operators (Table 1) | ✅ all 14, plus 4 extension operators (Havoc, interesting values, duplicate-run, dictionary-insert) |
| Fuzzing as an MDP (Eq. 7–9) | Gymnasium-style env, with no `gymnasium` dependency |
| DDPG (Alg. 2, Eq. 10–14) and DQN | ✅ both, **plus 11 more agents** |
| Dynamic instrumentation with Intel Pin | emulated executor, native executor, and AFL++ `afl-showmap` edge coverage mapped back onto ACFG blocks |

The pipeline runs static analysis → vulnerability prediction → block values → an MDP whose reward is the seed score, and closes the loop through a score-ranked seed pool.

---

## <i class="fa-solid fa-trophy" style="color: #e74c3c;"></i> Headline Result — the paper's algorithm ranks last, and nothing beats random

Six agents, **10 seeds each**, 25,000 mutations per run, identical reward and identical seed pool.

<div class="row mt-4">
    <div class="col-sm-12 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/vsgfuzz/agent_ranking.png" title="Unique crash sites per agent over 10 seeds" class="img-fluid rounded z-depth-1 shadow-lg" style="width: 100%; height: auto;" %}
    </div>
</div>
<div class="caption">
    <strong>Figure 1:</strong> Unique crash sites, mean ± std over 10 seeds. Orange is the paper's chosen method (DDPG), blue its own comparison point (the DQN family), green everything added here. <strong>DDPG finishes last</strong> — and <code>random</code>, a coin flip among the fourteen operators with no state, no network and no training, sits mid-table.
</div>

| agent | unique crashes | block coverage | runs with a crash | execs to 1st crash | steps/s | vs `random` (A₁₂, p<sub>Holm</sub>) |
|---|--:|--:|:--:|--:|--:|:--:|
| `dueling_dqn` | **3.1 ± 0.7** | **117.8 ± 4.6** | 10/10 | 2,704 | 132 | 0.74, p = 0.28 |
| `dqn` | 2.6 ± 0.7 | 112.6 ± 5.1 | 10/10 | 1,904 | 166 | 0.58, p = 0.97 |
| `ppo` | 2.6 ± 0.7 | 107.1 ± 2.8 | 10/10 | 2,504 | 376 | 0.58, p = 0.97 |
| `random` | 2.5 ± 0.9 | 112.0 ± 4.4 | 10/10 | 2,404 | **6,317** | — (baseline) |
| `ucb1` | 1.8 ± 0.7 | 101.2 ± 5.1 | 10/10 | 2,104 | 5,226 | 0.30, p = 0.33 |
| `ddpg` *(the paper)* | 1.4 ± 1.2 | 97.8 ± 12.1 | **7/10** | 4,004 | 51 | 0.28, p = 0.33 |

Two findings, stated at the confidence the data actually supports:

* **No agent significantly beats uniform-random operator choice.** After Holm correction the best result against `random` is p = 0.28. The baseline the paper argues against — the "equal seed mutation strategy" — is not beaten by any of the thirteen learners, while running **120× faster per step** than DDPG.
* **DDPG ranks last, and *that* part is significant.** Against `dueling_dqn`: A₁₂ = 0.85 (large effect), p<sub>Holm</sub> = 0.030. It is also the only agent with a reliability problem — it found **no crash at all in 3 of its 10 runs**, while every other agent found one in all 10.

---

## <i class="fa-solid fa-chart-simple" style="color: #9b59b6;"></i> Why DDPG Collapses — a measurable mechanism, not a guess

The failure is not bad luck; it is visible in what the policy *does*. Operator-distribution entropy (1 = uniform over the fourteen operators, 0 = always the same one) puts DDPG at **0.10**: 96% of its steps go to two operators, and *which* two differs by seed.

<div class="row mt-4">
    <div class="col-sm-12 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/vsgfuzz/operator_concentration.png" title="Policy collapse versus outcome" class="img-fluid rounded z-depth-1 shadow-lg" style="width: 100%; height: auto;" %}
    </div>
</div>
<div class="caption">
    <strong>Figure 2:</strong> Operator entropy against unique crash sites. DDPG sits alone in the bottom-left corner — a collapsed policy and the worst outcome. Read the trend carefully, though: across these six agents the rank correlation is positive but <strong>not significant</strong> (Spearman ρ = 0.55, p = 0.19), and <code>random</code> has maximal entropy while sitting mid-table. Collapse explains DDPG; staying exploratory is not sufficient on its own.
</div>

This is exactly the failure mode you would predict from pushing a **continuous-control algorithm through a non-differentiable argmax** onto fourteen discrete operators — and it is precisely the gap the paper never explains. Both plausible readings are implemented (`ddpg` with a straight-through Gumbel-softmax relaxation, `ddpg_argmax` as the literal reading), and both land at the bottom of the table.

The stateless bandits (`ucb1`, `thompson`, `exp3`) are the most informative baseline in the set for a different reason: they see **no state at all**. When a bandit ties with a deep agent, the deep policy is not using the state — which suggests most of the work is being done by the seed pool's score-ranked selection rather than by the mutation policy.

---

## <i class="fa-solid fa-flask-vial" style="color: #16a085;"></i> Ablation — does vulnerability guidance actually help?

This is the paper's central claim: that scoring seeds by *predicted vulnerability* beats scoring them by coverage alone. It is testable, and the paper does not test it. Three settings, 3 agents × 10 seeds = **30 runs each**:

* `uniform` — `VP = 0` everywhere, which reduces Eq. (5) to `BV = W(b)` and turns VSGFuzz into a plain coverage-guided fuzzer;
* `model` — the trained graph-embedding predictor, i.e. the method as published;
* `oracle` — ground-truth labels, the ceiling a *perfect* predictor could ever reach.

<div class="row mt-4">
    <div class="col-sm-12 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/vsgfuzz/ablation_vp.png" title="Vulnerability-guidance ablation" class="img-fluid rounded z-depth-1 shadow-lg" style="width: 100%; height: auto;" %}
    </div>
</div>
<div class="caption">
    <strong>Figure 3:</strong> Crash sites and block coverage across the three settings, 30 runs each.
</div>

| VP source | meaning | unique crashes | block coverage | vs `uniform` |
|---|---|--:|--:|:--:|
| `uniform` | coverage only | 2.60 [2.27, 2.90] | 110.7 | — (baseline) |
| `model` | predicted (the paper) | 2.30 [2.00, 2.60] | 106.8 | A₁₂ = 0.40 (small), p = 0.137 |
| `oracle` | ground truth | 2.50 [2.13, 2.83] | 108.0 | A₁₂ = 0.48 (negligible), p = 0.803 |

**No measurable benefit — and there is a structural reason to expect that.** Even a *perfect* predictor does not separate from ignoring vulnerability entirely (A₁₂ = 0.48, p = 0.80). In Eq. (5), `k·p_v` is a constant added to *every* block of a function, so it shifts every path through that function by the same amount: it can re-rank whole **functions**, but says nothing about which **path inside** a function is worth reaching — and reaching the buggy path, not the buggy function, is the hard part.

That diagnosis points directly at the fix: a **distance-to-target** formulation in the style of AFLGo uses the same prediction to steer *within* a function. It is the first item on the future-work list.

---

## <i class="fa-solid fa-brain" style="color: #2980b9;"></i> The Vulnerability Prediction Model

structure2vec graph embeddings over ACFGs (8 attributes per basic block), trained on 150 synthetic programs / 1,800 functions for 25 epochs.

<div class="row mt-4">
    <div class="col-sm-12 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/vsgfuzz/vp_training.png" title="Vulnerability prediction model training" class="img-fluid rounded z-depth-1 shadow-lg" style="width: 100%; height: auto;" %}
    </div>
</div>
<div class="caption">
    <strong>Figure 4:</strong> Validation accuracy holds near 0.91 while validation loss climbs — the classifier's <em>decisions</em> stay stable even as its confidence overfits. On a held-out program it never saw: <strong>93.3% function-level accuracy</strong>, mean <code>VP</code> of 0.85 on vulnerable functions against 0.06 on secure ones.
</div>

The paper reports 99.8% accuracy on Juliet. That number is reachable here too — the synthetic generator's `signal_strength` knob controls how separable the two classes are, and at 1.0 the task is nearly trivial. **The default was deliberately set to keep the predictor imperfect**, because a perfect predictor would make the ablation above meaningless. The `oracle` setting then measures the ceiling separately.

---

## <i class="fa-solid fa-microchip" style="color: #f39c12;"></i> Real Binaries — and an honest caveat about coverage

The executor is pluggable, and the three backends trade fidelity against throughput by three orders of magnitude:

<div class="row mt-4">
    <div class="col-sm-12 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/vsgfuzz/executor_throughput.png" title="Executor throughput" class="img-fluid rounded z-depth-1 shadow-lg" style="width: 100%; height: auto;" %}
    </div>
</div>
<div class="caption">
    <strong>Figure 5:</strong> Executions per second on two cores, log scale. Comparing 13 agents × 3 seeds × 25,000 mutations is on the order of 10⁶ executions — days of wall clock through QEMU, and execution noise would swamp the differences between policies. The emulated backend makes the <em>policy</em> the only variable.
</div>

The real-binary path works end to end: `angr` extracts **28 functions / 1,029 basic blocks from `/usr/bin/who` in 4.5 s** (the paper reports ~1.8 s with IDA Pro), and `afl-showmap` supplies real edge coverage that finds real ASan-detected bugs in two of three bundled vulnerable targets.

> **A caveat worth knowing before trusting the block mapping.** AFL has two edge-ID schemes. In QEMU mode the bitmap index derives from the block address, which is what lets a statically extracted CFG edge be pre-hashed to the slot it will set at runtime — the only mode in which the Eq. (5) block values apply to real coverage. Compile-time instrumentation assigns *random* IDs, and the mapping resolves nothing: measured overlap on one function was 81 static edges against 8 observed IDs, **0 matches**. In that case the raw edge IDs are used as coverage units, scoring degrades to plain coverage guidance, and a warning says so.

---

## <i class="fa-solid fa-scale-balanced" style="color: #34495e;"></i> Experimental Discipline

The methodology is the point of the project as much as the pipeline is.

* **Two-stage design.** A wide screening sweep (13 agents × 3 seeds) carries *no* significance testing and is used only to choose which agents to confirm; a separate confirmatory run (6 agents × 10 seeds) carries every statistical claim. Testing all thirteen at once would make the comparison family large enough that correction swallows any real effect.
* **Non-parametric statistics.** Vargha–Delaney A₁₂ effect sizes with Holm-corrected p-values — the standard for fuzzing evaluation, where crash counts are neither normal nor independent across seeds.
* **Multiple seeds, always.** The paper reports single runs with no variance; fuzzing is highly stochastic. Everything here is mean ± std, plus the number of runs that found *any* crash and the executions-to-first-crash.
* **Committed raw results.** Every run's full JSON timeline is in the repository, so the published numbers can be checked without re-running anything.
* **A preprint that cannot drift.** Every number in the accompanying `paper/` is generated from the run JSON by a script rather than typed in.
* **74 tests**, all runnable without a GPU or any binary tooling, and a one-command `make quick` that reproduces a small end-to-end result in about 20 seconds.

---

## <i class="fa-solid fa-circle-exclamation" style="color: #95a5a6;"></i> What This Does *Not* Show

The default target is a **synthetic program with executable ACFG semantics** — every branch is a real predicate over the input bytes (magic-byte comparisons, length checks, checksums, in the style of LAVA-M bug injection), and planted vulnerable blocks crash when reached. That is a benchmark, not a claim about real software. It tells you which algorithm learns a mutation schedule better under an identical reward; **it does not tell you how many CVEs you will find.** For that, the AFL++ backend is the honest path. The results are a *ranking* on one benchmark family, and only the gap between the DQN family and DDPG is wide enough to lean on.

Nor is this a refutation of the paper's real-world numbers, which were measured on real binaries with tooling that could not be reproduced here. It is a demonstration that the *algorithmic* choice at the centre of the method does not survive a controlled comparison — which is a claim about method design, and a useful one either way.

<p class="mt-3">
  Full paper mapping, all 13 documented ambiguities, architecture notes, extensions and the real-binary guide:
  <a href="https://github.com/Emad-Mahmodi/vsgfuzz-rl">github.com/Emad-Mahmodi/vsgfuzz-rl</a>
</p>

---

## <i class="fa-solid fa-scroll" style="color: #95a5a6;"></i> Based On

> **A reinforcement learning based fuzzing technique for binary programs vulnerabilities detection**
> G. Cao, Y. Ma, M. Geng — *Array*, 27 (2025) 100458
> [doi:10.1016/j.array.2025.100458](https://doi.org/10.1016/j.array.2025.100458) (CC BY)

This repository is an **independent reimplementation** re-derived from the text and is not affiliated with the paper's authors. Code released under MIT. Only fuzz software you own or are authorised to test.
