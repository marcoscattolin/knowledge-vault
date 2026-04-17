---
title: "Contextual Bandits — Lecture Notes CSE599i"
source: "https://courses.cs.washington.edu/courses/cse599i/18wi/"
author:
  - "[[Lalit Jain]]"
  - "[[Neeraja Abhyankar]]"
  - "[[Joshua Fan]]"
  - "[[Kunhui Zhang]]"
published: 2018-02-01
created: 2026-04-17
description: "Lecture notes from UW's Online and Adaptive ML course covering contextual bandits: problem formulation, off-policy evaluation, C-Exp3, Exp4, LinUCB, epsilon-greedy, and ILOVETOCONBANDITS."
tags:
  - "clippings"
  - "bandits"
  - "contextual-bandits"
  - "reinforcement-learning"
  - "lecture-notes"
---

## Contextual Bandits — CSE599i Lecture 10

### Lalit Jain (lecturer) — University of Washington, Winter 2018

---

## 1 Introduction

A **contextual bandit** is a multi-armed bandit where the learner receives **side information (context)** before choosing an arm. The reward of each arm changes every round depending on the context. Regret is defined relative to the best **policy** (mapping from contexts to arms), not the best single arm.

Two viewpoints:
- **Learner sees context directly**: arms are chosen based on observable features.
- **Expert setting**: contexts are hidden inside experts (policies); the learner only sees expert advice and past arm performance.

---

## 2 Applications

- **News/ad recommendation**: arms = articles, context = user features (browsing history, location, age).
- **Medical treatment**: arms = treatments, context = patient symptoms.
- **Mobile health interventions**: arms = messages/actions, context = GPS, heart rate, calendar.

---

## 3 Notation

| Symbol | Meaning |
|--------|---------|
| K | Number of arms |
| T | Total rounds |
| C ⊆ Rᵈ | Context set |
| π : C → [K] | Policy (context → arm) |
| Π | All policies |
| Iₜ | Arm chosen at time t |
| rₜ | Reward at time t |
| ℓᵢ,ₜ | Loss of arm i at time t |

---

## 4 Problem Setting

Each round t:
1. Observe context xₜ ∈ C
2. Choose arm Iₜ ∈ [K]
3. Receive reward rₜ ∈ [0,1] (function of xₜ and Iₜ)
4. Update dataset with (rₜ, xₜ, Iₜ)

**Stochastic setting**: rₜ ~ P(·|xₜ, Iₜ) i.i.d.
**Adversarial setting**: rₜ chosen by adversary.

**Contextual pseudo-regret**:
> R̄_T = max_{π: C→[K]} E[Σ ℓ_{Iₜ,t} − Σ ℓ_{π(xₜ),t}]

---

## 5 Off-Policy Evaluation

Key problem: evaluating policy π from data collected under a *different* logging policy p.

**Inverse Propensity Score (IPS) estimator** — unbiased estimator of expected reward:

> R̂(xₛ, a) = rₛ · 1[aₛ=a] / p(aₛ|xₛ)

**Intuition**: if p(aₛ|xₛ) is large (arm likely chosen anyway), downweight the reward; if the arm was rarely chosen and still got high reward, upweight it. Requires **coverage**: if π(a|x) > 0 then p(a|x) > 0.

---

## 6 Naive Approach: C-Exp3 (Finite Context Set)

Run a separate Exp3 instance for each context x ∈ C.

**Algorithm (C-Exp3)**:
- For each context x, maintain a distribution p(x) over arms.
- At round t with context xₜ: sample arm from p(xₜ), observe loss, update distribution via exponential weights.

**Regret bound**:
> R̄_T ≤ √(2T|C|K ln K)

Note: this is *better* than comparing against a single arm (which ignores context), but can be bad when |C| is large. Key insight: this is O(√(TK ln |Π|)) since |Π| = K^|C|.

---

## 7 Expert Setting: Exp4

When |C| is large, C-Exp3 has too little data per context. **Exp4** (Exponential weights for Exploration and Exploitation with Experts) runs Exp3 over *policies* (experts), not arms.

**Key idea**: maintain qt = distribution over policies. Each expert k provides a distribution ξₖ,ₜ over arms. Mix them: pₜ = E_{k~qₜ}[ξₖ,ₜ]. Use IPS to estimate expert losses, update qt via exponential weights.

**Algorithm (Exp4)**:
1. Initialize q₁ = uniform over Π.
2. At each round: receive expert advice ξₖ,ₜ for all k ∈ Π.
3. Draw arm Iₜ ~ pₜ = Σ qₖ,ₜ ξₖ,ₜ.
4. Compute IPS loss estimates: ℓ̃ᵢ,ₜ = ℓᵢ,ₜ / pᵢ,ₜ · 1[Iₜ=i].
5. Estimate each expert's loss: ỹₖ,ₜ = Σᵢ ξₖ,ₜ(i) ℓ̃ᵢ,ₜ.
6. Update qₜ₊₁ via exponential weights on ỹₖ,ₜ.

