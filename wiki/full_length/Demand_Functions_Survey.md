---
title: "Demand Functions in Decision Modeling: A Comprehensive Survey and Research Directions"
source: ""
author:
  - "[[Jian Huang]]"
  - "[[Mingming Leng]]"
  - "[[Mahmut Parlar]]"
published: 2012-07-01
created: 2026-04-17
description: "Survey completo delle forme funzionali delle funzioni di domanda in sei categorie (prezzo, rebate, leadtime, spazio, qualità, pubblicità) con analisi single-firm e game-theoretic multi-firm, vantaggi/svantaggi di ciascun modello e direzioni di ricerca."
tags:
  - "clippings"
  - "pricing-theory"
  - "demand-modeling"
  - "price-elasticity"
  - "operations-research"
  - "economics"
  - "survey"
---

## Demand Functions in Decision Modeling: A Comprehensive Survey and Research Directions

### Jian Huang (Jiangxi University of Finance and Economics), Mingming Leng (Lingnan University), Mahmut Parlar (McMaster University) — 2012

---

## Abstract

Il paper fornisce un survey delle forme matematiche più usate per caratterizzare le funzioni di domanda che dipendono dalle attività operative e di marketing di un'impresa. Le sei categorie analizzate sono: (i) prezzo, (ii) rebate, (iii) leadtime, (iv) spazio a scaffale, (v) qualità, (vi) pubblicità. Per ciascuna categoria vengono esaminati modelli single-firm e modelli game-theoretic multi-firm con interazione strategica. Le forme funzionali più pervasive — lineare, power/iso-elastica, MNL (MultiNomial Logit) e MCI (Multiplicative Competitive Interaction) — compaiono in tutte e sei le categorie. Le categorie (i) e (v) dominano la letteratura per numero di pubblicazioni.

---

## 1 Introduzione

I consumatori percorrono cinque fasi nel ciclo d'acquisto (Lilien et al. 1992): stimolo, ricerca, valutazione, decisione, post-acquisto. In ciascuna fase le decisioni operative e di marketing dell'impresa — prezzo, rebate, leadtime, spazio, qualità, pubblicità — influenzano la domanda.

Il paper seleziona i modelli secondo quattro criteri: (1) domanda sensibile alle decisioni dell'impresa; (2) modelli non già recensiti altrove (es. Urban 2005 per inventory-dependent, Erickson 2003 per advertising dinamico); (3) privilegio per i modelli fondativi rispetto alle estensioni marginali; (4) semplicità e rappresentatività.

**Struttura del paper:** Sezioni 2–7 = survey delle sei categorie; Sezione 8 = studi empirici; Sezione 9 = sintesi e direzioni future.

---

## 2 Modelli di Domanda Dipendenti dal Prezzo

### 2.1 Single-Firm: Modelli Deterministici

I modelli deterministici sono classificati in sei famiglie. La Tabella 1 originale li elenca sistematicamente.

**Modello Lineare (LM)**

> `d(p) = a − bp`, con a, b > 0 e p ≤ a/b

- Ampiamente usato per la facilità di stima dei parametri e per i risultati analitici espliciti.
- L'elasticità `−bp/(a−bp)` è decrescente e concava in p.
- Limite: richiede un prezzo massimo finito; non cattura nonlinearità reali.

**Modello Power / Iso-elastico (PM)**

> `d(p) = a · p^(−b)`, con a > 0, b > 1

- Elasticità costante pari a `−b` per costruzione (constant elasticity model).
- Si linearizza con il logaritmo → stima OLS facilitata.
- Collegamento teorico con la funzione di produzione Cobb-Douglas.
- Limite: quando p → 0 la domanda → ∞ (market size non bounded).

**Modello Esponenziale (EM)**

> `d(p) = a · exp(−βp)`

- Risposta alla riduzione di prezzo con rendimenti crescenti.
- Usato per pricing dinamico con inventario.

**Modello Logit (LtM)**

> `d(p) = a · exp(−bp) / [1 + exp(−bp)]`

- Parametri stimabili con regressione.
- Naturalmente bounded in (0, a).

**Modello Ibrido (HM)**

> `d(p) = θ(a₁ − bp)^α + (1−θ)a₂p^(−β)`, θ ∈ [0,1]

