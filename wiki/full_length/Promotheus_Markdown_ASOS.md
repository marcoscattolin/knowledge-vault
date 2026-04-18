---
title: "Promotheus: An End-to-End Machine Learning Framework for Optimizing Markdown in Online Fashion E-commerce"
source: "https://arxiv.org/abs/2207.01137"
author:
  - "[[Eleanor Loh]]"
  - "[[Jalaj Khandelwal]]"
  - "[[Brian Regan]]"
  - "[[Duncan A. Little]]"
published: 2022-08-14
created: 2026-04-17
description: "Framework end-to-end per ottimizzare il markdown (sconti promozionali) in un retailer fashion online (ASOS), con un algoritmo supply-side 'cold start' (Ithax) e un sistema completo di price elasticity optimization (Promotheus), validati in un test online con +86% di profittabilità rispetto al pricing manuale."
tags:
  - "clippings"
  - "dynamic-pricing"
  - "markdown-pricing"
  - "retail"
  - "machine-learning"
  - "e-commerce"
  - "price-elasticity"
---

## Promotheus: An End-to-End Machine Learning Framework for Optimizing Markdown in Online Fashion E-commerce

### Eleanor Loh, Jalaj Khandelwal, Brian Regan, Duncan A. Little (ASOS.com) — 2022

---

## Abstract

Il paper presenta due sistemi end-to-end per la gestione del markdown (prezzi promozionali scontati) in un contesto di fashion e-commerce, entrambi sviluppati e deployati su ASOS.com. Il primo sistema, **Ithax**, adotta una strategia di pricing supply-side senza stima della domanda, e funziona come soluzione "cold start" per raccogliere dati di pricing. Il secondo, **Promotheus**, è un framework completo che utilizza modelli di price elasticity per ottimizzare i prezzi. Entrambi i sistemi superano significativamente le decisioni manuali dei team operativi: in un test online controllato, Promotheus ottiene un miglioramento dell'**86%** in profittabilità e Ithax del **79%** rispetto alle strategie manuali.

---

## 1 Introduzione

Il markdown è cruciale nel retail fashion, dove il buyer acquista lo stock mesi prima senza conoscere i trend futuri. In fashion e-commerce, il markdown può rappresentare una quota sostanziale del budget operativo.

**Gap principale affrontato**: la maggior parte dei retailer ricade su strategie rule-based dei team operativi, rinunciando ai guadagni realizzabili con il machine learning. Le difficoltà sono molteplici:
- **Partial information problem**: i modelli di price elasticity lavorano in un contesto di inferenza controfattuale — i prezzi non applicati storicamente non hanno outcome osservabili.
- **Spurious correlations**: nella distribuzione storica, gli sconti più profondi sono stati applicati ai prodotti con peggiori performance, per cui i modelli naive imparano la correlazione sbagliata (prezzo più basso → meno vendite).
- **Completezza del modello**: la moda è soggetta a eventi estremi (virality social media) difficili da catturare completamente.

**Contributi principali**:
1. Ithax: algoritmo supply-side per markdown optimization senza demand estimation (cold start)
2. Promotheus: framework completo con price elasticity e depth optimization
3. Procedure di validazione robuste per il partial information problem
4. Framework per la tactical agility dei team operativi
5. Validazione online in un test reale (non solo offline)

---

## 2 Background e Sfide

### Price Optimization come Inferenza Controfattuale

La previsione della domanda nel price optimization **non è** un semplice problema di supervised learning: il modello deve generalizzare a prezzi mai applicati storicamente. I valori attesi per prezzi alternativi sono fondamentalmente non osservabili nei dati storici.

**Implicazione**: l'offline evaluation è necessariamente limitata; il gold standard è il test online.

### Problemi di Specificazione del Modello

- **Negative elasticity requirement**: le curve di elasticità devono essere non-decrescenti (meno prezzo → più vendite), eccetto per Veblen goods. I modelli naive violano spesso questo requisito a causa della correlazione storica inversa.
- **Outlier events**: eventi di picco (virality) distorcono i modelli tree-based, elevando le stime dell'intera foglia. Soluzione adottata: winsorizzazione del target.
- **Completeness**: variabili difficilmente catturabili (es. trend social) possono dominare la domanda di specifici prodotti.

---

## 3 Supply-Side Algorithm: Ithax

### Obiettivo

Costruire un insieme di prodotti `P_t` da includere in un evento markdown, con profondità di sconto assegnate, rispettando target di **stock value** e **stock depth** definiti dal business.

### Metriche Chiave

> **Discount Depth** `d_{p,t}`: percentuale di sconto per il prodotto `p` nel periodo `t`

> **Stock Value** `V(P_t) = Σ_{p} full_price_p * stock_units_{p,t}`: valore a prezzo pieno dello stock incluso nel markdown

> **Stock Depth** `M(P_t) = 1 - Σ(1-d_p)*price_p*units_p / Σ price_p*units_p`: profondità media ponderata degli sconti

