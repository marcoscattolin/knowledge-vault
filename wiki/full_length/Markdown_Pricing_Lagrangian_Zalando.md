---
title: "Tricks from the Trade for Large-Scale Markdown Pricing: Heuristic Cut Generation for Lagrangian Decomposition"
source: "https://arxiv.org/abs/2404.02996"
author:
  - "[[Robert Streeck]]"
  - "[[Torsten Gellert]]"
  - "[[Andreas Schmitt]]"
  - "[[Asya Dipkaya]]"
  - "[[Vladimir Fux]]"
  - "[[Tim Januschowski]]"
  - "[[Timo Berthold]]"
published: 2025-02-04
created: 2026-04-17
description: "Introduce heuristic cut generation strategies within a Lagrangian decomposition framework to accelerate large-scale markdown pricing at Zalando, improving weekly profits by millions of Euros with negligible impact on solve time."
tags:
  - "clippings"
  - "dynamic-pricing"
  - "markdown-pricing"
  - "optimization"
  - "operations-research"
  - "retail"
  - "machine-learning"
---

## Tricks from the Trade for Large-Scale Markdown Pricing: Heuristic Cut Generation for Lagrangian Decomposition

### Streeck, Gellert, Schmitt, Dipkaya, Fux, Januschowski (Zalando SE) — Berthold (TU Berlin) — 2025

---

## Abstract

In automated decision-making for online fashion retail, the *predict-then-optimize* paradigm is routinely applied for markdown pricing. The optimization step is a large-scale mixed-integer program (MIP), prohibitive for off-the-shelf solvers. This paper introduces heuristics — both primal heuristic methods and a cutting-plane generation technique — designed to work alongside a Lagrangian decomposition framework, yielding near-optimal solutions. Deployed in Zalando's production environment (catalog of millions of products), the approach improves weekly profits by millions of Euros. Empirically, the method improves revenue by 3–6% and profit by 2–5% with negligible impact on solution speed.

---

## 1 Introduction

Online retailers like Zalando (~50M customers, 25+ European countries, ~2M articles) face strict constraints:

1. **Scalability**: assortments grow continuously
2. **Speed**: price updates must meet commercial timelines (minutes to hours)

Markdown pricing — setting discounted prices to clear inventory before season end — is one of the most impactful commercial levers. It requires **integrated optimization** across all markets simultaneously (due to *linking constraints* on discount rates and GMV targets), which makes the problem size enormous.

The *predict-then-optimize* paradigm is standard: ML forecasting models estimate demand, and the output feeds a MIP optimizer. The key prior work (Li et al., 2022) established the basic Lagrangian decomposition framework at Zalando. Since its 2021 deployment, occasional non-convergence issues motivated the extensions in this paper.

**Main contribution**: extensions to the cutting-plane-based Lagrangian descent via novel heuristic cut generation strategies, addressing scalability and convergence, with demonstrated multi-million Euro impact in production.

---

## 2 Related Work

The paper sits at the intersection of three communities:

- **Machine Learning**: demand forecasting (DeepAR, N-BEATS, TFT), reinforcement learning for pricing
- **Econometrics**: price elasticity estimation, causal inference methods
- **Operations Research**: MIP heuristics, cut generation, Lagrangian relaxation (the focus here)

Notable related work on heuristics for MIPs includes feasibility pump (Berthold et al.), local branching (Fischetti & Lodi), and cut generation from Lagrangian relaxations (Fischetti & Salvagnin 2011). The specific tuning for markdown pricing is novel and — to the authors' knowledge — under-explored in this form.

---

## 3 Background and Problem Formulation

### 3.1 Primal Pricing Problem

For an assortment of *n* articles, the goal is to choose prices x to maximize long-term profit (LTP):

> **Primal problem (P)**:
> maximize sum over i of f_i(x_i), subject to Ax ≤ b and x in X = X_1 × ... × X_n
> where Ax ≤ b are *linking constraints* (e.g., average discount rate targets per country), and X_i is the per-article feasible set.

