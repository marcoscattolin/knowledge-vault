---
title: "DISCO: An End-to-End Bandit Framework for Personalised Discount Allocation"
source: "https://arxiv.org/abs/2406.06433"
author:
  - "[[Jason Shuo Zhang]]"
  - "[[Benjamin Howson]]"
  - "[[Panayiota Savva]]"
  - "[[Eleanor Loh]]"
published: 2024-06-12
created: 2026-04-18
description: "Framework end-to-end di contextual bandit per l'allocazione personalizzata di discount code in e-commerce (ASOS): RBF per action encoding continuo, DNN per context embedding, Thompson Sampling con integer program per controllo operativo del budget."
tags:
  - "clippings"
  - "bandits"
  - "contextual-bandits"
  - "dynamic-pricing"
  - "retail"
  - "reinforcement-learning"
  - "thompson-sampling"
  - "e-commerce"
---

## DISCO: An End-to-End Bandit Framework for Personalised Discount Allocation

### Zhang, Howson, Savva, Loh (ASOS.com / Imperial College London) — 2024

---

## Abstract

DISCO è un framework contextual bandit end-to-end per l'allocazione personalizzata di codici sconto su ASOS.com. Adatta il classico Thompson Sampling integrandolo in un integer program per il controllo dei costi operativi. Per mitigare la degradazione delle performance con action space ad alta dimensionalità, costruisce rappresentazioni a bassa dimensione per azioni e contesto tramite radial basis functions (RBF) e una rete neurale profonda (DNN). Il modello preserva la negative price elasticity attesa (clienti che acquistano di più in risposta a sconti maggiori). In un A/B test online, DISCO migliora l'average basket value di **>1%** rispetto al sistema legacy.

---

## 1 Introduzione

### Problema

L'ottimizzazione del prezzo/sconto soffre di **informazione parziale** (partial information): si osservano gli outcome solo per le decisioni di pricing già prese, rendendo difficile la previsione della domanda in risposta a policy diverse. Il framework bandit affronta nativamente questo problema bilanciando esplorazione e sfruttamento.

### Sfide tecniche principali

1. **Action space continuo**: i discount "% off" sono naturalmente continui. La discretizzazione porta a high-dimensional action sets e impedisce l'information sharing tra azioni simili.
2. **Vincoli operativi**: il business deve controllare la distribuzione degli sconti (budget markdown, brand management). I bandit standard non incorporano vincoli globali.
3. **Negative price elasticity**: i modelli di prezzo devono preservare la relazione inversa tra sconto e valore del basket; modelli che la violano sono inaffidabili in produzione.

### Contributi

- (i) Encoding dello spazio delle azioni con **radial basis functions** (RBF)
- (ii) Rappresentazioni di contesto tramite **DNN** (embedding dal penultimate layer)
- (iii) **Thompson Sampling** per la gestione explore-exploit
- (iv) Integrazione del modello bandit in un **integer program** per allocazione con controllo operativo

---

## 2 Formulazione del Problema