- Interpola tra lineare e power; utile per confrontare effetti in sistemi multi-echelon.

**Osservazione chiave (Lau e Lau 2003):** per sistemi single-echelon la scelta della forma funzionale porta a conclusioni strutturalmente simili; nei sistemi multi-echelon anche piccole variazioni nella forma portano a differenze significative nell'efficienza e nei rapporti di profitto del canale.

### 2.1.2 Modelli Stocastici

Il termine stocastico ε è price-independent con c.d.f. F(·) su [A, B].

| Tipo | Forma |
|---|---|
| Additivo (Mills 1959) | `D(p,ε) = d(p) + ε` |
| Moltiplicativo (Karlin & Carr 1962) | `D(p,ε) = d(p)·ε` |
| Ibrido I (Young 1978) | `D(p,ε) = d₁(p)·ε + d₂(p)` |
| Ibrido II (Chen & Simchi-Levi 2004) | `D(p,ε) = ε₁·d(p) + ε₂` |

- Nel modello **additivo**: varianza della domanda indipendente da p; coefficiente di variazione crescente in p.
- Nel modello **moltiplicativo**: coefficiente di variazione indipendente da p; varianza decrescente in p.
- Kocabıyıkoğlu e Popescu (2011) unificano questi modelli con la misura *elasticity of lost-sales rate* (LSR).

### 2.1.3 Modelli Willingness-To-Pay (WTP)

> `d(p) = Λ · ∫[p,∞] f(v) dv`

dove Λ = market size, V = WTP del consumatore con p.d.f. f(v).

La distribuzione di V determina la forma aggregata della domanda:

| Distribuzione di V | Domanda aggregata |
|---|---|
| Uniforme [A, B] | Lineare: `(B−p)/(B−A)` |
| Esponenziale(λ) | Log-lineare: `Λ·exp(−λp)` |
| Logistica(μ,σ) | Funzione logistica |
| Weibull(α,β) | Esponenziale |
| Pareto(b) | Power: `p^(−b)` |

Utile quando si vuole analizzare le scelte a livello micro (consumer choice); altrimenti si usa direttamente un modello aggregato.

### 2.1.4 Modelli Poisson Flow

Per il pricing dinamico su orizzonti finiti (revenue management), i clienti arrivano secondo un processo di Poisson con intensità λ(t) e hanno reservation price V(t).

> Domanda attesa al prezzo p(t): `λ(t)·[1 − F(p(t))` per unità di tempo

Estensioni: Zhao & Zheng (2000) — Poisson non-omogeneo con F(·,t) variabile nel tempo; Xu & Hopp (2009) — utilità `U(p,t) = V(t) − θ(t)p` con acquisto se U ≥ 0.

### 2.2 Multi-Firm: Modelli con Competizione di Prezzo

Con n ≥ 2 imprese che producono beni differenziati, la domanda dell'impresa i soddisfa:
- `∂dᵢ/∂pᵢ < 0` (domanda decrescente nel proprio prezzo)
- `∂dᵢ/∂pⱼ > 0` (domanda crescente nel prezzo del concorrente)
- Elasticità propria IPE (increasing price elasticity) in pᵢ

**Modello Lineare (Anderson et al. 1992; Vives 1999)**

> `dᵢ(p) = aᵢ − bᵢpᵢ + γᵢ·Σⱼ≠ᵢ pⱼ`

Derivato dalla massimizzazione dell'utilità del consumatore rappresentativo con utility quadratica.

**Modello Attraction (MNL e MCI)**

> `dᵢ(p) = M·vᵢ(pᵢ) / [v₀ + Σⱼ vⱼ(pⱼ)]`

- **MNL**: `vᵢ(pᵢ) = exp(αᵢ − βᵢpᵢ)` → elasticità di mercato non monotona in p
- **MCI**: `vᵢ(pᵢ) = δᵢ · pᵢ^(−γᵢ)` → elasticità di mercato monotonicamente decrescente in p
- Cooper (1993): MCI più appropriato per competizione di prezzo; MNL più appropriato per competizione pubblicitaria.

**Modello Cobb-Douglas (log-lineare competitivo)**

