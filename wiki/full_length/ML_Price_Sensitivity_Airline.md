---
title: "Machine Learning based Framework for Robust Price-Sensitivity Estimation with Application to Airline Pricing"
source: "https://arxiv.org/abs/2205.01875"
author:
  - "[[Ravi Kumar]]"
  - "[[Shahin Boluki]]"
  - "[[Karl Isler]]"
  - "[[Jonas Rauch]]"
  - "[[Darius Walczak]]"
published: 2022-12-21
created: 2026-04-17
description: "Two-stage semi-parametric framework for robust price elasticity estimation in airlines: ML-based nuisance estimation (stage 1) + Bayesian dynamic GLM for price-sensitivity parameters (stage 2), reducing estimation error from 25% to 4%."
tags:
  - "clippings"
  - "dynamic-pricing"
  - "causal-inference"
  - "price-elasticity"
  - "airline"
  - "machine-learning"
---

## ML-Based Framework for Robust Price-Sensitivity Estimation — Airline Pricing

### Kumar, Boluki, Isler, Rauch, Walczak (PROS Inc.) — December 2022

---

## Abstract

**Problem**: Estimate price elasticities from observational airline sales data, without loss (no-purchase) information, to drive automated dynamic pricing systems.

**Methodology**: 
- Poisson semi-parametric demand model: parametric price part + non-parametric nuisance volume part.
- **Two-stage estimation**: 
  - Stage 1: ML (deep neural nets) to estimate nuisance parameters (baseline demand and price conditional on features).
  - Stage 2: Bayesian dynamic GLM to estimate price-sensitivity parameters, using Neyman-orthogonal scores for robustness.

**Results**: Two-stage approach reduces price-sensitivity estimation error from **25% to 4%** in realistic simulation settings.

---

## 1 Introduction

### 1.1 Problem Context

Airlines price **itineraries** (route + departure date + class + restrictions). Key challenges:
- **No loss data**: airlines selling via OTAs (Expedia, KAYAK) don't observe customers who didn't book → can't directly model purchase probabilities.
- **Confounders**: demand and price are both driven by the same features (seasonality, booking horizon, competition, events) → naïve regression gives biased elasticity estimates.
- **Randomized pricing experiments** are costly, perceived as unfair, and hard to scale.

### 1.2 Airline-Specific Complications
- Seasonal demand patterns (summer, holidays)
- High-sensitivity leisure travelers book early; low-sensitivity business travelers book late
- Competition from other airlines on same routes
- Revenue Management (RM) systems compute bid prices at aggregate level; pricing is a separate refinement layer
- RM-generated prices are correlated with many demand-driving features → strong confounding

### 1.3 Key Insight
Neither fully parametric models (too rigid) nor pure ML (non-interpretable, biased elasticities) suffice alone. The solution: **semi-parametric hybrid** — parametric for price effects (interpretable) + ML for everything else (flexible).

---

## 2 Model: Poisson Semi-Parametric Demand Response

**Demand model**:
> Y_{it} | X_{it}, P_{it} ~ Poisson(λ(P_{it}; X_{it}, θ, φ))
> log λ = P_{it} θᵀ W_{it} + φ(X_{it})

Where:
- Y_{it}: observed purchases for itinerary i in period t
- P_{it}: price offered
- X_{it}: feature vector (seasonality, booking horizon, competition, etc.)
- W_{it}: features interacting with price (heterogeneous elasticity)
- θ: **price-sensitivity parameters** (target of estimation)
- φ(·): nuisance volume function (non-parametric, captures baseline demand)

**Price model**:
> P_{it} | X_{it} = g(X_{it}) + ε_{it}, E[ε|X] = 0

g(·) captures RM system pricing rules — complex and high-dimensional.

**Goal**: estimate θ (price elasticity) robustly despite not knowing φ(·) or g(·).

---

## 3 Estimation Approaches

### 3.1 Direct Approach: Wide & Deep Neural Network