Il reward target è il **full price basket value** atteso (valore del carrello prima dell'applicazione dello sconto):

> E[F_{t,i,a} | X_{t,i}, A_{t,i} = a] = g(X_{t,i}, a)

dove:
- `X_{t,i}` = contesto del cliente `i` al round `t`
- `A_{t,i}` = profondità di sconto allocata (es. 0.2 = "20% off")
- `F_{t,i,a}` = full price basket value

Il **markdown cost** è `C_{t,i,a} = F_{t,i,a} * A_{t,i}` — il costo di applicare lo sconto.

La scelta di modellare il full price (non il discounted) basket value è motivata dall'osservazione che i clienti rispondono a sconti più profondi aumentando il valore full price degli acquisti, non necessariamente il valore scontato.

---

## 3 Architettura DISCO

### 3.1 Action Feature Representation — Radial Basis Functions

Le RBF trasformano la profondità di sconto continua in una rappresentazione a bassa dimensione:

> ψ_{2,z}(a | μ_z, α_z) = exp(-(a - μ_z)² / (2α_z))

con `ψ_2 = {ψ_{2,z}(a | μ_z, α_z)}` in R^{d2}.

**Vantaggi delle RBF rispetto ad alternative:**
- Encoding continuo scalare → assume linearità (troppo restrittivo)
- One-hot / discretizzazione → no information sharing, aumenta dimensionalità
- RBF → **pooled learning** tra azioni simili; bassa dimensionalità; preserva monotonia (negative price elasticity)

Configurazione scelta: **K=3 centroidi, α=20** — il più basso K che preserva monotonia con sufficiente capacità espressiva.

### 3.2 Context Feature Representation — DNN

Una rete neurale profonda viene usata per ridurre la dimensionalità del contesto cliente:

- Input: **N=76 feature** (storia acquisti, resi, utilizzo codici sconto, interazioni sul sito)
- Architettura: 4 layer [64, 16, 6, 1]
- Output del penultimate layer: **embedding 6-dimensionale** per cliente
- Training: 5M clienti attivi, dati dell'anno precedente, Adam optimizer (lr=0.001), dropout

Il DNN è addestrato sulla previsione del log full price basket value e funge da funzione di representation learning `ψ_1 : X → R^{d1}`.

### 3.3 Reward Prediction — Bayesian Log-linear Regression

Le feature finali per il cliente `i` con sconto `a` sono il prodotto cartesiano di ψ_1 e ψ_2:

> ψ(X_{t,i}, a) = {ψ_1(X_{t,i}), ψ_2(a), ψ_1(X_{t,i}) × ψ_2(a)} ∈ R^d

con `d = d1 + d2 + d1*d2`.

Il modello di reward è log-lineare bayesiano:

> E[ln(F_{t,i,a}) | X_{t,i} = x, A_{t,i} = a] = ⟨θ, ψ(x, a)⟩

**Thompson Sampling** mantiene una distribuzione posteriore su θ:

> θ̂_t ~ N(μ = V̄_t^{-1} B_t, σ² = β_t² V̄_t^{-1})

Il posterior si aggiorna efficientemente con la Woodbury matrix identity in O(d²) operazioni. Per ogni cliente, si campiona un pseudo-reward `F̃_{t,i,a}` per ogni azione, guidando l'esplorazione tramite incertezza.

### 3.4 Ottimizzazione dell'Allocazione — Integer Program

I reward campionati vengono usati come input di un integer program che alloca gli sconti rispettando i vincoli operativi:

> max Σ_{i,a} (w · R̃_{i,a} - C̃_{i,a}) · s_{i,a} · e_a

Soggetto a:
- `s_{i,a} ∈ {0, 1}` — ogni cliente riceve al massimo uno sconto
- `Σ_a s_{i,a} ≤ 1` ∀i
- `Σ_i s_{i,a} ≤ N_a` ∀a — distribuzione target degli sconti fissata dagli operatori

dove `w` è un peso per bilanciare revenue maximization vs cost minimization, `N_a` è il numero di utenti da allocare alla profondità `a`, e `e_a` è il tasso di engagement storico per lo sconto `a`.

---

## 4 Esperimenti

> ⚠️ Per motivi di sensibilità commerciale, tutte le profondità di sconto, valori revenue e % di miglioramento sono riscalate in unità arbitrarie.

### 4.1 Information Sharing e Negative Price Elasticity

**Configurazione RBF**: diversi schemi di encoding sono stati valutati su WAPE, monotonicity (negative price elasticity) e dimensionalità. Tutti i modelli mostravano accuracy simile (WAPE≈0.140, ρ≈0.475), ma solo le configurazioni RBF con K=3 e α=20 preservavano la monotonia desiderata.

**Uncertainty e extrapolation**: il modello Bayesiano mostrava appropriata maggiore incertezza per regioni dello spazio azioni non osservate nel training (a < 0.6), pur mantenendo buona accuracy per extrapolation grazie alle RBF.

### 4.2 Reward Prediction Model

| Modello | WAPE | Spearman ρ |
|---|---|---|
| DNN | 0.153 | 0.409 |
| Random Forest | 0.153 | 0.409 |
| LightGBM | 0.154 | 0.410 |
| Linear Regression | 0.160 | 0.346 |

Il DNN è comparabile a RF/LGBM in accuracy ma superiore per generare **context embedding** per il downstream bandit. Il reward model finale (Bayesian log-linear) su due campagne di training raggiunge WAPE=0.139, ρ=0.438 su campagna test unseen — e WAPE=0.134, ρ=0.461 anche su una campagna di tipo diverso (canale email, finestra di riscatto più lunga, profondità di sconto shallower = extrapolation).

### 4.3 Active Learning con Vincoli Globali

Confronto tra algoritmi bandit con integer program (warm-start e cold-start), valutato su ABV (Average Basket Value) tramite rejection sampling su campagna randomizzata:

| Algoritmo | Note |
|---|---|
| **TS-IP** (Thompson Sampling + IP) | Migliore performance a lungo termine in entrambi gli scenari |
| UCB-IP | Performance declina nel tempo — esplorabilità penalizzata dall'IP |
| Greedy-IP | Inizialmente buono, degrada per dataset biasati da policy greedy |
| ε-Greedy-IP | Simile a Greedy-IP |
| Random | Baseline |

**Benchmark TS-ULCC** ("Unconstrained Learner, Constrained Consumer"): un learner che esplora senza vincoli aggiorna il modello, mentre un consumer vincolato raccoglie reward. TS-IP è solo **0.234% peggiore** di TS-ULCC su 100 batch (p<0.001), e il gap non cresce nel tempo (p>0.05). Il vincolo IP ha quindi un impatto minimo sulla capacità di active learning.

---

## 5 Online A/B Test

**Setup**: clienti idonei randomizzati in Test (DISCO) vs Control (allocazione casuale con stessa configurazione `N_a`).

| Metrica | DISCO vs Control randomizzato | DISCO vs Undifferentiated (stesso sconto a tutti) |
|---|---|---|
| Revenue | +1.12% (p<0.001) | +3.56% (p<0.001) |
| Reward (revenue - cost) | +1.23% (p<0.01) | +4.10% (p<0.001) |

Accuracy online: WAPE=0.133, ρ=0.446 — coerente con offline evaluation, confermando la validità del processo di valutazione.

---

## Conclusioni / Takeaway

- **RBF come action encoding** è la chiave per gestire action space continui nei bandit: permette information sharing, bassa dimensionalità e preservazione della negative price elasticity.
- **Thompson Sampling + Integer Program** permette di combinare active learning con controllo operativo reale, con un costo minimo (~0.23%) sulla capacità di esplorazione.
- **DNN per context embedding** è necessario per ridurre la dimensionalità del contesto cliente senza perdere capacità predittiva.
- Il framework è data-efficient: bastano **2 campagne di training** per raggiungere buona accuracy.
- Generalizzabile a: product recommendations, CRM targeting, qualsiasi problema di personalizzazione con action space continuo e vincoli operativi.

---

## Key References

- Agrawal & Goyal (ICML 2013) — *Thompson sampling for contextual bandits with linear payoffs* (base algoritmica di TS)
- Lattimore & Szepesvári (2020) — *Bandit Algorithms* (textbook di riferimento)
- Li et al. (WWW 2010) — *A contextual-bandit approach to personalized news article recommendation* (LinUCB; framework offline evaluation)
- Loh et al. (KDD 2022) — *Promotheus: An end-to-end ML framework for optimizing markdown in online fashion e-commerce* (lavoro precedente ASOS su markdown optimization)
- Goldenberg & Albert (RecSys 2022) — *Personalizing benefits allocation without spending money: Uplift modeling in a budget constrained setup* (alternativa uplift-based)