> `dᵢ(p) = aᵢ · pᵢ^(−βᵢ) · Πⱼ≠ᵢ pⱼ^(γᵢⱼ)`

- Elasticità propria e incrociata costanti (price-independent).
- Si linearizza con logaritmo → stima OLS diretta.
- Milgrom & Roberts (1990): le quattro forme (lineare, MNL, Cobb-Douglas, constant expenditure) soddisfano la log-supermodularità, che garantisce l'esistenza e l'unicità dell'equilibrio di Nash.

---

## 3 Modelli di Domanda Dipendenti dal Rebate

Il rebate R (sconto diretto al consumatore finale) entra nella funzione di domanda in forma additiva o moltiplicativa.

**Modelli principali:**

> Additivo stocastico (Arcelus et al. 2005): `D(p, R) = a₀ + αR − bp + ε`

> Moltiplicativo stocastico: `D(p, R) = δ₀ · R^α · p^(−β) · ε`, con α < β

> Power (Arcelus & Srinivasan 2003): `d(p, R) = δ₀ · (p − R)^(−γ)`

> Generale (Aydin & Porteus 2008): `D(p, R) = g(p − aR) · ε`, con g(·) non-lineare decrescente

> Additivo multi-firm (Geng & Mallik 2011): `D(p, Rᵣ, Rₘ) = a − b(p − Rᵣ − Rₘ) + ε`

**Pro e contro dei modelli rebate:**
- I modelli lineari portano a risultati analitici espliciti ma non catturano nonlinearità.
- Il modello power di Khouja (2006): elasticità al rebate costante; quando R→0, d→0 (non realistico).
- Il modello power di Arcelus & Srinivasan (2003): elasticità `R/(p−R)` variabile con p e R; più realistico.

**Direzioni di ricerca:** la competizione tra rebate di più imprese (oligopoly) è quasi inesplorata.

---

## 4 Modelli di Domanda Dipendenti dal Leadtime

Il leadtime di consegna L garantito dall'impresa riduce la domanda se elevato. Baker et al. (2001): meno del 10% dei consumatori finali basa la decisione d'acquisto sul solo prezzo.

### 4.1 Single-Firm

**Modello Lineare (Palaka et al. 1998; Pekgün et al. 2008)**

> `d(p, L) = a − αp − βL`

dove a = domanda massima a p=0 e L=0; α, β > 0.
Il sistema di produzione è modellato come una coda M/M/1 con rate μ; il vincolo di service level impone `d(c, k/μ) > 0`.

**Modello Cobb-Douglas (So & Song 1998)**

> `d(p, L) = Θ · p^(−a) · L^(−b)`, a, b > 0

Problema di ottimizzazione con vincolo di service level: `1 − exp{−[μ − d(p,L)]·L} ≥ γ`.

**Modello MNL (Ho & Zheng 2004)**

> `d = λ · exp(U) / [exp(U) + A]`

dove utilità `U = θ₀ − θ_L · L + θ_ω · ω` e `ω = Pr(delivery ≤ L)` è la qualità del servizio.

**Modello WTP (Li 1992)**

Utilità del consumatore: `U(q, p, L) = u(q, p) − δL`. Il cliente acquista se l'utilità attesa supera il reservation price.

### 4.2 Multi-Firm con Competizione su Leadtime

**Lineare competitivo (Boyaci & Ray 2003, 2006; Pekgün et al. 2009):**

> `dᵢ(pᵢ, Lᵢ, pⱼ, Lⱼ) = aᵢ − bᵢpᵢ − γᵢLᵢ + θᵢⱼpⱼ + φᵢⱼLⱼ`

**MCI con prezzo e leadtime (So 2000):**

> `dᵢ(p, L) = Θ · [δᵢ · pᵢ^(−a) · Lᵢ^(−b)] / [Σⱼ δⱼ · pⱼ^(−a) · Lⱼ^(−b)]`

**Osservazione:** i modelli leadtime-dipendenti mancano ancora di una componente stocastica ε analoga a quella dei modelli price-dipendenti → gap di ricerca aperto.

---

## 5 Modelli di Domanda Dipendenti dallo Spazio a Scaffale

L'allocazione di spazio scaffale (shelf space) sᵢ a ciascun prodotto i influenza la domanda (Urban 2005). L'effetto ha sia componente diretta (elasticità propria) che incrociata tra prodotti.

