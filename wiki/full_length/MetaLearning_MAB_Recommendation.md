---
title: A Hybrid Meta-Learning and Multi-Armed Bandit Approach for Context-Specific Multi-Objective Recommendation Optimization
source: https://arxiv.org/abs/2409.08752
author:
  - "[[Tiago Cunha]]"
  - "[[Andrea Marchini]]"
published: 2024-10-01
created: 2026-04-17
description: Juggler-MAB combines Expedia's Juggler meta-learning framework with contextual MABs for real-time, context-specific weight adjustments in multi-stakeholder hotel recommendations.
tags:
  - clippings
  - bandits
  - recommendation-systems
  - meta-learning
  - multi-objective
---

## A Hybrid Meta-Learning and MAB Approach for Context-Specific Multi-Objective Recommendation Optimization

### Tiago Cunha, Andrea Marchini (Expedia Group) — RecSys SURE Workshop, October 2024

---

## Abstract

Online marketplaces must balance multiple stakeholders: users (relevance), providers (compensation), and the platform (revenue). This paper introduces **Juggler-MAB**, combining:
- **Juggler**: a meta-learning model that predicts optimal utility/compensation weights per search query
- **MAB**: a contextual bandit that makes real-time fine-grained weight corrections based on device, brand, and geo context

Evaluated on 0.6M searches from Expedia's lodging platform:
- +2.9% NDCG improvement
- −13.7% regret reduction
- +9.8% best arm selection rate

---

## 1 Introduction

**Problem**: Juggler (Cunha et al., 2021) selects from 5 pre-configured (utility weight, compensation weight) pairs using a meta-learning model trained offline. Limitations:
- Fixed options: cannot fine-tune for specific segments (device, brand)
- Infrequent retraining: slow to adapt to rapid changes in traffic patterns

**Proposed solution**: Add a contextual MAB layer on top of Juggler to provide fast, segment-specific corrections without retraining the base model.

**Research questions**:
1. Does MAB + Juggler outperform Juggler alone?
2. Are contextual features useful for MAB decisions?

---

## 2 Background

### Juggler Framework
- Meta-learning model predicting optimal (utility, compensation) weight pair per search query.
- 5 pre-configured options: lower/lower, lower/higher, neutral/neutral, higher/lower, higher/higher.
- Deployed in production at Expedia's Lodging Ranking stack.
- Trained offline via simulation; infrequent retraining cycles.

### Multi-Armed Bandits
- Balance exploration/exploitation in sequential decision-making.
- Used in recommender systems for exploration-exploitation dilemma.
- AdaptEx SDK (Black et al., 2023): Expedia's self-service contextual bandit platform.

---

## 3 Juggler-MAB Architecture

**Two-stage scoring**:

> sortScore = (w^Juggler_utility + w^MAB_utility) · utilityScore + (w^Juggler_comp + w^MAB_comp) · compensationScore

Stage 1 — **Juggler**: predicts coarse (w_utility, w_comp) from search context.
Stage 2 — **MAB**: applies fine-grained corrective weights (w^MAB_utility, w^MAB_comp) based on contextual features.

**MAB formulation**:
- Arms: pairs (w^MAB_utility, w^MAB_comp) ∈ {−0.3, 0.0, 0.3} × {−0.2, 0.0, 0.2} → 9 arms
- Context: device type, brand, geographic category
- Reward: NDCG (proxy for conversion rate)
- Goal: max_π E[Σ rₜ(xₜ, π(xₜ))]

**Design choices**:
- Contextual features selected for identifying under-performing Juggler segments
- NDCG as single-dimension reward (limitation: ignores compensation)
- Exploration: Thompson Sampling, ε-greedy

---

## 4 Experimental Setup

- **Dataset**: 0.6M searches, 31 consecutive days, Expedia lodging platform
- **Infrastructure**: AWS i3.4xlarge EC2, lodging MAB simulator (replay historical data)
- **Evaluation**: daily reward updates; 22K searches/day on average

**Bandits tested**:
- Gaussian Thompson (GT): classical, no context
- ε-greedy (ε=0.1, 0.3): classical, no context
- RLS (Recursive Least Squares + Thompson Sampling): contextual linear bandit
  - Variants: brand only, device only, geo only, combinations

**Metrics**: avg reward, avg regret, % best arm selections

---

## 5 Results

| Bandit | Avg Reward | Avg Regret | Best Arm % |
|--------|-----------|-----------|------------|
| Juggler (baseline) | 0.1776 | 0.0373 | 75.2% |
| GT | 0.1791 | 0.0358 | 78.7% |
| ε-greedy (0.3) | 0.1811 | 0.0339 | 80.9% |
| ε-greedy (0.1) | 0.1824 | 0.0325 | 82.2% |
| **RLS_brand** | **0.1827** | **0.0322** | **82.5%** |
| RLS_device | 0.1822 | 0.0327 | 82.0% |
| RLS_geo | 0.1825 | 0.0325 | 82.3% |
| RLS_geo+brand | 0.1827 | 0.0323 | 82.5% |

**Key findings**:
- All MAB variants outperform Juggler baseline.
- Best contextual bandit: **RLS_brand** (single feature context).
- More features ≠ better: adding more context didn't improve beyond brand alone.
- ε-greedy (0.1) is a strong non-contextual baseline.

**Business impact (vs. Juggler)**:
- Average daily price: −0.86 (lower prices → higher relevance → higher conversion expected)
- Guest rating: +0.06
- Star rating: +0.08
- Margin %: −0.005 (compensation decreases — risk factor for marketplace health)

**Interpretation**: MAB learns to downweight compensation and increase/decrease relevance depending on context. Trade-off: improved NDCG but lower margin — needs A/B test validation in production.

---

## 6 Conclusions and Future Work

**Contributions**:
- First integration of meta-learning + MAB for multi-stakeholder recommendation
- Empirical validation on large-scale real data

**Limitations**:
- Single-objective reward (NDCG) ignores compensation objective
- Simulation uses deterministic logging policy → off-policy bias

**Future directions**:
1. Online A/B testing in production
2. Dynamic arm space (adaptive arm generation)
3. Multi-objective reward function
4. Fairness constraints for provider segments
5. Long-term value optimization (sequential RL)

---

## Key References

- Cunha, Partalas & Nguyen (2021) — *Juggler: multi-stakeholder ranking with meta-learning*
- Black et al. (2023) — *AdaptEx: a self-service contextual bandit platform*
- Lattimore & Szepesvári (2020) — *Bandit Algorithms* (textbook)
- Li et al. (2010) — *Contextual-bandit approach to personalized news recommendation*
