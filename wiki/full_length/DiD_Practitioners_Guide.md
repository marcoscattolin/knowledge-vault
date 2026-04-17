---
title: "Difference-in-Differences Designs: A Practitioner's Guide"
source: "https://arxiv.org/abs/2503.13323"
author:
  - "[[Andrew Baker]]"
  - "[[Brantly Callaway]]"
  - "[[Scott Cunningham]]"
  - "[[Andrew Goodman-Bacon]]"
  - "[[Pedro H. C. Sant'Anna]]"
published: 2025-03-18
created: 2026-04-17
description: "A unified framework for conducting DiD studies, covering weights, covariates, multiple periods, and staggered treatments — built around the idea that complex DiD designs are aggregations of 2×2 building blocks."
tags:
  - "clippings"
  - "causal-inference"
  - "econometrics"
  - "DiD"
---

## Difference-in-Differences Designs: A Practitioner's Guide

### Baker, Callaway, Cunningham, Goodman-Bacon, Sant'Anna — March 2025

---

## Abstract

Difference-in-Differences (DiD) is arguably the most popular quasi-experimental research design. Its canonical form, with two groups and two periods, is well-understood. However, empirical practices can be ad hoc when researchers go beyond that simple case. This article provides an organizing framework for discussing different types of DiD designs and their associated DiD estimators. It discusses covariates, weights, handling multiple periods, and staggered treatments.

---

## 1 Introduction

Dating to the 1840s, DiD is now the most common research design for estimating causal effects in the social sciences. A basic DiD design requires two time periods and two groups — one treated and one not. The DiD estimate equals the change in outcomes for the treated group minus the change in outcomes for the untreated group. Under the **parallel trends assumption**, this estimates the average treatment effect on the treated (ATT).

In practice, researchers apply DiD to more complex settings: multiple periods, staggered treatment timing, continuous treatments, covariates. For years, the default was **Two-Way Fixed Effects (TWFE)** regression. Recent research has shown that TWFE can fail badly when treatment effects are heterogeneous — producing estimates that are misleading in magnitude or even have the wrong sign.

**The central insight of this paper**: even complex DiD designs are aggregations of simple **2×2 building blocks** — comparisons between one group whose treatment changes and another whose does not. This unifies a wide variety of DiD designs and guides methodological choices.

The paper follows a **forward-engineering philosophy**: start from the causal question, define target parameters, state identification assumptions explicitly, then derive estimation techniques. This contrasts with the common "reverse-engineering" approach of starting from a regression and post-hoc justifying it.

---

## 2 Running Example: Medicaid and Mortality

The paper uses the ACA Medicaid expansion as a running example. States expanded Medicaid at different times (2014–2023), with some never expanding. The outcome is the crude adult mortality rate (ages 20–64) per 100,000, by county, 2009–2019 (2,604 counties).

This example illustrates three core complications of modern DiD studies:
- **Weights**: California accounts for 4.5% of 2014-expansion states but 23% of adults — weighting changes results meaningfully.
- **Covariates**: Expansion states differ systematically (e.g., fewer Southern states), making raw parallel trends implausible.
- **Staggered timing**: States expanded in different years, defining multiple treatment cohorts.

---

## 3 The Canonical 2×2 DiD

### 3.1 Target Parameter: the ATT

The first step is defining the causal quantity of interest using the **potential outcomes framework**. The target parameter is the **Average Treatment Effect on the Treated** at time *t*:

> ATT(t) = E[Y_{i,t}(1) − Y_{i,t}(0) | D_i = 1]

The counterfactual E[Y_{i,t}(0) | D_i = 1] is never observed — this is the fundamental identification challenge.

Weighting enters early: if interest is in the average effect per county ("laboratories of democracy"), use equal weights. If interest is in the effect on the average adult, use population weights. When treatment effects are heterogeneous and correlated with weights, these parameters differ meaningfully.

### 3.2 Identification: Parallel Trends

The **parallel trends assumption** states that, absent treatment, the average outcome evolution would have been the same across treated and comparison groups:

> E[Y_{t2}(0) | D=1] − E[Y_{t1}(0) | D=1] = E[Y_{t2}(0) | D=0] − E[Y_{t1}(0) | D=0]

This assumption is **untestable** in the post-period but can be partially falsified using pre-period data.

### 3.3 Estimation: 4 Means or One Regression?

The DiD estimator is the difference of four group-time means:

> DiD = (Ȳ_{treated,post} − Ȳ_{treated,pre}) − (Ȳ_{control,post} − Ȳ_{control,pre})

This is numerically equivalent to the coefficient on the interaction term in a TWFE regression. In the 2×2 case, both are valid. Problems arise only in more complex designs.

---

## 4 Incorporating Covariates into 2×2 DiD

### 4.1 Covariate Balance

Before adjusting for covariates, check whether they are balanced between treated and control groups in the pre-period. Imbalance motivates conditional parallel trends.

### 4.2 Conditional Parallel Trends

