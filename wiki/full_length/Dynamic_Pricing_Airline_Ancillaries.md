---
title: Feature-Based Dynamic Pricing of Airline Ancillaries
source: https://kth.diva-portal.org/smash/get/diva2:1637829/FULLTEXT01.pdf
author:
  - "[[Muhammed Memedi]]"
published: 2021-09-14
created: 2026-04-17
description: KTH master's thesis applying contextual MAB algorithms to dynamic pricing of airline ancillaries, proposing 'extended play' strategy and two new algorithms (DEGLMUCB, OORMLP-β) to maximize ancillary revenue.
tags:
  - clippings
  - dynamic-pricing
  - bandits
  - contextual-bandits
  - airline
  - ancillaries
---

## Feature-Based Dynamic Pricing of Airline Ancillaries

### Muhammed Memedi — KTH Royal Institute of Technology, September 2021

*Supervisors: Ingvar Max Ziemann (KTH), Balint Fatér (case company — large Scandinavian travel agency)*

---

## Abstract

Airline ancillary revenue grew from $32.5B (2015) to $109.5B (2019), yet pricing strategies have remained simple and static. This thesis applies **multi-armed bandit (MAB) algorithms** for dynamic ancillary pricing to maximize revenue.

**Three contributions**:
1. Identify high-performing and robust pricing policies (contextual bandits beat non-contextual ones).
2. Propose **extended play** strategy: exploit monotonicity in willingness-to-pay to test multiple prices per customer session.
3. Propose two new algorithms: **DEGLMUCB** (contextual bandit) and **OORMLP-β** (valuation model-based).

Main finding: contextual bandits with extended play are both high-performing and robust across different customer behavior settings.

---

## 1 Introduction

### 1.1 Background
Ancillaries = non-ticket airline revenue: baggage fees, seat upgrades, SMS reminders, travel insurance, etc.

**Problem**: prices are currently set manually → static, invariant to demand fluctuations by time and customer type.

**Feature-based dynamic pricing**: use booking session information (context) to estimate demand and set prices accordingly, per customer.

### 1.2 Problem Formulation

A seller offers a product to customer at time t at price pₜ. Customer has valuation vₜ (willingness to pay, WTP). Purchase occurs iff vₜ ≥ pₜ.

**Key assumption — monotonicity in WTP**: if customer is willing to pay p*, they are willing to pay p ≤ p*; if unwilling to pay p*, also unwilling to pay p ≥ p*.

**Seller's goal**: maximize cumulative revenue over time horizon T.

**Dilemma**: set high prices (higher margin, fewer sales) vs. low prices (more sales, lower margin) — classic explore/exploit tradeoff.

---

## 2 Background

### 2.1 Stochastic Bandits
- K arms, each with unknown reward distribution Pₐ
- Learner chooses arm aₜ, observes reward xₜ ~ P_{aₜ}
- **Regret**: R(T) = μ* T − Σ μₜ,ₐ (difference from always playing optimal arm)

### 2.2 UCB1 Policy
Optimistic in face of uncertainty: choose arm with highest UCB = sample mean + confidence radius.

### 2.3 Stochastic Contextual Bandits / LinUCB
- Observe covariate vector cₜ ∈ Rᵈ before choosing arm
- Reward: rₐ,ₜ = cₜᵀ θ*_a + noise (linear model)
- **Disjoint LinUCB**: separate θ*_a per arm; ridge regression + UCB
- Regret: Õ(d√(KT))

### 2.4 Generalized Linear Models for UCB
Extend LinUCB to GLM link functions (logistic, probit, etc.):
> E[r | c, a] = µ(cᵀ θ_a)

where µ is the inverse link function. Filippi et al. (2010), Li et al. (2017).

### 2.5 Pricing Under Parametric Valuation Model
Customer valuation modeled as:
> vₜ = ϕ⁻¹(cₜᵀ θ*)

where ϕ is a monotone function. Purchase occurs iff vₜ ≥ pₜ.

**OORMLP** (Online Optimism with Regularized MLE Policy): Learns θ* via MLE; optimistically sets prices using upper confidence set on θ.

---

## 3 Methods

### 3.1 Data
Historical ancillary purchase data from a large Scandinavian travel agency. Contains features from booking sessions (route, cabin, days to departure, etc.) and binary purchase outcomes.

**Challenge**: non-purchase data has no observed price → **kNN imputation** (k=8) to estimate what price was offered to non-buyers based on similar sessions.

### 3.2 Extended Play Strategy

**Key insight from monotonicity assumption**: if a customer would buy at price p, they would also buy at any price p' ≤ p. Thus, one booking session can reveal information about *multiple price points simultaneously*.

