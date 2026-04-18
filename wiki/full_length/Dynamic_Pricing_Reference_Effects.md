---
title: "Dynamic Pricing and Learning with Long-term Reference Effects"
source: "https://arxiv.org/abs/2402.12562"
author:
  - "[[Shipra Agrawal]]"
  - "[[Wei Tang]]"
published: 2024-02-19
created: 2026-04-17
description: "Introduces the Averaging Reference Mechanism (ARM) for dynamic pricing, proves markdown policies are near-optimal under long-term reference effects, and provides an O(sqrt(T)) regret learning algorithm for unknown demand parameters."
tags:
  - "clippings"
  - "dynamic-pricing"
  - "pricing-theory"
  - "bandits"
  - "regret-bounds"
  - "reference-price"
  - "markdown-pricing"
---

## Dynamic Pricing and Learning with Long-term Reference Effects

### Shipra Agrawal, Wei Tang (Columbia University) — 2024

---

## Abstract

The paper studies dynamic pricing where customer demand depends on both the current price and a *reference price* — what customers perceive as the "normal" price. The authors introduce the **Averaging Reference Mechanism (ARM)**: the reference price equals the simple (equal-weight) average of all past prices, as opposed to the exponentially-weighted average of the well-studied ESM model.

Under ARM, fixed-price policies are shown to be highly suboptimal (Ω(T) revenue loss), while **markdown pricing is near-optimal** for all model parameters. For linear demand, a structural characterization of the near-optimal markdown curve is provided. When demand parameters are unknown, an efficient learning algorithm achieves regret Õ(p̄^3.5 √T), with a matching lower bound of Ω(p̄ √T).

---

## 1 Introduction

### 1.1 Motivation

Customer reference prices are increasingly shaped by **long-term price history** rather than short-term memory. Online platforms make historical prices readily accessible:
- Google Hotel labels like "28% less than usual" compare to the past year's average.
- Amazon displays "Was Price" based on recent price history.
- Third-party trackers (CamelCamelCamel, PriSync, Wiser) let customers compare prices over months or years.

This contrasts with the dominant **Exponential Smoothing Mechanism (ESM)**, which weights recent prices exponentially more and captures only short-term reference effects. Under ESM, a fixed price is near-optimal. Under ARM, it is not.

### 1.2 Main Contributions

| Contribution | Result |
|---|---|
| Fixed-price suboptimality under ARM | Revenue gap can be Ω(T) vs. optimal (Proposition 1) |
| Markdown optimality — gain-seeking (η⁺ ≥ η⁻) | Markdown is **optimal** (Theorem 1a) |
| Markdown near-optimality — loss-averse (η⁺ < η⁻) | Markdown within O(log T) of optimal (Theorem 1b) |
| Near-optimal markdown characterization (linear demand) | Explicit price curve formula + O(log T) computation (Propositions 2, 3) |
| Learning algorithm (unknown parameters) | Regret Õ(p̄^3.5 √T) (Theorem 2) |
| Regret lower bound | Ω(p̄ √T) (Proposition 4) |

---

## 2 Problem Formulation

### 2.1 Demand Model

At each round t, the seller sets price pₜ ∈ [0, p̄]. Observed demand:

> **Dₜ = D(pₜ, rₜ) + εₜ**
> where εₜ are i.i.d. zero-mean shocks, and
> **D(p, r) = H(p) + η⁺·(r − p)⁺ − η⁻·(p − r)⁺**

- H(p): base demand (non-decreasing in p in general; linear H(p) = b − ap for the tractable setting)
- η⁺ ≥ 0: gain effect — demand boost when price is below reference
- η⁻ ≥ 0: loss effect — demand reduction when price is above reference
- **Gain-seeking**: η⁺ > η⁻; **Loss-averse**: η⁺ < η⁻; **Neutral**: η⁺ = η⁻

### 2.2 Averaging Reference Mechanism (ARM)

> **rₜ₊₁ = (r₁ + Σₛ₌₁ᵗ pₛ) / (t + 1)**

Equivalently: **rₜ₊₁ = ζₜ rₜ + (1 − ζₜ) pₜ** with ζₜ = t/(t+1)

Key contrast with ESM: in ESM, ζₜ ≡ ζ is a constant (exponential weighting); in ARM, ζₜ → 1 as t grows, so each new price has a diminishing impact on the reference price — making past prices matter equally in the long run.

### 2.3 Regret Definition (Learning Setting)

> **REG[T, I] = V*(r₁) − E[Vᵖ(r₁)]**
>
> Cumulative expected revenue loss vs. a clairvoyant optimal policy over T rounds.

---

## 3 Characterizing Near-Optimal Pricing Policy

### 3.1 Suboptimality of Fixed Pricing

