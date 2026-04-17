---
title: "Uplift Modeling for Multiple Treatments with Cost Optimization"
source: "https://arxiv.org/abs/1908.05372"
author:
  - "[[Zhenyu Zhao]]"
  - "[[Totte Harinen]]"
published: 2020-03-26
created: 2026-04-17
description: "Extends meta-learner uplift models (X-Learner, R-Learner) to multiple treatment groups and introduces a net value optimization framework that accounts for heterogeneous treatment costs."
tags:
  - "clippings"
  - "uplift-modeling"
  - "causal-inference"
  - "treatment-effects"
  - "marketing"
---

## Uplift Modeling for Multiple Treatments with Cost Optimization

### Zhenyu Zhao, Totte Harinen (Uber Technologies) — 2020

---

## Abstract

Uplift modeling estimates treatment effects at the individual or subgroup level to enable personalized treatment assignment. This paper extends standard uplift models to support:
1. **Multiple treatment groups** (beyond the standard 2-arm trial)
2. **Different costs per treatment** (impression cost + triggered cost)

The authors extend the X-Learner and R-Learner meta-learners to the multi-treatment setting and propose a **net value optimization** framework. They evaluate on synthetic and real data (Uber promotion campaigns) and describe a production platform implementation.

---

## I Introduction

**Uplift modeling** vs. standard predictive modeling: uplift predicts the *incremental* effect of a treatment (CATE), not whether a user will convert regardless. A user with high baseline conversion probability may have low uplift — they would convert anyway.

**Key practical challenge**: most industry experiments have multiple treatment groups (different channels, promotion types, product versions). Existing methods focus on 2-arm designs.

**Two complicating factors addressed**:
- Multiple treatment groups (>1 treatment vs. control)
- Heterogeneous treatment costs (different channels cost different amounts; promotions have triggered costs)

**Contributions**:
- Extend X-Learner and R-Learner to multiple treatments
- Propose net value CATE framework incorporating costs
- Empirical evaluation on synthetic + real data
- Platform design for Uber-scale deployment

---

## III Uplift as Causal Inference

Using the **Neyman-Rubin potential outcomes framework**:

> Yᵢ(1) − Yᵢ(0) = individual causal effect

> τ(xᵢ) = E[Yᵢ(1) − Yᵢ(0) | X = xᵢ] = **CATE**

Uplift modeling = using ML to estimate CATE. Two main approaches:
1. **Meta-learners**: combine standard ML models to estimate CATE (easy to implement, fast).
2. **Modified algorithms**: e.g. Causal Random Forest (Wager & Athey), which modifies the splitting criterion to maximize treatment effect heterogeneity.

This paper focuses on meta-learners.

---

## IV Meta-Learners (2-arm baseline)

### Two Model Approach
> τ̂(xᵢ) = µ̂₁(xᵢ) − µ̂₀(xᵢ)

Fit separate models for treated and control. Simple but underperforms X/R-Learner.

### X-Learner (Künzel et al., 2017)
1. Fit outcome models µ̂₀(x), µ̂₁(x).
2. Compute pseudo-effects: D̃ᵢ⁰ = µ̂₁(x) − Yᵢ (for control) and D̃ᵢ¹ = Yᵢ − µ̂₀(x) (for treated).
3. Fit τ̂₀(x) and τ̂₁(x) on pseudo-effects.
4. Combine: τ̂(x) = ê(x)τ̂₀(x) + (1 − ê(x))τ̂₁(x) where ê(x) is the propensity score.

Advantage: leverages treatment/control data asymmetry efficiently.

### R-Learner (Nie & Wager, 2017)
Minimize residualized loss:
> τ̂(·) = argmin_τ { Σᵢ [(Yᵢ − m̂(Xᵢ)) − (Wᵢ − ê(Xᵢ))τ(Xᵢ)]² + Λ(τ) }

Where m̂(x) = E[Y|X=x] and ê(x) = propensity score. Neyman-orthogonal construction → robust to nuisance parameter estimation error.

---

## V Uplift for Multiple Treatments with Costs

