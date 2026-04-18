---
title: "Markdown Pricing Under an Unknown Parametric Demand Model"
source: "https://arxiv.org/abs/2312.15286"
author:
  - "[[Su Jia]]"
  - "[[Andrew A. Li]]"
  - "[[R. Ravi]]"
published: 2023-12-23
created: 2026-04-17
description: "Risolve completamente il minimax regret per markdown pricing (prezzi monotonicamente decrescenti) sotto famiglie parametriche di domanda, introducendo il crossing number come misura di complessità e fornendo bound ottimali Θ(log²n) e Θ̃(n^{k/(k+1)})."
tags:
  - "clippings"
  - "dynamic-pricing"
  - "markdown-pricing"
  - "pricing-theory"
  - "regret-bounds"
  - "operations-research"
  - "online-learning"
---

## Markdown Pricing Under an Unknown Parametric Demand Model

### Su Jia (Cornell University), Andrew A. Li, R. Ravi (Carnegie Mellon University) — 2023

---

## Abstract

Un venditore deve massimizzare il ricavo su n round fissando prezzi che **non possono mai aumentare** (markdown policy), con un modello di domanda sconosciuto appartenente a una famiglia parametrica. Senza vincolo di monotonia, il regret minimax è Õ(n^{2/3}) per la famiglia Lipschitz e Õ(n^{1/2}) per famiglie parametriche generali. Con la monotonia, era noto solo Õ(n^{3/4}) per famiglie non-parametriche. Questo lavoro **risolve completamente il problema per famiglie parametriche**: introduce il *crossing number* κ(F) come misura di complessità, e dimostra bound ottimali Θ(log²n) per κ = 0 e Θ̃(n^{k/(k+1)}) per κ = k ≥ 1. Tutti i bound sono asintoticamente superiori a quelli senza vincolo di monotonia, stabilendo una *separazione* netta tra markdown e unconstrained pricing.

---

## 1 Introduzione

### Motivazione pratica

Il vincolo di monotonia nei prezzi è rilevante in molti contesti reali: aumentare i prezzi può danneggiare la reputazione del venditore. Luca e Reshef (2021) trovano che **un aumento dell'1% del prezzo riduce il rating medio del 3–5%**. Di conseguenza, molti venditori adottano politiche di markdown (prezzi solo decrescenti), che penalizzano però il processo di apprendimento della domanda.

### Gap nella letteratura

La letteratura precedente sul markdown pricing si concentrava su famiglie non-parametriche:
- Chen (2021) e Jia et al. (2021): regret ottimale **Õ(n^{3/4})** sotto ipotesi di unimodalità e Lipschitzness del ricavo.
- Migliore sotto differenziabilità C²: **Õ(n^{5/7})** (Jia et al. 2021).

Per famiglie parametriche senza monotonia: **Õ(n^{1/2})** (Broder e Rusmevichientong 2012).

### Domande di ricerca

- **Q1**: Esiste una markdown policy con regret o(n^{5/7}) per famiglie parametriche?
- **Q2**: Ogni markdown policy ha regret ω(√n) su qualche famiglia parametrica? (Separazione dalla pricing non vincolata)

Il paper risponde affermativamente a entrambe le domande.

---

## 2 Formulazione e Assunzioni

### Setup del problema

- Orizzonte temporale: n round discreti.
- Al round t, il venditore seleziona il prezzo X_t ∈ [0,1] e osserva la domanda D_t con E[D_t] = d(X_t).
- Ricavo: R_t = D_t · X_t; ricavo atteso: r(x) = d(x) · x.

> **Markdown policy**: X è una markdown policy se X_{t-1} ≥ X_t quasi certamente per ogni t = 2, …, n.

### Definizione di Regret

> **Reg(X, d)** = n · max_{p ∈ [0,1]} {p · d(p)} − E[Σ_{t=1}^n X_t · d(X_t)]
>
> **Reg(X, F)** = sup_{d ∈ F} Reg(X, d)

### Assunzioni standard