**Proposition 1.** There exists an ARM instance with linear demand and loss-neutral customers where any fixed-price policy p satisfies:

> **V*(r₁) − Vᵖ(r₁) = Ω(T)**

Intuition: under ESM, only a constant window of past prices matters; under ARM, the averaging window grows linearly with t, so the entire price history influences reference price. A simple two-price policy (high then low) achieves Ω(T) more revenue than any fixed price.

### 3.2 Near-Optimality of Markdown Pricing

A **markdown policy** is any policy where prices are non-increasing over time: pₜ ≥ pₜ₊₁ for all t.

**Theorem 1.**
- **(1a) Gain-seeking customers (η⁺ ≥ η⁻):** The optimal policy is a markdown policy.
- **(1b) Loss-averse customers (η⁺ < η⁻):** There exists a markdown policy p with:

> **V*(r₁) − Vᵖ(r₁) = O(p̄(p̄ − r₁)(η⁻ + η⁺) ln T)**

**Key lemmas supporting Theorem 1:**
- *Lemma 3.1:* If customers are gain-seeking, optimal policy from any starting point is markdown (proved by showing swapping an increasing-price pair always improves revenue).
- *Lemma 3.2:* Even for loss-averse customers, if starting reference price = p̄, the optimal policy is markdown.
- *Lemma 3.3 (Lipschitz on reference price):* V*(r', t₁) − V*(r, t₁) ≤ O(p̄ t₁ (r' − r)(η⁻ + η⁺) ln T/t₁) for r' ≥ r.

Theorem 1b follows by running the markdown policy designed for r₁ = p̄ regardless of the true r₁.

### 3.3 Structure of Near-Optimal Markdown Curve (Linear Demand)

For linear base demand H(p) = b − ap, the near-optimal price curve p̃(r) has the following structure:

> **pₜ = p̄** for t ∈ [1, t† − 1]  (hold at maximum price)
> **pₜ = p†** at t = t†            (initial markdown)
> **pₜ = pₜ₋₁ − η⁺ rₜ₋₁ / (2(a + η⁺)t + η⁺)** for t ∈ [t† + 1, T − 1]
> **pₜ = (η⁺ rₜ + b) / (2(a + η⁺))** at t = T

where t† and p† depend on (a, b, η⁺, r₁). The markdown rule depends only on the **gain parameter η⁺**, not η⁻.

- When η⁺ = η⁻ (loss-neutral): p̃(r) is **exactly optimal**.
- When η⁺ ≠ η⁻: p̃(p̄) is **near-optimal** (revenue gap = O(ln T)) for any r₁.

**Proposition 3 (Efficient computation):** Algorithm 3 computes p̃(r) by solving only O(ln T) linear systems (via binary search on t†).

---

## 4 Learning and Optimization under Demand Uncertainty

### 4.1 Learning Challenges

Two key challenges arise that make standard algorithms inapplicable:

1. **Non-stationarity prevents direct parameter estimation.** Iterated least squares (used in prior work) requires stationary demand; under ARM, demand depends on the entire history of prices through the evolving reference price.

2. **Restart mechanisms fail under ARM.** The optimal revenue V*(rₜ, t) is very sensitive to the reference price rₜ. Moving the reference price from rₜ to a target rₜ' requires O(t |rₜ' − rₜ|) rounds, which leads to linear regret when t = O(T).

### 4.2 Key Insight: Reparameterization via Two-dimensional θ*

The near-optimal price curve can be reparameterized to depend on a **2-dimensional policy parameter** θ* = (C*₁, C*₂) instead of the full 4-dimensional model parameter I = (a, b, η⁺, η⁻):

> **C*₁ = η⁺ / (2(a + η⁺))**
> **C*₂ = b / (2(a + η⁺))**

This parameter θ* also fully characterizes the **greedy price** (single-round revenue maximizer) for reference prices r ∈ (p̄ − δ, p̄]:

> **pGR(r) = C*₁ · r + C*₂**

### 4.3 Algorithm 1: Explore-then-Exploit

**Phase 1 & 2 — Exploration:** Run `LearnGreedy(T₁, rₐ, p̄)` and `LearnGreedy(T₁, r_b, p̄)` for two carefully chosen reference prices rₐ, r_b ∈ (p̄ − δ, p̄]. Each call uses bandit stochastic convex optimization (Algorithm 2) to estimate pGR(rₐ) and pGR(r_b).

- Before each learning round, a subroutine `ResetRef` resets the current reference price to the target r by offering prices in [0, p̄].
- Leveraging the quadratic structure of Rev(·, r), an improved estimation error of O(T₁^{−1/2}) is achieved (vs. O(T₁^{−1/4}) for general strongly-concave functions).