At Zalando's scale: ~500,000 articles, each with up to 10,000 variables (5,000 binary) and 10,000 constraints, pricing across 14 countries on weekly granularity. Total problem: billions of variables — intractable for general-purpose solvers.

**Linking constraints example — sales-weighted discount rate (sDR)**:

> sDR = sum_i (p_i - x_i_bar) * s_{i,x_i_bar} / sum_i p_i * s_{i,x_i_bar}
>
> where p_i = undiscounted price, s_{i,x_i_bar} = units sold at price x_i_bar.

Typically 48 sDR constraints (24 lower and 24 upper bounds, one pair per country).

### 3.2 Lagrangian Relaxation

The linking constraints are dualized using multipliers λ ≥ 0:

> **Lagrangian relaxation**:
> LR(λ, x) = sum_i f_i(x_i) - λ^T (b - Ax)
>
> Optimizing over x decomposes into n **independent single-article problems**:
> LR(λ) = sum_i max_{x_i in X_i} [f_i(x_i) + λ^T A_i x_i] - λ^T b

Decomposability is key: each article's subproblem can be solved in under 10 seconds. With ~1,000-core cloud clusters (AWS C5), one full Lagrangian evaluation (all 500K articles) takes **3–12 minutes** of wall time, or 150–700 CPU hours. Given a 3-hour turnaround window, only **10–15 outer iterations** are feasible — far fewer than typical for a problem with 48 constraints.

---

## 4 Overview of Algorithms

### 4.1 Cutting Plane Procedure

The Lagrangian multiplier problem min_{λ≥0} LR(λ) is reformulated as a linear program whose constraints correspond to feasible primal solutions:

> **Multiplier LP (exact)**:
> min_{λ≥0, µ} µ  s.t.  µ - (b - Ax)^T λ ≥ sum_i f_i(x_i)  for all x in X

Since |X| can be exponential, a **cutting plane** approach iteratively adds only violated constraints. At iteration j:

> **Relaxed cutting plane problem (3)**:
> min_{λ≥0, µ} µ  s.t.  µ ≥ sum_i f_i(X_i^k) + λ^T(b - AX^k)  for all k ≤ j

Each outer iteration: (1) solve LR(λ^j) to get X^{j+1}; (2) add new cut; (3) re-solve the small LP to get new λ^{j+1}, µ^{j+1}.

**Bounds tracking**:
- *Dual bound* (upper bound on P): min_{k≤j} LR(λ^k)
- *Relaxed primal bound* (lower bound on µ*): µ^j

The LP in step (3) has only j constraints and one variable per linking constraint — very cheap. The bottleneck is step (1): solving LR(λ) over the entire assortment.

### 4.2 Primal Heuristic

Even at convergence, Lagrangian solutions typically violate linking constraints. A small MIP selects, for each article i, which iteration's price suggestion to adopt:

> **Primal MIP (5)**:
> maximize (scaled profit + scaled constraint violation slack)
> s.t. x_i = X_i^{k_i} for some k_i ≤ j (binary selection variables y_{ik})
>      sum_k y_{ik} = 1 for all i

Since each article's objective f_i(X_i^k) is already evaluated, this MIP is much easier than the original. It produces the final feasible price recommendation.

---

## 5 Heuristics for the Markdown Pricing Problem

### 5.1 Motivation

Each exact Lagrangian evaluation costs 3–12 minutes. The idea: generate **additional valid cuts cheaply** by recombining already-evaluated article solutions X^1,...,X^j (for which all f_i(X_i^k) are cached). A heuristic solution X^{j+1} is valid (in X), but not guaranteed to violate the current cut:

> **Cut validity condition** (Inequality 4):
> LR(λ^j, X^{j+1}) - µ^j > 0  (i.e., µ^j is violated → cut is useful)

### 5.2 Three Heuristic Strategies

