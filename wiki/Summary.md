# Summaries

Indice sintetico degli articoli nella vault, organizzato per argomento.

---

## Causal Inference & Difference-in-Differences

### [[DiD_Practitioners_Guide|Difference-in-Differences Designs: A Practitioner's Guide]]
*Baker, Callaway, Cunningham, Goodman-Bacon, Sant'Anna — 2025*

Framework unificato per studi DiD basato sull'idea che qualsiasi design complesso è un'aggregazione di building block 2×2. Copre ATT, parallel trends, covariates (RA/IPW/DR), event studies, staggered adoption e fallimenti di TWFE con treatment effects eterogenei. Include una checklist in 8 passi.

→ [[full_length/DiD_Practitioners_Guide|Leggi]]

---

### [[Causal Inference in the Wild|Causal Inference in the Wild: Elasticity Pricing]]
*Lars Roemheld — 2021*

Esempio industriale di stima dell'elasticità al prezzo tramite Double Machine Learning (DML). Random Forest residualizza prezzo e quantità rispetto ai confounders (stagionalità, qualità prodotto); la regressione sui residui produce un'elasticità causalmente valida. Controllare i confounders sposta la stima da −0.6 a −1.9, ribaltando la raccomandazione di pricing.

→ [[full_length/Causal Inference in the Wild|Leggi]]

---

## Uplift Modeling

### [[A Quick Uplift Modeling Introduction|A Quick Uplift Modeling Introduction]]
*Shelby Temple — 2020*

Guida introduttiva all'uplift modeling: perché stimare l'effetto incrementale del trattamento (CATE) differisce dal predire la probabilità di conversione. Introduce i quattro tipi di clienti (persuadables, sure things, lost causes, sleeping dogs), la valutazione tramite uplift curve e i principali approcci implementativi.

→ [[full_length/A Quick Uplift Modeling Introduction|Leggi]]

---

### [[Uplift_Modeling_Multiple_Treatments|Uplift Modeling for Multiple Treatments with Cost Optimization]]
*Zhenyu Zhao, Totte Harinen (Uber) — 2020*

Estende X-Learner e R-Learner a setting multi-trattamento e introduce un framework di net value optimization che incorpora costi di impression e triggered. Su un dataset reale Uber (600K+ osservazioni, 139 feature), i modelli net value superano significativamente i meta-learner standard che ignorano i costi.

→ [[full_length/Uplift_Modeling_Multiple_Treatments|Leggi]]

---

## Contextual Bandits

### [[An Overview of Contextual Bandits|An Overview of Contextual Bandits]]
*Ugur Yildirim — 2024*

Panoramica accessibile dei contextual bandit come framework per la personalizzazione adattiva. Copre il dilemma explore-exploit, i principali algoritmi (LinUCB, Thompson Sampling), il confronto con A/B testing e supervised learning, e considerazioni pratiche per il deployment in produzione.

→ [[full_length/An Overview of Contextual Bandits|Leggi]]

---

### [[Contextual_Bandits_Lecture|Contextual Bandits — Lecture Notes CSE599i]]
*Lalit Jain et al. — UW, 2018*

Note di lezione su teoria dei contextual bandit: setup, off-policy evaluation (IPS), C-Exp3, Exp4, LinUCB, ε-greedy ed ILOVETOCONBANDITS. Include dimostrazioni dei regret bound e tabella comparativa degli algoritmi per setting stocastico/avversariale con policy set piccolo/grande.

→ [[full_length/Contextual_Bandits_Lecture|Leggi]]

---

### [[MetaLearning_MAB_Recommendation|Juggler-MAB: Hybrid Meta-Learning and MAB for Multi-Objective Recommendation]]
*Tiago Cunha, Andrea Marchini (Expedia) — RecSys 2024*

Sistema a due stadi che combina il ranker meta-learning Juggler con un MAB contestuale per correzioni real-time sui pesi utilità/compensazione. Testato su 0.6M ricerche hotel: il bandit contestuale RLS_brand ottiene +2.9% NDCG, −13.7% regret e +9.8% best arm selection rispetto al baseline Juggler.

→ [[full_length/MetaLearning_MAB_Recommendation|Leggi]]

---

## Dynamic Pricing

### [[ML_Price_Sensitivity_Airline|ML Framework for Robust Price-Sensitivity Estimation — Airline Pricing]]
*Kumar, Boluki, Isler, Rauch, Walczak (PROS) — 2022*

Framework semi-parametrico a due stadi per stimare price elasticity da dati osservazionali senza informazioni sui no-purchase. Stage 1: DNN per stimare parametri di disturbo; Stage 2: GLM Bayesiano dinamico con score Neyman-ortogonali per la price sensitivity. Riduce l'errore di stima dal 25% al 4% in simulazione.

→ [[full_length/ML_Price_Sensitivity_Airline|Leggi]]

---

### [[Dynamic_Pricing_Airline_Ancillaries|Feature-Based Dynamic Pricing of Airline Ancillaries]]
*Muhammed Memedi (KTH) — 2021*

Tesi magistrale che applica MAB contestuali al pricing dinamico degli ancillari aerei. Contributo chiave: strategia "extended play" che sfrutta la monotonicità del WTP per estrarre segnale da più price point per sessione. Propone DEGLMUCB e OORMLP-β. I contextual bandit con extended play risultano i più performanti e robusti.

→ [[full_length/Dynamic_Pricing_Airline_Ancillaries|Leggi]]