Adapt Google's Wide & Deep architecture:
- **Wide part** (linear): models P_{it} θᵀ W_{it} → price-sensitivity
- **Deep part** (neural net): models φ(X_{it}) → baseline demand
- Joint training with Poisson negative log-likelihood loss

**Problem**: regularization of the deep part introduces bias into θ estimates. The non-parametric φ(·) is unobservable → can't validate how well the deep part learns it.

### 3.2 Two-Stage Approach (Main Contribution)

Addresses regularization bias via **Neyman-orthogonal score functions** (Double ML / DML framework adapted for Poisson).

**Key property** (Neyman orthogonality): the score function for estimating θ should be locally insensitive to small perturbations in the nuisance parameter φ around its true value.

**Stage 1** — estimate nuisance parameters using ML:
- Fit ê(X) = E[P|X] using deep neural network (price residualization)
- Fit φ̂(X) using ML (baseline demand residualization)
- Compute residuals: ΔP = P − ê(P|X), ΔY as transformed residuals

**Stage 2** — estimate θ using Bayesian Dynamic GLM:
- Use residualized quantities from Stage 1 in a parametric Poisson model
- Fit θ via **variational Bayesian** approach with Laplace approximation
- Online/streaming updates as new data arrives
- Natural uncertainty quantification for downstream use in pricing optimization

**Why Bayesian?**
- Built-in uncertainty estimates → useful for exploration (UCB-style pricing)
- Handles time-varying elasticities naturally
- Computationally efficient: sequential updates, no full refit needed

---

## 4 Price Optimization and RL Connection

Given estimated θ̂ with uncertainty, the firm can:
- Compute expected revenue-maximizing price for each request
- Use **UCB-style exploration**: occasionally test higher/lower prices proportional to uncertainty in θ̂
- Frame as Reinforcement Learning problem: observe demand response, update θ̂, repeat

The uncertainty estimates from the Bayesian stage 2 are key inputs for safe price experimentation.

---

## 5 Results

### Simulation Study
- Generates synthetic airline data with known ground-truth elasticity
- Tests: single-product scenarios, multi-product with confounders, missing covariates

| Method | Estimation Error (θ) |
|--------|---------------------|
| Direct (Wide & Deep) | ~25% |
| **Two-stage approach** | **~4%** |

The two-stage approach is robust even when the nuisance model is misspecified or subject to regularization.

### Real Airline Data
- Qualitative comparison confirms two-stage approach gives more stable, interpretable elasticity parameters
- Day-of-week and time-of-day elasticities match domain expectations (weekdays have higher WTP for business routes, mornings/evenings premium)

---

## 6 Key Contributions

1. **Poisson semi-parametric model** for demand response with feature-dependent price sensitivity
2. **Two-stage estimation** extending Double ML (Chernozhukov et al., 2018) to Poisson/non-linear settings
3. **Bayesian dynamic GLM** for robust, interpretable, uncertainty-quantified elasticity estimation
4. **Production-relevant framework**: interpretable, validatable, streaming-compatible
5. Reduces estimation error from 25% → 4% in simulation

---

## Relation to Literature

- **Double Machine Learning (DML)**: Chernozhukov et al. (2018) — this paper extends DML from partially-linear/additive-noise models to Poisson demand models.
- **IV methods**: Two-Stage Least Squares (2SLS) — IVs are hard to find in airlines; this paper avoids IV by leveraging observable confounders + orthogonalization.
- **Causal forests / CATE methods**: focus on binary treatments; this paper handles continuous price treatment.

---

## Key References

- Chernozhukov et al. (2018) — *Double/Debiased Machine Learning* (DML)
- Chiang, Chernozhukov et al. (2022) — *Concentrating-out approach for Neyman orthogonal scores in non-linear models*
- Den Boer (2015) — *Dynamic pricing and learning* (survey)
- Hartford et al. (2017) — *Deep IV: neural network IV estimation*
- Xu et al. (2021) — *Feature-based dynamic pricing*