| Strategy | Description | Cost |
|---|---|---|
| **Random** | Sample each article's price uniformly from past iterations | Negligible (baseline) |
| **Maximum violation** | For each article, pick the iteration k maximizing f_i(X_i^k) + λ^T A_i X_i^k (Eq. 6) — reweight cached values and sort | O(n·j) cheap reweight |
| **Feasibility** | Run the primal MIP (Section 4.2) on current solutions | More expensive; not repeatable in sequence |

> **Maximum violation per article** (Eq. 6):
> For each i: argmax_{k ≤ j} [f_i(X_i^k) - λ^T A_i X_i^k]

### 5.3 Stopping Criteria and Algorithm

Heuristic cuts are added in inner loops between exact LR evaluations. The algorithm (Algorithm 1) switches back to exact LR evaluation if:
1. The new cut does not change the optimal λ
2. **Efficacy** falls below threshold tol_e:

> **Cut efficacy** (Eq. 7):
> e(λ^j, µ^j, X^j) = (LR(λ^j, X^j) - µ^j) / ||λ^j||

**Hyperparameters (mostly ad-hoc)**:
- n_bar = 10 (max outer iterations)
- m_bar = 100 (max heuristic cuts per outer iteration)
- tol_e = 1.0 (efficacy threshold)
- tol_µ = 1e-6 (dual gap tolerance)

### 5.4 Theoretical Comparison with Disaggregated Formulation

Three formulations of the cutting plane problem, in increasing tightness:
1. **Aggregated (3)**: j+1 variables, j constraints — cheapest
2. **Aggregated + heuristic cuts (9)**: j+s constraints, same variables
3. **Disaggregated / multi-cut (8)**: ~N+j variables, N·j constraints — tightest but intractable for N≈500K

The heuristic cut formulation (9) lies strictly between (3) and (8) in bound quality. After a finite number of maximum-violation heuristic applications, the bound equals the disaggregated bound — because the maximum violation heuristic exactly separates the constraint family implied by (8).

A **partially aggregated** compromise (10) randomly groups articles into M groups. This is evaluated experimentally but found inferior.

---

## 6 Experiments and Real-World Impact

**Setting**: ~500,000 articles, 14 countries, 48 sDR linking constraints, pricing updated at least twice per week, 3-hour turnaround target.

### 6.1 Comparison of Cut Generation Strategies

Tested on a hard instance that fails to converge without heuristics:

| Strategy | Relative dual gap at iteration 10 |
|---|---|
| Baseline (no heuristic) | >> 100% (no convergence) |
| Random cuts | ~same as baseline (ineffective) |
| Feasibility heuristic | ~7.4% |
| **Maximum violation** | **< 0.4%** (below 2% at iteration 6) |

Random cuts are indistinguishable from baseline. Maximum violation heuristic is clearly best. The feasibility heuristic is intermediate. **Maximum violation selected for production.**

### 6.2 Maximum Violation vs. Partial Aggregation

Tested on 86 "difficult" dual problems. Both strategies eventually reach the same bound (equal to the disaggregated bound), but maximum violation is substantially faster:

**Table 1 — Geometric mean ± geometric std of solving time (seconds) to reach target gap (86 instances)**:

| Gap to best bound | Partially aggregated | Maximum violation |
|---|---|---|
| 0.1% | 0.12 ± 18.43 s | 0.11 ± 4.12 s |
| 0.01% | 4.27 ± 10.24 s | **0.59 ± 3.45 s** |
| 0.001% | 47.98 ± 3.35 s | **1.69 ± 2.90 s** |

Key observations:
- For tight gaps (0.001%), maximum violation is ~28x faster in geometric mean
- Partial aggregation has high variance (some instances take >100s); maximum violation is consistent (<20s always)
- Easier to tune stopping criteria for maximum violation (monotone improvement, checkable at every step)