### A. Multiple Treatment Groups

**Setting**: control t₀ and m treatments t₁,...,tₘ.

**Extended X-Learner**: for each treatment group tj, estimate µ_tj(x) and compute pseudo-effects pairwise against control. Propensity score becomes:

> τ̂_tj(x) = [ê_tj / (ê_tj + ê_t₀)] · τ̂_t₀(x) + [ê_t₀ / (ê_tj + ê_t₀)] · τ̂_tj(x)

Recommend treatment with highest predicted uplift for each user.

**Extended R-Learner**: same strategy — estimate propensity scores and mean outcomes for each group, plug into R-Learner minimization.

### B. Net Value Optimization

**Cost structure**:
- cₜⱼ: impression cost per treated user (e.g. channel cost per send)
- sₜⱼ: triggered cost per converted user (e.g. promotion discount redeemed)

**Net value CATE**:
> τ_tj(x, v, s_tj, c_tj) = E[(v − s_tj)Y_tj − (v − s_t₀)Y_t₀ − (c_tj − c_t₀) | X = x]

**Net value X-Learner**: modify pseudo-effects to incorporate costs:
> D̃ᵢ^{tj,t₀} = (v − s_tj)Y_tj − (v − s_t₀)µ_t₀(x) − (c_tj − c_t₀)

**Net value R-Learner**: modify the CATE objective to include (v − s)Y − (v − s̄)m̂ − (c − c̄) in place of Y − m̂.

---

## VII Empirical Evaluation

### Synthetic Data

| Model | Two-arm AUUC | Four-arm AUUC | Time (4-arm) |
|-------|-------------|---------------|--------------|
| R-Learner | 0.0185 | **0.0310** | 111s |
| X-Learner | **0.0205** | 0.0305 | 125s |
| Two Model | 0.0135 | 0.0283 | 54s |
| KL (decision tree) | 0.0195 | **0.0331** | 1460s |
| CTS | 0.0153 | 0.0270 | 1254s |

Meta-learners (X and R) are competitive with tree-based methods and dramatically faster.

**Net value optimization**: standard meta-learners optimize conversion without costs → sub-optimal net value. Net value X/R-Learner outperform all single-treatment strategies and standard models when costs vary.

### Real Data (Uber Promotion Campaign)
- 600K training / 800K testing observations, 139 features
- 1 control, 2 treatments with different impression + triggered costs
- **Net value models significantly outperform standard uplift models and original experiment groups**
- Standard uplift models can *hurt* net value if they send high-cost promotions to users with low incremental response

---

## VIII Platform Implementation (Uber)

**End-to-end ML platform components**:
1. **Training data processor**: merges treatment tags, outcome labels, user features.
2. **Model training**: fits multiple uplift models; selects best by AUUC cross-validation.
3. **Model storage**: trained model persisted for inference.
4. **Prediction module**: scores target cohort; outputs uplift per user per treatment.
5. **Deployment**: online service (real-time scoring) or offline batch jobs.

User configures: target metric Y, features, experiment tag, cohort filter, cost/value assignments, targeting cut-point (e.g. top 50% by uplift score).

---

## IX Conclusion

Key takeaways:
- Meta-learners (X and R) extend naturally to multiple treatments with good accuracy and fast computation.
- **Net value optimization is critical when treatments have different costs** — optimizing conversion alone leads to sub-optimal business outcomes.
- Practically deployed at Uber as a horizontal ML platform.

**Future work**: extend to regression problems (continuous incremental value); study data quality challenges at scale.

---

## Key References

- Künzel et al. (2017) — *Meta-learners for estimating heterogeneous treatment effects* (X-Learner)
- Nie & Wager (2017) — *Quasi-Oracle estimation of heterogeneous treatment effects* (R-Learner)
- Rzepakowski & Jaroszewicz (2012) — *Decision trees for uplift modeling with multiple treatments*
- Wager & Athey (2015) — *Estimation and inference of heterogeneous treatment effects using random forests*
- Gutierrez & Gerardy (2016) — *Causal inference and uplift modeling: a review*