### 5.1 Modello Statico: Cobb-Douglas (Corstjens & Doyle 1981)

> `dᵢ(s) = δᵢ · sᵢ^(αᵢ) · Πⱼ≠ᵢ sⱼ^(αᵢⱼ)`

- αᵢ = elasticità diretta dello spazio per il prodotto i (αᵢ ≥ 0)
- αᵢⱼ = elasticità incrociata (αᵢⱼ > 0 se complementari, αᵢⱼ < 0 se sostituti)
- Ottimizzazione con geometric programming; la non-concavità dell'obiettivo può impedire l'ottimalità globale.

Estensione con prezzo (Martín-Herrán et al. 2006):

> `dᵢ(sᵢ, p₁, p₂) = Θ · (sᵢ)^α · (pᵢ)^(−βᵢ) · (pⱼ)^(εᵢ)`

### 5.2 Modello Dinamico (Corstjens & Doyle 1983)

> `dᵢₜ = δᵢ · (sᵢₜ)^(αᵢₜ) · Πⱼ≠ᵢ (sⱼₜ)^(αᵢⱼₜ) · (dᵢ,ₜ₋₁)^ρ`

dove ρ = elasticità goodwill inter-periodo (0 < ρ < 1).

**Evidenza empirica (Desmet & Renaudin 1998):** l'elasticità media dello spazio è 0.21, ma varia da −0.44 a +0.80 per categoria, inclusi valori negativi (moda). Questo contraddice l'assunzione di elasticità sempre positiva nei modelli teorici.

---

## 6 Modelli di Domanda Dipendenti dalla Qualità

### 6.1 Qualità del Prodotto: Utility-Based (Single-Firm)

**Modello di Moorthy (1988) e Tirole (1988):**

> `d(p, y) = a · [1 − F(p/y)]`

dove θ = gusto del consumatore con p.d.f. f(θ), y = qualità. Il consumatore acquista se θy − p ≥ 0.

Con distribuzione uniforme su [A, B]: coincide con il modello lineare aggregato LM I.

**Modello Tirole con due qualità y₁ < y₂:**
- Se θ₁ ≥ θ₂: `d₂ = a(1 − F(θ₂))`
- Se θ₁ < θ₂: `d₁ = a[F(θ̂) − F(θ₁)]`, `d₂ = a[1 − F(θ̂)]`, con `θ̂ = (p₂−p₁)/(y₂−y₁)`

**Modello power utility (Zhao et al. 2009):**

> `u(p) = θ · y^(1/n) − p`

Proprietà: elasticità marginale costante `1/n − 1`; coefficiente di avversione relativa al rischio `1 − 1/n`.

### 6.2 Modelli Lineari Competitivi (Banker et al. 1998)

Per n = 2 imprese che competono su prezzo p e qualità y:

> `dᵢ(p, y) = kᵢ − α·pᵢ + β·pⱼ + γ·yᵢ − δ·yⱼ`

con α > β (prezzo proprio più impattante del concorrente) e γ > δ (qualità propria più impattante).

### 6.3 Qualità del Servizio (Service Quality)

**Modello Generale (Desai & Srinivasan 1995):**

> `d^J(p, y) = T_J − αp + g(y) + ε`, g(y) concava in y

**Modello Square-Root (Desiraju & Moorthy 1997):**

> `d(p, y) = Δ − αp + β√y`

- `∂d/∂y > 0` e `∂²d/∂y² < 0`: domanda crescente in y con rendimenti decrescenti.

**Modello Log-Separabile per n imprese (Bernstein & Federgruen 2004):**

> `dᵢ(p, y) = φᵢ(y) · qᵢ(p)`

dove φᵢ(y) scala la domanda in funzione dei livelli di servizio e qᵢ(p) è uno dei modelli classici (lineare, logit, Cobb-Douglas, CES).

**Modello Attraction con qualità del servizio:**

> `dᵢ(p, y) = M · vᵢ(pᵢ, yᵢ) / [v₀ + Σⱼ vⱼ(pⱼ, yⱼ)]`

con `vᵢ(pᵢ, yᵢ) = exp[bᵢ(yᵢ) − βᵢpᵢ]` (MNL).