**Extended play**: after a purchase at price pₜ, "virtually" test that the customer would also have bought at prices pₜ, pₜ+Δ, ..., up to some limit. This turns one observation into multiple training signals → **accelerates learning**.

Two versions:
- **E-UCB** (Extended UCB): applies extended play to standard UCB
- **DElinUCB** (Disjoint Extended LinUCB): applies extended play to contextual LinUCB
- **DEGLMUCB**: extends to GLM (probit link function)

### 3.3 DEGLMUCB
Contextual bandit using GLM (probit) reward model + extended play:
> P(purchase | c, p) = Φ(cᵀ θ* − p · β*)

At each round: fit probit model via MLE; compute UCB over the parameter space; choose price maximizing UCB of expected revenue.

### 3.4 OORMLP-β
Modification of OORMLP to better fit the ancillary pricing setting:
- Adds regularization parameter β on the valuation model
- Better handles uncertainty in early rounds
- Assumes customer valuation is a parametric function of covariates → faster convergence if assumption holds

### 3.5 Simulator
Trained a **probit model** on historical data to act as the ground-truth customer behavior simulator:
> P(purchase | c, p) = Φ(cᵀ β)

Four environment settings tested:
1. Probit model (baseline, matches training assumptions)
2. Exaggerated feature dependency (heavy context dependence)
3. Linear valuation + uniform noise
4. Linear valuation + exponential noise

### 3.6 Evaluation Metrics
- **Average revenue per round**
- **Average regret** (difference from oracle optimal price)
- **Conversion rate**

---

## 4 Results

### Experiment 1: Probit Simulator (matched assumptions)
- All contextual bandits (DEGLMUCB, DElinUCB) converge quickly to optimal prices
- Extended play significantly accelerates convergence vs. standard versions
- OORMLP-β performs well under matched assumptions

### Experiment 2: Extended Play vs. Standard
- **Extended play consistently improves revenue and reduces regret** for both UCB and linUCB variants
- Larger exploration parameter α → less exploration, higher revenue in short run but slower learning

### Experiment 3: Exaggerated Feature Dependency
- Contextual bandits (DEGLMUCB) remain robust
- Non-contextual bandits struggle to find the right price for different customer segments

### Experiment 4: Different Environment Settings
| Policy | Probit | Linear+Uniform | Linear+Exp | Feature-Dep |
|--------|--------|---------------|------------|-------------|
| DEGLMUCB | ✅ High | ✅ Good | ✅ Good | ✅ High |
| OORMLP-β | ✅ High | ⚠️ Medium | ❌ Fails | ✅ High |

**Key finding**: OORMLP-β can fail when core valuation model assumptions are violated (especially with exponential noise). DEGLMUCB is more robust.

---

## 5 Discussion

**Contextual vs. non-contextual**: contextual bandits are essential when customer WTP varies by booking features. Non-contextual methods converge to a single price that ignores customer heterogeneity.

**Extended play**: a practically important contribution — turns the monotonicity assumption into a learning advantage. Each booking session teaches the algorithm about multiple price thresholds.

**OORMLP-β**: strong when its parametric assumptions hold (useful if valuation model is well-calibrated), but brittle otherwise. Contextual bandits are safer in production.

**Ethical note**: feature-based pricing sets different prices for different booking orders (not different customers). No personal information is used. However, price variation across groups may be perceived as unfair — future work should investigate fairness constraints.

---

## 6 Conclusions

1. **Contextual bandit policies with extended play** are the best combination: high revenue, robust to different customer behavior settings.
2. **Extended play** is a novel and practical strategy that significantly accelerates learning by exploiting monotonicity in WTP.
3. **DEGLMUCB** and **OORMLP-β** are new algorithms better suited to the ancillary pricing problem than off-the-shelf MAB algorithms.
4. OORMLP-β achieves similar performance to contextual bandits under matched assumptions but fails under model mismatch.

**Future work**:
- Delayed feedback (customer decision takes time)
- Production cost modeling
- Multi-product pricing (ancillary bundles)
- Fairness-constrained pricing policies

---

## Key References

- Javanmard & Nazerzadeh (2019) — *Feature-based dynamic pricing* (logarithmic regret)
- Wang et al. (2020) — *Dynamic pricing with contextual features*
- Shukla et al. (2019) — *Feature-based dynamic pricing of airline ancillaries* (APP-LM, APP-DES, DNN-CL)
- Shah et al. (2019) — *Semi-parametric contextual pricing* (O(√T) regret)
- Filippi et al. (2010) — *Generalized linear bandits*
- Li et al. (2010) — *LinUCB for personalized recommendation*
- Lattimore & Szepesvári (2020) — *Bandit Algorithms* (textbook)