When unconditional parallel trends is implausible, one can condition on covariates *X*:

> E[ΔY(0) | X, D=1] = E[ΔY(0) | X, D=0]

This is more credible when covariates explain cross-group differences in outcome trends.

### 4.3–4.4 Estimation Strategies with Covariates

Three semiparametrically efficient estimators target the ATT under conditional parallel trends:

- **Regression Adjustment (RA)**: model E[ΔY | X, D=0] and impute counterfactual trends.
- **Inverse Probability Weighting (IPW)**: reweight control units by propensity score to match treated.
- **Doubly Robust (DR)**: combines RA and IPW — consistent if either model is correctly specified.

**TWFE with covariates is not recommended**: it implicitly estimates a weighted average of conditional ATTs with weights that may be negative and are determined by the data, not the researcher.

### 4.5 Heterogeneity Analysis

Subgroup ATTs (by region, demographics, etc.) are natural extensions of the forward-engineering approach — simply redefine the target population and apply the same steps.

---

## 5 DiD with Multiple Time Periods

### 5.1 Simple Event Studies (2×T)

With a single treatment group and multiple periods, the 2×2 building block generalizes to a series of 2×2 DiDs — one per time period. The **event study plot** shows ATT(t) for each period t, with:
- **Post-period estimates**: test the size and dynamics of the treatment effect.
- **Pre-period estimates ("pre-trends")**: falsification tests for parallel trends — should be near zero.

Key estimation note: the TWFE event study and the direct 2×2 approach give identical estimates in the 2×T case. However, collapsing to a single post-period dummy (Di,t) gives a different — and generally biased — scalar summary.

### 5.2 Staggered Treatment Adoption (G×T)

When units adopt treatment at different times, each cohort *g* defines a distinct treatment group with its own event study. The **group-time ATT** is:

> ATT(g, t) = E[Y_{i,t}(g) − Y_{i,t}(∞) | G_i = g]

**Identification**: Parallel trends must hold between each cohort *g* and the "clean" comparison group — either never-treated units or not-yet-treated units at time *t*.

**Aggregation**: Individual ATT(g,t) parameters can be aggregated into interpretable summaries:
- Average post-treatment effect (overall ATT)
- Event-time averages (effect at each relative time to treatment)
- Cohort-specific ATTs

**Preferred estimators**: Callaway & Sant'Anna (2021), Sun & Abraham (2021), Borusyak et al. (2024), Gardner (2021). All share the forward-engineering approach.

### 5.3 Limitations of TWFE in Staggered Designs

In staggered settings, TWFE uses **already-treated units as controls** for later-treated units. When treatment effects are heterogeneous or dynamic, this creates negative weights on some 2×2 comparisons, potentially biasing the TWFE coefficient toward the wrong sign.

> "TWFE has well-understood, potentially serious, and easily remedied problems, and we do not recommend using it."

---

## 6 Conclusion: The Forward-Engineering Checklist

The paper proposes **8 steps** for any DiD study:

1. **Define target parameters** — use potential outcomes notation to specify what causal quantity you want.
2. **State identification assumptions** — be explicit about parallel trends, no-anticipation, and overlap.
3. **Justify the estimation method** — RA, IPW, or DR; state the modeling restrictions.
4. **Discuss sources of uncertainty** — sampling vs. design-based inference; cluster structure.
5. **Estimate** — apply the method derived in steps 1–4.
6. **Conduct sensitivity analysis** — test robustness to plausible violations of identifying assumptions.
7. **Conduct heterogeneity analysis** — subgroup parameters where relevant.
8. **Keep learning** — if DiD assumptions are implausible, explore other designs.

---

## Appendix: Extensions

- **A.1 Treatments that turn on and off** — requires tracking full treatment paths as potential outcomes.
- **A.2 Continuous/multi-valued treatments** — dose-specific ATT curves; stronger parallel trends required.
- **A.3 Triple Differences (DDD)** — relaxes parallel trends by exploiting partition-specific variation; cannot generally be expressed as a difference of two DiDs.
- **A.4 Distributional DiD** — targets quantile treatment effects or distributional parameters instead of ATT.
- **A.5 Repeated cross-sections / unbalanced panels** — requires stationarity assumption to pool across periods; compositional changes introduce bias if ignored.

---

## Key References

- Callaway & Sant'Anna (2021) — *Difference-in-Differences with multiple time periods*, Journal of Econometrics
- Goodman-Bacon (2021) — *DiD with variation in treatment timing*, Journal of Econometrics
- Sun & Abraham (2021) — *Estimating dynamic treatment effects in event studies with heterogeneous treatment*
- Borusyak, Jaravel & Spiess (2024) — *Revisiting Event Study Designs*
- de Chaisemartin & D'Haultfoeuille (2020) — *Two-way fixed effects estimators with heterogeneous treatment effects*, AER
- Roth, Sant'Anna, Bilinski & Poe (2023) — *A Guide to DiD Designs* (survey)
- Sant'Anna & Zhao (2020) — *Doubly Robust DiD*