---

## 7 Modelli di Domanda Dipendenti dalla Pubblicità

La pubblicità A influenza la domanda come: (i) canale informativo che riduce la differenziazione, o (ii) persuasione che crea differenziazione.

### 7.1 Single-Firm

**Modello Generale (Dorfman & Steiner 1954):**

> `d(p, A)` con `∂d/∂p < 0 < ∂d/∂A`

Profitto: `π(p, A) = p·d(p,A) − C(d) − c_A·A`

**Derivato da utilità (Pepall et al. 1999):**

> `d(p, A) = a[1 − p/g(A)]`, con g(0) = 1, g'(A) > 0

**Modello Log-Separabile / Moltiplicativo (supply chain):**

> `d(A₁, A₂, p) = a · d₁(A₁, A₂) · d₂(p)`

dove l'effetto pubblicitario `d₁` è specificato come:
- Power: `d₁ = Θ·(A₁)^α·(A₂)^β` — bounded superiormente ma può diventare negativo quando A→0
- Square-root: `d₁ = k₁√A₁ + k₂√A₂` — non bounded superiormente; = 0 quando A₁ = A₂ = 0 (non realistico)

### 7.2 Multi-Firm

**MNL (Gruca & Sudharshan 1991):**

> `dᵢ = Vᵢ / Σⱼ Vⱼ`, con `Vᵢ = aᵢ·exp(−βᵢpᵢ + γᵢAᵢ)`

**MCI (Mills 1961):**

> `dᵢ = a·Vᵢ / Σⱼ Vⱼ`, con `Vᵢ = δᵢ·Aᵢ^α`

Elasticità di mercato del MNL è non-monotona in A (prima cresce poi decresce); quella del MCI è monotonicamente decrescente. Cooper (1993): per competizione pubblicitaria il MNL è più appropriato (opposto a quanto vale per la competizione di prezzo).

---

## 8 Studi Empirici

### Modelli Prezzo-Dipendenti
- **Genesove & Mullin (1998)** — industria dello zucchero: testano lineare, log-lineare, esponenziale, quadratico; l'**esponenziale** produce le SEE più basse.
- **Sudhir (2001)** — yogurt e burro d'arachidi: il modello **logit** (log-likelihood −196 vs −343 per il moltiplicativo) domina nettamente.
- **Besanko et al. (1998)** — MNL multi-brand: le elasticità empiriche sono > 1 in valore assoluto, coerenti con la teoria.
- **Nakanishi & Cooper (1982)**: la stima empirica con MCI è migliore di quella con MNL (coerente con Remark 7).

### Modelli Coupon/Rebate-Dipendenti
- **Leone & Srinivasan (1996)** e **Kumar & Swaminathan (2005)**: il modello esponenziale generalizzato con termine di decadimento del coupon ha R² aggiustato 0.69–0.81 vs 0.54–0.62 per lineare, semi-log, double-log.

### Modelli Spazio-Dipendenti
- **Desmet & Renaudin (1998)**: elasticità media 0.21 con range −0.44/+0.80; le categorie fashion hanno elasticità negative → violano le assunzioni dei modelli Cobb-Douglas.

---

## 9 Sintesi e Tabella Comparativa

### Distribuzione delle forme funzionali per categoria (Tabella 4 originale)

| Forma Funzionale | Prezzo | Rebate | Leadtime | Spazio | Qualità | Pubblicità |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| Generale | ✓ | ✓ | — | — | ✓ | ✓ |
| Cobb-Douglas | ✓ | — | ✓ | ✓ | — | — |
| Esponenziale | ✓ | — | — | — | — | — |
| Lineare | ✓ | ✓ | ✓ | — | ✓ | — |
| Log-Separabile | — | — | — | — | ✓ | ✓ |
| MCI | ✓ | — | ✓ | — | ✓ | ✓ |
| MNL | ✓ | — | ✓ | — | ✓ | ✓ |
| Power/Iso-elastico | ✓ | ✓ | — | — | ✓ | ✓ |
| Square-Root | — | — | — | — | ✓ | ✓ |
| Utility-Based | ✓ | ✓ | ✓ | — | ✓ | — |

### Confronto sintetico dei modelli