> **Cover** `c_{p,t} = stock_units_{p,t} / sold_units_{p,t}`: settimane di copertura dello stock (proxy per performance del prodotto — cover alto = prodotto che vende poco)

### Logica dell'Algoritmo

Ithax mappa i prodotti in **cover bands** (fasce di cover) con profondità di sconto crescenti: prodotti con cover più alto (performance peggiore) ricevono sconti più profondi. L'algoritmo usa una ricerca binaria iterativa per aggiustare i confini delle cover bands fino a rispettare i target `V*` e `M*`.

**Funzione obiettivo** (minimizzazione biobettivo):

> `f1(P_t) = |V(P_t) - V*| / V*` (deviazione relativa dallo stock value target)
> `f2(P_t) = |M(P_t) - M*|` (deviazione assoluta dallo stock depth target)

Convergenza considerata raggiunta quando `f1 < 0.05` e `f2 < 0.005`.

**Workflow (Algorithm 1)**:
1. Inizializza la cover-depth mapping `ν_0`
2. **Depth allocation step**: popola `P_hat` rispettando `V*`, prioritizzando i prodotti con cover maggiore
3. **Boundary adjustment step**: valuta `M(P_hat)` rispetto a `M*`; se `M > M*` riduce i prodotti ad alta profondità (Algorithm 3); se `M < M*` aggiunge prodotti ad alta profondità o li trasferisce da depth bassa ad alta (Algorithm 4)
4. Ripete fino a convergenza (tipicamente < 25 iterazioni, < 30 minuti)

### Estensioni per Tactical Agility

- **Inclusions/Exclusions**: i team operativi possono forzare o escludere prodotti specifici
- **Product Group Priority**: si può suddividere il target `V*` per gruppo di prodotto, eseguendo Algorithm 2 separatamente per ciascun gruppo

---

## 4 Promotheus: Full Price Optimization

### Demand Forecasting Model

Il modello predice **log-sales** in funzione del discount depth e di un vettore di covariate:

> `log(s_{p,t}) = f(d_{p,t}, x_{p,t})`

**Scelte implementative**:
- **Algoritmo**: Gradient Boosted Trees (LightGBM) — outperforma regressioni lineari (WAPE ~0.68) e reti neurali (WAPE ~0.71); best model WAPE ~0.53
- **Monotonic constraint** sul depth: impedisce al modello di apprendere relazioni spurie positive tra sconto e vendite
- **Feature set** (n=25): caratteristiche prodotto, storico prezzi, performance recente per canale di vendita, stagionalità
- **Winsorizzazione** al 0.5° percentile del target: elimina l'influenza degli outlier estremi (eventi virali) mantenendo stabilità del modello — il modello viene winsorizzato in training ma validato su log-sales non winsorizzate

### Validazione con Partial Information

Metrica principale: **WAPE (Weighted Absolute Percentage Error)**:

> `WAPE = Σ|ŝ_i - s_i| / Σ s_i`

**Procedura di cross-validazione**: Time-Series K-Fold con K=10 fold, holdout delle ultime 5 settimane per fold, sliding backwards nel tempo. Garantisce che il partial information problem sia sempre presente nel training/validation split e copre un'intera stagione retail.

Adottare questa procedura (vs. singola split point-in-time) è stata cruciale per ottenere modelli stabili nel tempo.

---

## 5 Depth Optimization

### Feasible Region Construction

Dopo aver stimato il modello, si limita lo spazio d'azione `π_p` ai soli depth dove il modello raggiunge WAPE accettabile (threshold=0.55). La WAPE viene calcolata out-of-sample con la time-series KFold, per gruppo di prodotto e profondità di sconto.

**Tabella 2 — WAPE per gruppo prodotto e depth** (regioni infeasible in grigio, soglia 0.55):

| Depth (norm.) | Gruppo A | Gruppo B | Gruppo C | Gruppo D |
|:---:|:---:|:---:|:---:|:---:|
| 0.2 | **0.570** | **0.579** | 0.397 | **0.844** |
| 0.3 | 0.460 | 0.453 | 0.474 | 0.449 |
| 0.4 | 0.458 | 0.466 | 0.496 | 0.441 |
| 0.5 | 0.450 | **0.507** | **0.505** | 0.445 |
| 0.6 | 0.496 | **0.526** | **0.540** | 0.485 |
| 0.7 | **0.505** | **0.509** | **0.566** | 0.461 |
| 0.8 | **0.641** | **0.632** | **0.626** | **0.537** |

*(I valori in grassivo superano la soglia 0.55 — regioni infeasibili)*

### Decision Making

Per ogni prodotto `p` nella feasible region `π_p`, il depth ottimale è:

> `d*_p = argmax_{d ∈ π_p} ŝ_{p,d} * ĝ_{p,d}`