In full Algorithm 1 runs (3 hard instances), maximum violation also outperforms both partial aggregation group sizes (2,000 and 5,000 articles) and closes gaps to ~0.0001% vs. ~0.01% for partial aggregation.

### 6.3 Impact on Commercial KPIs

Forward-running simulation across 8 dates comparing primal solutions with/without maximum violation heuristic (100 cuts per iteration):

**Table 2 — Simulated improvement in commercial KPIs (M €)**:

| Experiment | LTP | GMV | PC2 |
|---|---|---|---|
| 0 | 0.18 | 0.31 | 0.09 |
| 1 | 0.55 | 0.87 | 0.23 |
| 2 | 3.30 | 6.02 | 1.42 |
| 3 | 3.63 | 6.07 | 1.54 |
| 4 | 3.65 | 1.38 | 0.75 |
| 5 | 3.97 | 4.74 | 1.28 |
| 6 | 5.71 | 0.00 | 0.00 |
| 7 | 6.20 | 7.25 | 2.09 |
| **avg** | **3.40** | **3.33** | **0.93** |
| **sum (8 weeks)** | **27.19** | **26.64** | **7.40** |

Projected annualized LTP improvement: ~175M €.

**Causal impact analysis** (post-deployment validation, 25 weeks pre / 10 weeks post, Bayesian structural time-series, using old optimizer without heuristic as covariate):
- PC2: p=0.0001, observed weekly effect +5.8M € (expected +1.56M €) — statistically significant
- GMV: p=0.18, observed +7.9M € (expected +16.7M €) — directionally consistent, not significant

---

## 7 Conclusion / Takeaway

**Main message**: heuristic cut generation (maximum violation strategy) within a Lagrangian decomposition framework solves the convergence and speed challenges of large-scale markdown pricing. The approach:
- Requires no additional exact LR evaluations
- Provides valid cuts by cheaply recombining cached article solutions
- Provably converges to the disaggregated bound
- Outperforms partial aggregation in speed, consistency, and tunability
- Generates multi-million Euro weekly profit improvements in production at Zalando

**Practical limits**:
- Parameter tuning (n_bar, m_bar, tol_e) is ad-hoc; sensitivity analysis was limited
- Direct A/B testing was not possible due to concurrent unrelated experiments; causal impact analysis used as alternative
- The specific threshold values likely do not generalize directly to other applications without re-tuning

**Future work** (implicit): further heuristic combinations, generalizing to other Lagrangian MIP applications beyond markdown pricing.

---

## Key References

- Li et al. (2022) — *Large-scale price optimization for an online fashion retailer* (Springer) — foundational prior work: Lagrangian decomposition setup, primal heuristic, problem formulation at Zalando
- Guignard (2003) — *Lagrangean relaxation* (TOP) — theoretical background on Lagrangian methods, disaggregated formulations
- Fischetti & Salvagnin (2011) — *A relax-and-cut framework for Gomory mixed-integer cuts* — closest related work on heuristic cuts from Lagrangian relaxations of general MIPs
- Frangioni (2005) — *About Lagrangian methods in integer optimization* — multi-cut / disaggregated formulation theory
- Mitra, Garcia-Herreros & Grossmann (2016) — *Cross-decomposition with primal-dual multi-cuts* — partial aggregation formulation context
- Berthold (2014a, 2014b) — *Heuristic algorithms in global MINLP solvers*; *RENS – the optimal rounding* — general MIP heuristics background
- Brodersen et al. (2015) — *Inferring causal impact using Bayesian structural time-series models* — method used for production impact measurement
- Ferreira, Lee & Simchi-Levi (2016) — *Analytics for an online retailer* — predict-then-optimize paradigm for pricing
- Kunz et al. (2023) — *Deep learning based forecasting: a case study from the online fashion industry* — forecasting models that feed the optimizer at Zalando
- Turner et al. (2023) — *Cutting plane selection with analytic centers and multiregression* — cut efficacy measure used in stopping criteria