| Modello | Vantaggi | Svantaggi |
|---|---|---|
| Lineare | Risultati analitici espliciti; facile stima parametri | Non cattura nonlinearità; richiede bound finito su p |
| Power/Iso-elastico | Elasticità costante; linearizzabile con log; legame con Cobb-Douglas | d → ∞ per p → 0; elasticità fissa non realistica |
| Esponenziale | Rendimenti crescenti alla riduzione di prezzo; no bound su p | Meno comune, stima più complessa |
| Logit (MNL) | Bounded; stimabile come log-lineare; cattura scelte discrete | Proprietà IIA (independence of irrelevant alternatives) |
| Attraction MCI | Elasticità di mercato monotona decrescente; adatta a price competition | Meno adatta ad advertising competition |
| Cobb-Douglas (multi-firm) | Cattura nonlinearità; elasticità incrociata costante; facilmente stimabile | Elasticità fissa; può essere non-concavo (ottimizzazione difficile) |
| Log-Separabile | Separa effetto prezzo da effetto qualità/servizio | Meno realistico se le interazioni p-y sono rilevanti |
| WTP / Utility-Based | Fondamento microeconomico; consente analisi a livello micro | Più complesso; stima parametri richiede dati individuali |

---

## Conclusioni / Takeaway

1. **Il modello più usato è quello price-dependent**: il pricing è lo strumento competitivo più studiato; quasi tutti i modelli di altre categorie includono anche il prezzo come variabile.
2. **Le forme funzionali universali** (lineare, MNL, MCI, power) compaiono in tutte le sei categorie; la scelta dipende dal tradeoff tractability vs realismo.
3. **Gap aperti:**
   - Rebate: assenza di modelli con competizione tra più di due imprese.
   - Leadtime: nessun modello stocastico con ε indipendente; mancano modelli utility-based per la competizione.
   - Spazio: i modelli si concentrano su sistemi retail tangibili; manca l'estensione ai sistemi di servizio.
   - Qualità: pochi modelli per oligopoly (n > 3); le utility non-lineari (MNL, MCI) sono quasi inesplorate.
   - Pubblicità: mancano modelli che integrino la competizione pubblicitaria in supply chain multi-livello.
4. **Studi empirici:** dominati dai modelli price-dependent; quasi assenti per leadtime e qualità. Il modello migliore varia per industria (esponenziale per zucchero, logit per yogurt/burro d'arachidi).
5. **Implicazione pratica:** per sistemi multi-echelon la scelta della forma funzionale è critica; anche piccole differenze cambiano l'efficienza del canale e i rapporti di profittabilità.

---

## Key References

- Mills (1959) — *Uncertainty and price theory* (modello additivo stocastico, base stocastica)
- Karlin & Carr (1962) — modello moltiplicativo stocastico e iso-elastico
- Petruzzi & Dada (1999) — *Pricing and newsvendor problem* (confronto additivo vs moltiplicativo)
- Kocabıyıkoğlu & Popescu (2011) — *Elasticity approach to newsvendor with pricing* (unified LSR framework)
- Anderson, Palma & Thisse (1992) — *Discrete Choice Theory of Product Differentiation* (fondamento utility dei modelli competitivi)
- Milgrom & Roberts (1990) — log-supermodularità e Nash equilibrium per modelli competitivi
- Bernstein & Federgruen (2004) — *General equilibrium model for industries with price and service competition* (attraction, linear, log-separable multi-firm)
- Corstjens & Doyle (1981) — *A model for optimizing retail space allocations* (Cobb-Douglas spazio statico)
- Banker, Khosla & Sinha (1998) — *Quality and competition* (modello lineare qualità competitivo)
- Tirole (1988) — *The Theory of Industrial Organization* (utility-based quality demand, Hotelling)
- Cooper (1993) — *Market share models* (MNL vs MCI, share elasticity)
- So & Song (1998) — Cobb-Douglas leadtime monopoly
- Ho & Zheng (2004) — MNL leadtime competition
- Genesove & Mullin (1998) — *Testing static oligopoly models* (empirical sugar industry)
- Sudhir (2001) — *Competitive pricing behavior in the US auto market* + yogurt/peanut butter (logit vs multiplicative empirical)