1. **Prezzo ottimale unico**: r ha un unico massimo p*(r).
2. **Prima-ordine ottimalità**: r è differenziabile e r'(p*) = 0.
3. **Rumore subgaussiano**: la domanda D a ogni prezzo è subgaussiana con norma ≤ c_sg.
4. **Dominio compatto**: il parametro θ ∈ Θ compatto.
5. **Smoothness**: d(p; θ) è due volte continuamente differenziabile.
6. **Lipschitz optimal price mapping**: p*(θ) è c*-Lipschitz.

---

## 3 Crossing Number

### Idea chiave

Il crossing number κ(F) misura la complessità di una famiglia di funzioni di domanda tramite due proprietà:

1. **Identifiabilità**: quante valutazioni di prezzo distinte servono per identificare univocamente la funzione di domanda.
2. **Robustezza della parametrizzazione**: come l'errore di stima si propaga dall'errore empirico.

### Definizione formale

**k-identifiability**: la famiglia F è k-identificabile se per qualsiasi k+1 prezzi distinti p = (p_0, …, p_k), la profile mapping Φ_p: F → R^{k+1} (che mappa d → (d(p_0), …, d(p_k))) è iniettiva. Geometricamente: le curve di qualsiasi due funzioni si intersecano al più k volte.

**Robust parametrization**: una parametrizzazione di ordine k è robusta se la mappa inversa Φ_p^{-1} soddisfa:

> ||Φ_p^{-1}(y) − Φ_p^{-1}(y')||_1 ≤ c_2 · ||y − y'||_1 · h(p)^{-k}

dove h(p) = min_{i≠j} |p_i − p_j| è la dispersione dei prezzi di esplorazione.

> **Crossing number κ(F)**: minimo k ≥ 0 tale che F è (i) k-identificabile e (ii) ammette una parametrizzazione robusta di ordine k.

### Esempi importanti

| Famiglia | Crossing number |
|----------|----------------|
| Lineare a singolo parametro: d(x;a) = 1−ax | 0 |
| Esponenziale: d(x;a) = e^{1−ax} | 0 |
| Logit: d(x;a) = e^{1−ax}/(1+e^{1−ax}) | 0 |
| Polinomi di grado k | k |
| Famiglia Lipschitz | ∞ |

**Intuizione per la famiglia lineare a due parametri** (k=1): due curve si intersecano al più una volta; identificare la funzione richiede due prezzi distinti. La robustezza è analoga a quella della matrice di Vandermonde invertita.

### Sensibilità

La sensitivity s cattura quanto è "piatta" la funzione di ricavo attorno a p*. Se le prime s−1 derivate di r in p* sono zero e r^{(s)}(p*) < 0, allora r è s-sensitiva. Questo influenza direttamente i bound di regret.

> Se r è s-sensitiva: |r(p* + ε) − r(p*)| = O(ε^s)

Per s = 2 (caso di base): regret per-round O(ε²). Per s > 2: bound migliori possibili.

---

## 4 Famiglia 0-Crossing

### Pricing non vincolata (baseline)

> **Theorem 1** (Broder e Rusmevichientong 2012): Per qualsiasi famiglia 0-crossing F, la policy MLE-greedy ha Reg = O(log n). Inoltre esiste F 0-crossing tale che qualsiasi policy ha Reg = Ω(log n).

### Policy CM (Cautious Myopic)

La MLE-greedy può violare la monotonia (prezzi crescenti). Si introduce la **Cautious Myopic (CM) policy**, basata sul principio di *conservatism under uncertainty* (opposto al classico *optimism under uncertainty* delle UCB):

**Algoritmo CM**:
1. Partizione dell'orizzonte in m = O(log n) fasi con fase j di durata t_j = ⌈9^j log n⌉.
2. In ogni fase j: seleziona il prezzo corrente P_j per t_j round; calcola la media empirica D̄_j; stima il parametro θ̂_j; costruisce un intervallo di confidenza di ampiezza w_j.
3. Il prezzo per la fase j+1 è il **massimo** tra i prezzi ottimali compatibili con l'intervallo di confidenza (conservatismo), clampato a P_j per garantire la monotonia:

> P_{j+1} ← min{ max{p*(θ) : |θ − θ̂_j| ≤ w_j}, P_j }

### Bound di regret per k = 0

> **Theorem 2** (Upper bound): Reg(CM, F) = O(log²n) per qualsiasi famiglia 0-crossing F.

> **Theorem 3** (Lower bound): Esiste F 0-crossing tale che qualsiasi markdown policy ha Reg = Ω(log²n).

Il gap rispetto a O(log n) senza monotonia è necessario: il conservatismo forza la policy a "sprecare" round per evitare l'overshooting, e questo costo è intrinseco alla monotonia.

**Sketch del lower bound**: A ogni round t = O(√n), si costruisce una coppia di domande (d_0, d_t) con ottimi vicini. Per evitare alto regret su d_t (la domanda con ottimo più alto), la policy deve stare sopra una soglia p̄_t con alta probabilità. Ma questo implica che la policy supera p̄_t anche con probabilità Ω(1) sotto d_0, causando regret Ω(δ_t²) per round. Sommando su t = 1, …, √n:

> Σ_{t=1}^{O(√n)} Ω(log n / t) = Ω(log²n)

### Caso ad alta sensibilità

> **Theorem 4**: Se s > 2, allora Reg(CM, F) = O(log n) — eguaglia il bound senza monotonia.

---

## 5 Famiglia k-Crossing Finita (k ≥ 1)

### Trade-off esplorazione-sfruttamento

Per k ≥ 1 servono k+1 prezzi di esplorazione distinti. Il dilemma:
- Prezzi **dispersi**: stima efficiente del parametro, ma rischio di overshooting (prezzi troppo bassi rispetto a p*).
- Prezzi **concentrati**: stima imprecisa, alto campione necessario.

### Policy ICM (Iterative Cautious Myopic)

**Algoritmo ICM** (parametri: m fasi, distanza h tra prezzi di esplorazione, n_j campioni per fase j):

1. Inizia da P_1 = 1.
2. In ogni fase j: esplora i prezzi P_j, P_j − h, …, P_j − kh, ciascuno per n_j round.
3. Stima θ̂ dalla profile mapping inversa; costruisce [L_j, U_j] come intervallo di confidenza per p*.
4. Confronta l'intervallo con il prezzo corrente P_j − kh:
   - **Good event** (P_j − kh > U_j): ci avviciniamo a p*, P_{j+1} ← U_j.
   - **Dangerous event** (L_j ≤ P_j − kh ≤ U_j): comportamento conservativo, P_{j+1} ← P_j − kh.
   - **Overshooting event** (P_j − kh < L_j): probabilmente già oltre p*, termina esplorazione e sfrutta il prezzo corrente.

### Regret bound generale (ICM)

> **Proposition 5**: Reg(ICM, F) ≤ Õ( h^{-sk} · [Σ_{j=1}^{m-1} n_j / n_{j-1}^{s/2} + n / n_m^{s/2}] + h^s · n )

L'errore di stima del parametro in fase j è:

> ||θ − θ̂||_1 = Õ(k · n_j^{-1/2} · h^{-k})

Propagando al prezzo ottimale (Lipschitz) e alla funzione di ricavo (sensitività s), il regret per fase j è Õ(k·n_j·(k·h^{-k}·n_{j-1}^{-1/2})^s).

### Bound ottimale per s = 2

Ottimizzando h e n_j (riducibile a un programma lineare):

> **Theorem 5** (Upper bound): Per qualsiasi famiglia k-crossing, 2-sensitiva:
>
> Reg(ICM, F) = Õ(n^{k/(k+1)})
>
> con h = n^{-m/(m(k+1)+1)} e n_j = n^{(mk+j)/(m(k+1)+1)}

**Confronto con risultati precedenti**:

| k | Bound nuovo | Bound precedente (non-parametrico) |
|---|-------------|-------------------------------------|
| 1 | Õ(√n) | Õ(n^{5/7}) |
| 2 | Õ(n^{2/3}) | Õ(n^{5/7}) |
| k ≥ 1 | Õ(n^{k/(k+1)}) | Õ(n^{5/7}) (per k ≤ 2) |

### Lower bound per k ≥ 1

> **Theorem 6** (Lower bound): Per qualsiasi k ≥ 2, esiste una famiglia k-crossing tale che ogni markdown policy ha Reg = Ω(n^{k/(k+1)}).

La costruzione usa famiglie di polinomi di grado k decrescenti. Per k ≥ 3, il lower bound Ω(n^{3/4}) è superiore a Õ(n^{5/7}) — non contraddizione, perché le funzioni costruite nella prova non sono unimodali.

---

## Riepilogo dei Risultati

| Crossing number k | Unconstrained pricing | Markdown pricing |
|-------------------|-----------------------|-----------------|
| k = 0 | Θ(log n) | **Θ(log²n)** |
| 1 ≤ k < ∞ | Θ̃(√n) | **Θ̃(n^{k/(k+1)})** |
| k = ∞ (Lipschitz) | Θ̃(n^{2/3}) | Θ̃(n^{3/4}) |

Ogni entry corrisponde a un upper bound e un matching lower bound. I nuovi risultati (markdown pricing) sono asintoticamente superiori a quelli senza vincolo di monotonia: la **separazione** è universale su tutti i crossing number finiti.

---

## Conclusioni / Takeaway

1. **Crossing number come misura unificante**: il framework del crossing number consolida in modo coerente risultati precedenti (nonparametric, well-separated, Lipschitz) e fornisce il linguaggio giusto per caratterizzare il minimax regret in markdown pricing.

2. **Conservatism under uncertainty**: il principio algoritmico chiave che distingue le policy ottimali per markdown pricing da quelle per pricing non vincolato (che usano optimism/UCB).

3. **Separazione necessaria**: il vincolo di monotonia causa sempre una perdita rispetto al pricing non vincolato; la perdita è quantificata esattamente dal crossing number.

4. **Miglioramento pratico**: per famiglie parametriche comuni (lineare, esponenziale, logit con k=0; polinomio lineare con k=1), le nuove policy superano nettamente il bound Õ(n^{5/7}) della letteratura non-parametrica.

5. **Limiti e lavoro futuro**: il paper non affronta il caso di non-stazionarietà della domanda, né vincoli di inventory. Il caso k = ∞ (Lipschitz) rimane con il bound Θ̃(n^{3/4}) già noto.

---

## Key References

- Broder J, Rusmevichientong P (2012) — *Dynamic pricing under a general parametric choice model* — Operations Research 60(4). (metodo base esteso; benchmark unconstrained, Θ(log n) e Θ̃(√n))
- Chen N (2021) — *Multi-armed bandit requiring monotone arm sequences* — NeurIPS 34. (primo bound Θ̃(n^{3/4}) per markdown non-parametrico)
- Jia S, Li A, Ravi R (2021) — *Markdown pricing under unknown demand* — SSRN 3861379. (versione preliminare non-parametrica; bound n^{5/7} per C²)
- Kleinberg R, Leighton T (2003) — *The value of knowing a demand curve* — FOCS 2003. (minimax Õ(n^{2/3}) per Lipschitz bandits senza monotonia)
- Kleinberg RD (2005) — *Nearly tight bounds for the continuum-armed bandit problem* — NeurIPS. (tight Θ̃(n^{2/3}) per one-crossing)
- Lai TL, Robbins H (1985) — *Asymptotically efficient adaptive allocation rules* — Advances Applied Math. (fondamento MAB)
- Luca M, Reshef O (2021) — *The effect of price on firm reputation* — Management Science. (motivazione empirica per markdown constraint)
- Wald A, Wolfowitz J (1948) — *Optimum character of the sequential probability ratio test* — Ann. Math. Stat. (usato nella prova del lower bound k=0)
- Gautschi W (1962) — *On the inverses of Vandermonde matrices* — Numer. Math. (bound norma matrice inversa Vandermonde, chiave per robustezza)