**Phase 3 — Exploitation:** Solve the 2×2 linear system:

> **ĈC₁ = (p̂GR(r_b) − p̂GR(rₐ)) / (r_b − rₐ)**
> **ĈC₂ = (p̂GR(rₐ) r_b − p̂GR(r_b) rₐ) / (r_b − rₐ)**

Then implement the price curve p̃(p̄, T₂, θ̂) for the remaining T − T₂ rounds.

---

## 5 Regret Analysis

**Theorem 2 (Regret upper bound).** With T₁ = Θ̃(p̄² √T / √(1 + p̄)), Algorithm 1 achieves:

> **REG[T, I] ≤ Õ(p̄³ √(p̄ T)) = Õ(p̄^{3.5} √T)**

**Proof outline (4 steps):**
1. **Estimation error on θ*.** Using Algorithm 2's bandit gradient estimates, ‖θ̂ − θ*‖ = O(p̄² √p̄ · log(log T₁) / ((r_b − rₐ) √T₁)).
2. **Resetting regret.** Total rounds used by ResetRef across both exploration phases is O(T₁).
3. **Lipschitz error in policy space.** Revenue gap between two price curves with different θ is bounded via strong concavity of V^p and Lipschitz continuity of p̃(r, t₁, θ) in θ (Propositions 5, 6).
4. **Combining.** Setting T₁ = Θ̃(p̄² √T / √(1 + p̄)) and δ = 1/T yields the Õ(p̄^{3.5} √T) bound.

**Comparison with ESM baseline:** den Boer & Keskin (2022) achieve Õ(p̄⁶ √T / (1 − ζ)²) under ESM. Under ARM, ζₜ = t/(t+1) → 1 so naive application would give linear regret, highlighting ARM's fundamentally different structure.

**Proposition 4 (Lower bound).** For any algorithm, with p̄ ≥ 1 and instances with η⁺ = η⁻ = 0:

> **sup_{I} REG[T, I] ≥ Ω(p̄ √T)**

There remains a gap between the lower bound's linear p̄ dependence and the upper bound's p̄^{3.5} — closing this gap is noted as an open problem.

---

## 6 Conclusions / Takeaway

**Main messages:**
- ARM (averaging reference mechanism) captures long-term reference effects and fundamentally differs from ESM: fixed-price policies lose Ω(T) revenue, while markdown policies are near-optimal.
- For linear demand, the near-optimal markdown curve has an explicit closed form depending only on η⁺, and can be computed in O(ln T) operations.
- With unknown parameters, explore-then-exploit with bandit convex optimization achieves near-optimal Õ(√T) regret.

**Limitations and future directions:**
- Results are for linear base demand; extending to non-parametric models is open.
- Algorithm is explore-first; UCB/Thompson Sampling might tighten the p̄ dependency.
- The paper assumes myopic customers; strategic (forward-looking) customers who anticipate markdown may behave differently.
- An intermediate model ζₜ = 1/t^α (interpolating between ARM α=1 and ESM α=0) is proposed as a future direction.
- Reference price update currently depends only on offered prices, not actual sales; incorporating sales volume into the update is noted as a modeling extension.

---

## Key References

- Popescu & Wu (2007) — *Dynamic Pricing Strategies with Reference Effects* (baseline under ESM; fixed price near-optimal for loss-averse customers)
- den Boer & Keskin (2022) — *Dynamic Pricing with Demand Learning and Reference Effects* (ESM learning benchmark; regret Õ(√T) under ESM)
- Fibich, Gavious & Lowengart (2003) — *Explicit solutions with nonsmooth reference-price effects* (ESM optimal policy characterization)
- Keskin & Zeevi (2014) — *Dynamic Pricing with Unknown Demand: Asymptotically Optimal Semi-Myopic Policies* (lower bound Ω(√T) without reference effects)
- Lazear (1986) — *Retail Pricing and Clearance Sales* (early markdown pricing theory)
- Shamir (2013) — *On the Complexity of Bandit and Derivative-Free Stochastic Convex Optimization* (bandit SCO algorithm used in exploration phase)
- Ji, Yang & Shi (2023) — *Online Learning and Pricing for Multiple Products with Reference Price Effects* (multi-product ESM with learning)
- Jia, Li & Ravi (2021, 2022) — *Markdown pricing under unknown demand / with monotonicity constraint* (related markdown learning work)
- Hu, Chen & Hu (2016) — *Dynamic Pricing with Gain-Seeking Reference Price Effects* (cyclic markdown optimal under gain-only ESM)
- Cheung, Simchi-Levi & Zhu (2020) — *RL for Non-Stationary MDPs* (general MDP regret; restart mechanisms not applicable to ARM)