dove `ĝ_{p,d}` è il profitto atteso unitario al depth `d`. L'obiettivo bilancia profittabilità e clearance (liquidazione dello stock).

---

## 6 Risultati

### 6.1 Performance di Ithax

Ithax ha centrato i target di stock depth e stock value per ogni evento markdown testato (>3 mesi, 4 eventi), con discrepanza sempre < 2%:

**Tabella 3 — Accuratezza Ithax su 4 markdown events**:

| Evento | Target Stock Depth | Achieved | Discrepancy | Target Stock Value | Achieved | Discrepancy |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| A | 0.444 | 0.446 | -0.45% | 0.875 | 0.8749 | +0.02% |
| B | 0.500 | 0.508 | -1.60% | 1.500 | 1.4991 | +0.06% |
| C | 0.500 | 0.499 | +0.12% | 1.475 | 1.4746 | +0.03% |
| D | 0.460 | 0.468 | -1.74% | 1.375 | 1.3745 | +0.04% |

*(Valori in unità arbitrarie per riservatezza commerciale)*

Comportamento coerente con le aspettative: i prodotti con cover più alto (peggiori performance) ricevono sconti più profondi. Convergenza sempre in < 25 iterazioni.

### 6.2 Demand Forecasting

- LightGBM: WAPE ≈ 0.53 (vs linear regression ≈ 0.68, neural nets ≈ 0.71)
- Online WAPE in linea con offline WAPE per gli stessi eventi markdown
- **Price elasticity strettamente negativa** per il 100% dei prodotti del catalogo (vendite non-decrescenti al diminuire del prezzo, grazie al monotonic constraint + winsorizzazione)

### 6.3 Business KPIs — Test Online

**Design del test**: split 50/50 del budget di stock value tra pricing manuale (operations team) e pricing algoritmico. Tra i prodotti algoritmici, 50% ulteriore split tra Promotheus (Full Optimization) e Ithax solo (Supply-side).

Test eseguito su tutti i mercati e canali disponibili. Metrica: **profitto per prodotto**. Analisi non-parametrica (dati non normali) con Mann-Whitney U test.

**Tabella 5 — Confronto pairwise di profittabilità**:

| Confronto | Uplift Mediane | Uplift Medie | P-value |
|:---|:---:|:---:|:---:|
| Full Optimization > Supply-side | +4.10% | +2.80% | p < 0.05 |
| Full Optimization > Manual | +86.60% | +31.30% | p < 0.001 |
| Supply-side > Manual | +79.30% | +27.70% | p < 0.001 |

I guadagni sono fortemente significativi. Il gap tra uplift di mediane e medie riflette la distribuzione non-normale della profittabilità per prodotto.

---

## 7 Conclusioni / Takeaway

**Messaggi principali**:
1. Un algoritmo supply-side puro (Ithax) supera già ampiamente il pricing manuale (+79%), anche senza stimare la domanda — utile come cold start e come baseline permanente.
2. Aggiungere price elasticity (Promotheus) porta ulteriori guadagni incrementali (+4% sulle mediane vs Ithax).
3. La validazione online è imprescindibile: l'offline evaluation è per definizione limitata in contesti di partial information.
4. Monotonic constraints + winsorizzazione sono scelte tecniche critiche per ottenere elasticità di segno corretto e modelli stabili.
5. La tactical agility (levers operativi come inclusions/exclusions e priorità per gruppo) è cruciale per l'adozione reale del sistema.

**Limiti e assunzioni**:
- Il modello ignora la cannibalizzazione tra prodotti
- La feasible region si basa su soglie empiriche (threshold=0.55 per WAPE)
- La convergenza di Ithax richiede che i target `V*` e `M*` siano ragionevolmente scalati rispetto al catalogo disponibile

**Applicabilità**: framework trasferibile a grocery, department stores e altri contesti retail e-commerce.

---

## Key References

- Loh et al. (2022) — *Promotheus: An End-to-End ML Framework for Optimizing Markdown in Online Fashion E-commerce* (questo paper, KDD 2022)
- Caro & Gallien (2012) — *Clearance pricing optimization for a fast-fashion retailer* — Operations Research (metodo esteso / confronto)
- Hu et al. (2021) — *Markdowns in e-commerce fresh retail: counterfactual prediction and multi-period optimization* — KDD 2021 (approccio correlato)
- Kunnumkal et al. (2020) — *Price optimization in fashion e-commerce* (approccio correlato)
- Bottou et al. (2013) — *Counterfactual reasoning and learning systems* — JMLR (fondamento teorico del partial information problem)
- Hernán & Robins (2020) — *Causal Inference: What If* — fondamento per completeness dei modelli causali
- Shalit, Johansson & Sontag (2017) — *Estimating individual treatment effect: generalization bounds and algorithms* — ICML (ITE nel contesto causal)
- Lane et al. (2004) — *Emerging trends in retail pricing practice* (contesto industry su markdown management)