**Regret bound (Theorem 4)**:
> R̄_T ≤ √(2TK ln |Π|)

Note: bound depends on K (arms), not |Π| (policies) — because the mixing step allows tighter second-moment bound.

**Limitation**: Exp4 maintains a distribution over all Π → computationally exponential in features when |Π| is large.

---

## 8 LinUCB: Linear Stochastic Contextual Bandits

When Exp4 is infeasible, assume **linear reward structure**:

> r_{a,t} = x_{a,t}ᵀ θ*_a + εₜ

Each arm a has its own coefficient vector θ*_a. For each arm, run **ridge regression** to estimate θ̂_a,t and compute an **ellipsoidal confidence set** C_{a,t}.

**UCB rule**: choose arm at = argmax_a max_{θ ∈ C_{a,t}} x_{a,t}ᵀ θ.

**Algorithm (LinUCB)**:
1. Initialize A_a = λI, b_a = 0 for each arm.
2. At each round: θ̂_a = A_a⁻¹ b_a; compute UCB p_{a,t}.
3. Choose arm at = argmax_a p_{a,t}.
4. Update A_{aₜ} += x_{aₜ,t} x_{aₜ,t}ᵀ; b_{aₜ} += rₜ x_{aₜ,t}.

**Regret bound**:
> Rₙ = O(RSλ^{1/2} dK ln(T/δ) √T)

Parameters are not shared across arms. Regret has worse arm/dimension dependence than Exp4 but is **computationally efficient** (no exponential feature scaling).

**Extension**: embed contexts in high-dimensional space via feature map φ (kernel trick) for non-linear rewards.

---

## 9 ε-Greedy Methods

For general (non-linear) stochastic settings where Exp4 is infeasible due to large Π.

**ε-Greedy algorithm**:
- Phase 1 (exploration, τ rounds): choose arms uniformly at random; collect training data.
- Phase 2 (exploitation, T − τ rounds): fit best policy πτ on exploration data; follow it.

**Regret bound (Theorem 6)**:
> If τ ∝ T^{2/3} → Regret = O(T^{2/3})

Weaker than Exp4's O(√T) but storage complexity = O(1) oracle calls.

**Epoch-greedy**: removes requirement to know T; runs one exploration + many exploitation rounds per epoch; same O(T^{2/3}) guarantee.

---

## 10 ILOVETOCONBANDITS

Combines the O(√T) regret of Exp4 with computational efficiency via oracle calls.

**Key idea**: maintain a sparse distribution qₜ over policies that:
1. Favors policies with low empirical regret (exploitation).
2. Ensures sufficient coverage for policies we still haven't ruled out (exploration).

**ArgMax Oracle (AMO)**: returns the policy in Π maximizing empirical reward over past observations.

**Optimization Problem (OP)** solved at each round to find qₜ:
- Constraint 1 (low regret): Σ_π qπ,ₜ · bₜ(π) ≤ 2K
- Constraint 2 (low variance): E_x[1/q^γ_t(π(x)|x)] ≤ 2K + bₜ(π) for all π

Where bₜ(π) ∝ empirical regret of π.

**Algorithm**: at each round, find qₜ satisfying OP via coordinate descent (≤ 4 ln(1/Kγ)/γ iterations). Play arm ~ q^γₜ(a|xₜ).

**Regret bound (Theorem 7)**:
> R ≤ O(√(KT ln(TN/δ)) + K ln(TN/δ))

Makes ≤ Õ(√T) oracle calls over T rounds.

---

## 11 Algorithm Comparison

| Setting | Small \|Π\| | Large \|Π\| |
|---------|------------|------------|
| **Stochastic** | Exp4 | LinUCB, ε-Greedy, ILOVETOCONBANDITS |
| **Adversarial** | Exp3, Exp4 | Exp4 |

| Algorithm | Regret | Computation | Notes |
|-----------|--------|-------------|-------|
| C-Exp3 | O(√(T\|C\|K ln K)) | O(\|C\|K) per round | Poor for large C |
| Exp4 | O(√(TK ln \|Π\|)) | O(\|Π\|) per round | Exponential in features |
| LinUCB | O(d√T) approx. | O(d²) per round | Assumes linear rewards |
| ε-Greedy | O(T^{2/3}) | 1 oracle call | Simple, low storage |
| ILOVETOCONBANDITS | O(√T) | Õ(√T) oracle calls | Optimal + efficient |

---

## Key References

- [1] Li et al. (2010) — *A contextual-bandit approach to personalized news article recommendation* (LinUCB)
- [4] Bubeck & Cesa-Bianchi (2012) — *Regret analysis of stochastic and nonstochastic multi-armed bandit problems* (Exp3, Exp4)
- [7] Abbasi-Yadkori, Pál, Szepesvári (2011) — *Improved algorithms for linear stochastic bandits*
- [10] Langford & Zhang (2008) — *The epoch-greedy algorithm*
- [11] Syrgkanis et al. (2016) — *Taming the monster: ILOVETOCONBANDITS*
- [12] Agarwal (2017) — *Exploration in contextual bandits*
