---
title: "Claude Certified Architect – Foundations Certification Exam Guide"
source: ""
author:
  - "[[Anthropic]]"
published: 2025-01-01
created: 2026-04-17
description: "Guida ufficiale all'esame di certificazione Claude Certified Architect – Foundations: 5 domini, scenari pratici, domande campione e esercizi di preparazione per chi costruisce applicazioni production-grade con Claude."
tags:
  - "clippings"
  - "claude-code"
  - "agent-sdk"
  - "mcp"
  - "certification"
  - "prompt-engineering"
  - "agentic-systems"
---

## Claude Certified Architect – Foundations Certification Exam Guide

### Anthropic, PBC — 2025

---

## Introduzione

La certificazione **Claude Certified Architect – Foundations** valida la capacità di prendere decisioni informate sui trade-off nell'implementazione di soluzioni real-world con Claude. L'esame copre quattro tecnologie core:

- **Claude Code** — CLI per sviluppo assistito da AI
- **Claude Agent SDK** — framework per agenti autonomi e multi-agente
- **Claude API** — integrazione diretta del modello
- **Model Context Protocol (MCP)** — standard per connettere Claude a sistemi backend

Le domande sono basate su scenari realistici: customer support, code review CI/CD, sistemi di ricerca multi-agente, data extraction.

### Target Candidate

Il candidato ideale è un solution architect con 6+ mesi di esperienza pratica con Claude che sa:

- Costruire applicazioni agentiche con l'Agent SDK (orchestrazione multi-agente, hooks, tool integration)
- Configurare Claude Code con `CLAUDE.md`, skills, MCP e plan mode
- Progettare tool e resource interface MCP per sistemi backend
- Ingegnerizzare prompt per output strutturato (JSON schema, few-shot)
- Gestire context window in conversazioni lunghe, multi-turn e multi-agent handoff
- Integrare Claude in pipeline CI/CD per code review e test generation

---

## Formato Esame

| Caratteristica | Dettaglio |
|---|---|
| Tipologia domande | Multiple choice, 1 risposta corretta + 3 distractors |
| Domande senza risposta | Conteggiate come errate (no penalità per guess) |
| Esito | Pass / Fail |
| Scoring | Scala 100–1000; **minimo passaggio: 720** |
| Scenari | 4 su 6 possibili, estratti casualmente |

---

## Scenari d'Esame

| # | Scenario | Domini Primari |
|---|---|---|
| 1 | Customer Support Resolution Agent | D1, D2, D5 |
| 2 | Code Generation con Claude Code | D3, D5 |
| 3 | Multi-Agent Research System | D1, D2, D5 |
| 4 | Developer Productivity con Claude | D2, D3, D1 |
| 5 | Claude Code per CI/CD | D3, D4 |
| 6 | Structured Data Extraction | D4, D5 |

---

## Dominio 1: Agentic Architecture & Orchestration (27%)

### Task 1.1 — Agentic Loop

**Concetti chiave:**
- Loop lifecycle: inviare request → ispezionare `stop_reason` (`"tool_use"` vs `"end_turn"`) → eseguire tool → reinserire risultati nella storia
- Tool results si accumulano nella conversation history per il ragionamento iterativo
- Differenza tra model-driven decision-making e pre-configured decision trees

**Skills:**
- Implementare control flow che continua su `"tool_use"` e termina su `"end_turn"`
- Evitare anti-pattern: parsing di segnali in linguaggio naturale per decidere terminazione, iteration caps arbitrari come meccanismo primario di stop

### Task 1.2 — Orchestrazione Multi-Agent

**Concetti chiave:**
- Hub-and-spoke: il coordinator gestisce tutta la comunicazione inter-subagent, error handling e routing
- I subagent operano con **contesto isolato** — non ereditano automaticamente la conversation history del coordinator
- Rischi di task decomposition troppo narrow → coverage incompleta

**Skills:**
- Coordinator dinamici che scelgono quali subagent invocare in base alla query
- Partizione dello scope di ricerca tra subagent per minimizzare duplicazioni
- Loop di refinement iterativo: coordinator valuta output, re-delega su gap, re-invoca synthesis
- Routing di tutta la comunicazione subagent attraverso il coordinator per osservabilità

### Task 1.3 — Subagent Invocation e Context Passing

**Concetti chiave:**
- `Task` tool come meccanismo per spawning subagent; `allowedTools` deve includere `"Task"` per il coordinator
- Il contesto del subagent deve essere **fornito esplicitamente nel prompt** — non eredità automatica
- `AgentDefinition`: descriptions, system prompts, tool restrictions per ogni tipo di subagent
- Fork-based session management per esplorare approcci divergenti da una baseline comune

**Skills:**
- Includere i risultati completi dei subagent precedenti nel prompt del successivo
- Usare formati strutturati per separare content da metadata (source URLs, doc names, page numbers)
- Spawning parallelo: emettere **più `Task` calls in un singolo response** del coordinator
- Prompt del coordinator basati su goal e quality criteria, non istruzioni procedurali step-by-step

### Task 1.4 — Multi-Step Workflows con Enforcement

**Concetti chiave:**
- Differenza tra enforcement programmatico (hooks, prerequisite gates) e prompt-based guidance
- Quando è richiesta compliance deterministica, le istruzioni nel prompt hanno un **tasso di fallimento non zero**
- Structured handoff protocol per escalation a mid-process: customer details, root cause, recommended actions

**Skills:**
- Prerequisiti programmatici che bloccano tool call downstream (es. bloccare `process_refund` finché `get_customer` non ha restituito un verified customer ID)
- Decomposizione di richieste multi-concern in item distinti, investigazione parallela, sintesi unificata
- Handoff summary strutturati per agenti umani che non hanno accesso alla transcript

### Task 1.5 — Agent SDK Hooks

**Concetti chiave:**
- `PostToolUse` hooks per intercettare tool results prima che il modello li elabori
- Hooks per intercettare tool call in uscita e applicare regole di compliance (es. blocco rimborsi sopra threshold)
- Hooks → garanzie deterministiche; prompt → compliance probabilistica

**Skills:**
- `PostToolUse` per normalizzare formati eterogenei (Unix timestamps, ISO 8601, numeric status codes)
- Interception hooks che bloccano azioni policy-violating e redirectano a workflow alternativi
- Scegliere hooks su prompt quando le regole di business richiedono **guaranteed compliance**

### Task 1.6 — Task Decomposition

**Concetti chiave:**
- Fixed sequential pipelines (prompt chaining) per task predictabili multi-aspetto
- Adaptive decomposition per task open-ended, basata su intermediate findings
- Valore dei piani di investigazione che generano subtask in base a ciò che viene scoperto

**Skills:**
- Prompt chaining per code review: analisi per file locale, poi pass cross-file di integrazione
- Decomposizione open-ended: prima mappare struttura, identificare aree high-impact, poi piano prioritizzato che si adatta

### Task 1.7 — Session State, Resumption e Forking

**Concetti chiave:**
- `--resume <session-name>` per continuare una sessione specifica
- `fork_session` per branch indipendenti da una baseline analitica condivisa
- Iniziare una nuova sessione con summary strutturato è più affidabile di riprendere con tool results stale

**Skills:**
- `--resume` con session names per continuare sessioni di investigazione named
- `fork_session` per esplorare strategie parallele (es. confrontare due approcci di refactoring)

---

## Dominio 2: Tool Design & MCP Integration (18%)

### Task 2.1 — Tool Interface Design

**Concetti chiave:**
- Le tool descriptions sono il **meccanismo primario** che gli LLM usano per la selezione dei tool
- Descrizioni ambigue o sovrapposte causano misrouting
- Il wording del system prompt influenza la selezione: keyword-sensitive instructions creano associazioni indesiderate

**Skills:**
- Tool descriptions che differenziano chiaramente scopo, input attesi, output e quando usarli vs alternative simili
- Rinominare tool e aggiornare descriptions per eliminare overlap funzionale
- Splittare tool generici in tool purpose-specific con contratti input/output definiti

### Task 2.2 — Structured Error Responses per MCP Tools

**Concetti chiave:**
- `isError` flag MCP per comunicare fallimenti all'agente
- Categorie di errore: transient (timeout), validation (input invalido), business (policy violation), permission
- Errori uniformi generici (`"Operation failed"`) impediscono recovery decisions appropriate
- Distinzione tra errori ritentabili e non ritentabili

**Skills:**
- Restituire metadata strutturati: `errorCategory`, `isRetryable` boolean, descrizione human-readable
- Flag `retriable: false` e spiegazione customer-friendly per violazioni di regole business
- Recovery locale nei subagent per errori transient; propagare solo errori non risolvibili con partial results

### Task 2.3 — Distribuzione Tool e tool_choice

**Concetti chiave:**
- Troppi tool (18 invece di 4-5) degradano l'affidabilità della selezione aumentando la complessità decisionale
- Agenti con tool fuori dalla loro specializzazione tendono a misusarli
- `tool_choice` options: `"auto"`, `"any"`, forced selection `{"type": "tool", "name": "..."}`

**Skills:**
- Limitare il tool set di ogni subagent a quelli rilevanti per il suo ruolo
- `tool_choice: "any"` per garantire che il modello chiami un tool invece di restituire testo conversazionale
- Forced tool selection per garantire che uno specifico tool sia chiamato prima

### Task 2.4 — MCP Server Integration

**Concetti chiave:**
- Scoping: project-level (`.mcp.json`) per team tooling condiviso; user-level (`~/.claude.json`) per server personali/sperimentali
- Environment variable expansion in `.mcp.json` (es. `${GITHUB_TOKEN}`) per credential management senza committare segreti
- MCP resources per esporre content catalogs (issue summaries, doc hierarchies, schema DB) riducendo exploratory tool calls

**Skills:**
- Configurare MCP server condivisi in `.mcp.json` con env var expansion per token di autenticazione
- Scegliere community MCP server esistenti per integrazioni standard; custom server per workflow team-specific
- Esporre content catalogs come MCP resources per visibilità dei dati senza tool calls esplorative

### Task 2.5 — Built-in Tools

| Tool | Uso |
|---|---|
| `Grep` | Ricerca contenuto file per pattern (function names, error messages, import statements) |
| `Glob` | Matching percorsi file per pattern (es. `**/*.test.tsx`) |
| `Read` / `Write` | Operazioni file complete |
| `Edit` | Modifiche targeted con unique text matching |
| `Bash` | Comandi shell |

- Quando `Edit` fallisce per match non-unico: usare `Read` + `Write` come fallback
- Approccio incrementale: partire con `Grep` per trovare entry point, poi `Read` per seguire import e tracciare flussi

---

## Dominio 3: Claude Code Configuration & Workflows (20%)

### Task 3.1 — CLAUDE.md Hierarchy

**Gerarchia CLAUDE.md:**
```
~/.claude/CLAUDE.md              ← user-level (non condiviso)
.claude/CLAUDE.md o CLAUDE.md   ← project-level (versionato)
subdirectory/CLAUDE.md          ← directory-level
```

- User-level settings si applicano solo a quell'utente — NON condivisi via version control
- `@import` syntax per referenziare file esterni e mantenere CLAUDE.md modulare
- `.claude/rules/` directory per file regola topic-specific come alternativa al CLAUDE.md monolitico

**Skills:**
- Diagnosticare problemi di hierarchy (es. nuovo team member che non riceve istruzioni perché sono a user-level)
- Splittare CLAUDE.md grandi in file topic-specific in `.claude/rules/` (es. `testing.md`, `api-conventions.md`)
- `/memory` command per verificare quali file di memoria sono caricati

### Task 3.2 — Custom Slash Commands e Skills

**Concetti chiave:**
- Project-scoped commands in `.claude/commands/` (condivisi via version control)
- User-scoped commands in `~/.claude/commands/` (personali)
- Skills in `.claude/skills/` con file `SKILL.md` e frontmatter: `context: fork`, `allowed-tools`, `argument-hint`
- `context: fork` esegue la skill in contesto sub-agent isolato, evitando che output verbosi inquinino la sessione principale

**Skills:**
- `context: fork` per skill con output verbose (codebase analysis) o esplorativi (brainstorming)
- `allowed-tools` nel frontmatter per restringere l'accesso ai tool durante l'esecuzione della skill
- `argument-hint` per richiedere parametri obbligatori quando la skill viene invocata senza argomenti
- Scegliere tra skills (on-demand, task-specific) e CLAUDE.md (always-loaded, standard universali)

### Task 3.3 — Path-Specific Rules

**Concetti chiave:**
- File `.claude/rules/` con YAML frontmatter `paths` contenente glob pattern per attivazione condizionale
- Le regole path-scoped si caricano solo quando si modificano file corrispondenti → riduzione contesto irrilevante
- Vantaggio rispetto a directory-level CLAUDE.md per convenzioni che si estendono su più directory (es. test file sparsi)

```yaml
# Esempio frontmatter in .claude/rules/testing.md
paths:
  - "**/*.test.tsx"
  - "**/*.spec.ts"
```

### Task 3.4 — Plan Mode vs Direct Execution

| Situazione | Approccio |
|---|---|
| Refactoring architetturale, migrazione librerie su 45+ file, scelta tra approcci con requisiti infrastrutturali diversi | **Plan mode** |
| Bug fix single-file con stack trace chiaro, aggiunta di una validazione a una funzione | **Direct execution** |
| Discovery verbosa per evitare context window exhaustion | **Explore subagent** |

- Plan mode: esplorazione sicura del codebase e design prima di committare cambiamenti
- Combinare: plan mode per investigazione + direct execution per implementazione

### Task 3.5 — Iterative Refinement

**Tecniche:**
- **Input/output examples** (2-3 esempi concreti): più efficaci di descrizioni in prosa quando le trasformazioni sono interpretate inconsistentemente
- **Test-driven iteration**: scrivere test suite prima, poi iterare condividendo test failures
- **Interview pattern**: chiedere a Claude di fare domande per emergere considerazioni non anticipate
- **Single vs sequential message**: fornire tutti i problemi in un unico messaggio se le fix interagiscono; sequenziale se indipendenti

### Task 3.6 — Claude Code in CI/CD

**Opzioni CLI:**
```bash
claude -p "prompt"                          # Non-interactive mode (non blocca su input)
claude --output-format json                 # Output JSON machine-parseable
claude --output-format json --json-schema schema.json  # Output con schema enforcement
```

**Skills:**
- Session isolation: usare un'istanza Claude **separata** per review del codice generato (stessa sessione = context bias)
- Includere findings di review precedenti nel contesto per riportare solo nuovi problemi su commit successivi
- Fornire test file esistenti per evitare scenari duplicati in test generation
- Documentare testing standards, valuable test criteria e fixture in CLAUDE.md

---

## Dominio 4: Prompt Engineering & Structured Output (20%)

### Task 4.1 — Criteri Espliciti per Ridurre False Positives

**Concetti chiave:**
- Istruzioni esplicite (es. "flag commenti solo quando il comportamento dichiarato contraddice il codice") vs vaghe ("controlla che i commenti siano accurati")
- Istruzioni generiche come "be conservative" non migliorano la precision
- Alto tasso di false positive mina la fiducia in categorie accurate

**Skills:**
- Criteri di review che definiscono quali issue riportare (bug, security) vs skippare (stile minore, pattern locali)
- Disabilitare temporaneamente categorie ad alto false-positive mentre si migliorano i prompt
- Criteri di severità con esempi di codice concreti per ogni livello

### Task 4.2 — Few-Shot Prompting

**Concetti chiave:**
- Few-shot examples = tecnica più efficace per output consistentemente formattato quando istruzioni dettagliate da sole non bastano
- Dimostrano la gestione di casi ambigui (selezione tool, branch-level test coverage gap)
- Riducono le allucinazioni in task di extraction (misure informali, strutture documento variabili)

**Skills:**
- 2-4 esempi targeted per scenari ambigui che mostrano il ragionamento su perché un'azione è stata scelta rispetto ad alternative plausibili
- Esempi che dimostrano il formato di output desiderato (location, issue, severity, suggested fix)
- Esempi per distinguere pattern di codice accettabili da issue genuine → riduzione false positive

### Task 4.3 — Structured Output con Tool Use e JSON Schema

**Concetti chiave:**
- `tool_use` con JSON schema = approccio più affidabile per output schema-compliant garantito
- `tool_choice: "auto"` → il modello può restituire testo invece di chiamare un tool
- `tool_choice: "any"` → il modello deve chiamare un tool (scelta libera quale)
- Forced: `{"type": "tool", "name": "..."}` → il modello deve chiamare quel tool specifico
- JSON schema strict elimina syntax errors ma **non** gli errori semantici (es. line items che non sommano al totale)
- Schema design: `required` vs `optional` fields, `enum` con `"other"` + detail string per categorie estensibili

**Skills:**
- `tool_choice: "any"` per garantire output strutturato quando esistono più extraction schema e il tipo documento è sconosciuto
- Forced tool selection per garantire che un'estrazione specifica avvenga prima di enrichment steps
- Campi nullable per documenti che potrebbero non contenere l'informazione → evita fabricazione di valori
- Aggiungere `"unclear"` nelle enum per casi ambigui

### Task 4.4 — Validation, Retry e Feedback Loops

**Concetti chiave:**
- **Retry-with-error-feedback**: appendere gli errori di validazione specifici al prompt al retry
- Retry inefficace quando l'informazione richiesta è semplicemente assente dal documento sorgente (vs errori di formato/struttura)
- `detected_pattern` field per tracciare quali costrutti di codice triggherano findings → analisi sistematica dei pattern di dismissal

**Skills:**
- Follow-up request che include documento originale, estrazione fallita ed errori specifici di validazione
- Identificare quando i retry saranno inefficaci (informazione non presente) vs efficaci (format mismatch, errori strutturali)
- Self-correction validation: estrarre `"calculated_total"` oltre a `"stated_total"` per flaggare discrepanze

### Task 4.5 — Batch Processing

**Concetti chiave:**
- **Message Batches API**: 50% risparmio costi, finestra di elaborazione fino a 24 ore, **no latency SLA garantita**
- Adatto per: report overnight, audit settimanali, test generation notturna
- **Non adatto** per workflow bloccanti (pre-merge checks)
- Non supporta multi-turn tool calling all'interno di una singola request
- `custom_id` per correlare request/response pairs

**Skills:**
- Sincronous API per pre-merge checks bloccanti; Batch API per analisi overnight/settimanale
- Calcolare frequenza di submission batch basata su SLA constraints
- Gestire fallimenti: resubmit solo documenti falliti (identificati da `custom_id`) con modifiche appropriate
- Raffinare prompt su sample set prima del batch processing per massimizzare first-pass success rate

### Task 4.6 — Multi-Instance e Multi-Pass Review

**Concetti chiave:**
- Self-review limitations: il modello mantiene contesto di ragionamento dalla generazione → meno probabile mettere in discussione le sue decisioni nella stessa sessione
- Istanze di review indipendenti (senza contesto del ragionamento precedente) più efficaci del self-review
- Multi-pass: split per-file (local analysis) + pass cross-file (integration) per evitare attention dilution

**Skills:**
- Seconda istanza Claude indipendente per review del codice generato
- Split di review multi-file in pass per-file locali + pass integration separato per data flow cross-file
- Verification pass dove il modello auto-riporta confidence scores per ogni finding

---

## Dominio 5: Context Management & Reliability (15%)

### Task 5.1 — Context Management in Sessioni Lunghe

**Concetti chiave:**
- Progressive summarization risks: condensare valori numerici, percentuali, date, aspettative del cliente in summary vaghi
- **"Lost in the middle"**: i modelli processano informazioni all'inizio e alla fine degli input lunghi ma possono omettere findings dalle sezioni centrali
- Tool results si accumulano nel contesto consumando token in modo sproporzionato rispetto alla loro rilevanza

**Skills:**
- Estrarre fatti transazionali (importi, date, order numbers, stati) in un blocco "case facts" persistente, separato dalla storia summarizzata
- Trim degli output verbose dei tool ai soli campi rilevanti prima che si accumulino nel contesto
- Placing key findings summaries all'inizio di input aggregati; organizzare risultati dettagliati con section headers espliciti
- Subagent che restituiscono dati strutturati (key facts, citations, relevance scores) invece di verbose content e reasoning chains

### Task 5.2 — Escalation e Ambiguity Resolution

**Concetti chiave:**
- Trigger appropriati per escalation: richiesta esplicita del cliente per un umano, policy exceptions/gap (non solo casi complessi), impossibilità di fare progressi significativi
- Escalation immediata quando il cliente la richiede esplicitamente, senza tentare prima di indagare
- Sentiment-based escalation e self-reported confidence scores = proxy inaffidabili per la complessità reale del caso
- Multiple customer matches → chiedere identificatori aggiuntivi, non selezione euristica

**Skills:**
- Criteri di escalation espliciti con few-shot examples nel system prompt
- Onorare immediatamente le richieste esplicite di agente umano
- Escalation quando la policy è ambigua o silenziosa sulla richiesta specifica del cliente

### Task 5.3 — Error Propagation in Sistemi Multi-Agent

**Concetti chiave:**
- Contesto di errore strutturato (failure type, attempted query, partial results, alternative approaches) = recovery decisions intelligenti del coordinator
- Distinzione tra access failure (timeout → decisione di retry) e valid empty result (query riuscita senza match)
- Anti-pattern: sopprimere silenziosamente errori (restituire empty results come successo) OR terminare l'intero workflow su singoli fallimenti

**Skills:**
- Restituire contesto di errore strutturato: failure type, cosa è stato tentato, partial results, alternative potenziali
- Subagent con recovery locale per errori transient; propagare solo errori non risolvibili con partial results
- Output di sintesi con coverage annotations che indicano finding ben-supportati vs gap per fonti non disponibili

### Task 5.4 — Context Management in Large Codebase Exploration

**Concetti chiave:**
- Context degradation in sessioni estese: il modello inizia a dare risposte inconsistenti e referenzia "typical patterns" invece di classi specifiche scoperte in precedenza
- Scratchpad files per persistere key findings oltre i confini del context
- Subagent delegation per isolare verbose exploration output
- Structured state persistence per crash recovery: ogni agente esporta stato in una posizione nota, il coordinator carica un manifest al resume

**Skills:**
- Spawning subagent per domande specifiche mantenendo il main agent per la coordinazione high-level
- Agenti che mantengono scratchpad files con key findings, referenziandoli per domande successive
- `/compact` per ridurre l'uso del contesto durante sessioni di esplorazione estese
- Crash recovery con structured agent state exports (manifests) caricati al resume

### Task 5.5 — Human Review Workflows e Confidence Calibration

**Concetti chiave:**
- Metriche di accuracy aggregate (es. 97% overall) possono mascherare performance scarse su specifici tipi di documento
- Stratified random sampling per misurare error rate su estrazioni high-confidence e rilevare pattern di errore nuovi
- Field-level confidence scores calibrati su labeled validation set per routing del review

**Skills:**
- Stratified random sampling di estrazioni high-confidence per misura ongoing dell'error rate
- Analisi accuracy per tipo di documento e campo prima di ridurre il human review
- Modelli che outputtano field-level confidence scores → calibrare threshold con labeled validation set
- Routing estrazioni low-confidence o documenti sorgente ambigui/contraddittori a human review

### Task 5.6 — Information Provenance in Multi-Source Synthesis

**Concetti chiave:**
- La source attribution si perde durante i passi di summarization quando i findings vengono compressi senza preservare claim-source mappings
- Statistiche in conflitto da fonti credibili → annotare conflitti con source attribution invece di selezionare arbitrariamente
- Dati temporali: richiedere publication/collection dates per prevenire differenze temporali interpretate come contraddizioni

**Skills:**
- Subagent che outputtano structured claim-source mappings (source URLs, doc names, relevant excerpts) preservati attraverso la synthesis
- Report con sezioni distinte: well-established findings vs contested ones
- Completare document analysis con conflicting values inclusi e annotati esplicitamente
- Render di diversi content type: financial data come tabelle, news come prosa, technical findings come structured lists

---

## Sample Questions

### Scenario 1: Customer Support Agent

**Q1 — Enforcement tool sequence**

> In produzione il 12% dei casi l'agente salta `get_customer` e chiama `lookup_order` usando solo il nome del cliente → account errati e rimborsi incorretti.

**Risposta corretta: A** — Aggiungere un prerequisito programmatico che blocca `lookup_order` e `process_refund` finché `get_customer` non ha restituito un verified customer ID.

*Perché non B/C (system prompt / few-shot)*: compliance probabilistica, insufficiente per conseguenze finanziarie. *Perché non D* (routing classifier): affronta la disponibilità dei tool, non l'ordine.

---

**Q2 — Tool description per selezione affidabile**

> L'agente chiama spesso `get_customer` per query sugli ordini invece di `lookup_order`. Entrambi hanno descrizioni minimali.

**Risposta corretta: B** — Espandere le descriptions di ogni tool con formato input, query di esempio, edge case e boundary conditions.

*Perché non A* (few-shot): token overhead senza risolvere la causa root. *Perché non C* (routing layer): over-engineered, bypassa NLU. *Perché non D* (consolidare in un tool): scelta architetturale valida ma eccessiva come "first step".

---

**Q3 — Escalation calibration**

> L'agente raggiunge solo 55% FCR: escala casi semplici (damage replacement con foto) e gestisce autonomamente casi che richiedono policy exception.

**Risposta corretta: A** — Aggiungere criteri di escalation espliciti con few-shot examples nel system prompt.

*Perché non B* (confidence score auto-riportato): LLM confidence mal calibrata. *Perché non C* (classifier su historical tickets): over-engineered senza aver provato ottimizzazione del prompt.

---

### Scenario 2: Code Generation con Claude Code

**Q4 — Project-scoped slash command**

> Creare un comando `/review` disponibile a tutti i developer quando clonano il repo.

**Risposta corretta: A** — In `.claude/commands/` nel repository del progetto.

---

**Q5 — Plan mode per microservizi**

> Ristrutturare applicazione monolitica in microservizi: decine di file, decisioni su service boundaries.

**Risposta corretta: A** — Entrare in plan mode per esplorare il codebase e progettare l'implementazione prima di fare cambiamenti.

---

**Q6 — Path-specific rules per convenzioni multiple**

> React hooks, API handlers async/await, repository pattern per DB, test file sparsi ovunque con stesse convenzioni.

**Risposta corretta: A** — `.claude/rules/` con YAML frontmatter glob patterns (es. `**/*.test.tsx`).

---

### Scenario 4: Multi-Agent Research

**Q7 — Coverage incompleta su topic broad**

> Sistema multi-agente su "AI impact on creative industries" copre solo visual arts, manca music, writing, film.

**Risposta corretta: A** — Coordinator che partiziona esplicitamente lo scope di ricerca tra subagent (es. assegnando subtopic o tipi di fonte distinti a ogni subagent).

---

### Scenario 5: CI/CD

**Q11 — Batch API vs real-time**

> Pre-merge checks bloccanti + technical debt reports overnight. Proposta: migrare tutto al batch API per 50% cost savings.

**Risposta corretta: A** — Usare batch solo per i technical debt reports; mantenere real-time per i pre-merge checks bloccanti.

---

### Scenario 6: Structured Data Extraction

**Q12 — Multi-pass review per 14 file**

> Single-pass su 14 file produce risultati inconsistenti: feedback superficiale su alcuni file, bug ovvi mancati, feedback contraddittori.

**Risposta corretta: A** — Splittare in pass focalizzati: analisi per file per issue locali + pass integration separato per data flow cross-file.

---

## Esercizi di Preparazione

### Exercise 1 — Multi-Tool Agent con Escalation Logic

*Domini: D1, D2, D5*

1. Definire 3-4 MCP tool con descriptions dettagliate (almeno 2 con funzionalità simile che richiedono descrizione attenta)
2. Implementare agentic loop basato su `stop_reason`
3. Aggiungere structured error responses: `errorCategory`, `isRetryable`, descrizione human-readable
4. Implementare hook che intercetta tool calls per enforcement di regola business (es. blocco sopra threshold)
5. Testare con messaggi multi-concern: decomposizione, gestione parallela, sintesi unificata

### Exercise 2 — Claude Code per Team Dev Workflow

*Domini: D3, D2*

1. Creare project-level `CLAUDE.md` con coding standards e testing conventions universali
2. Creare `.claude/rules/` con glob path scoping per aree diverse (`src/api/**/*`, `**/*.test.*`)
3. Creare project-scoped skill con `context: fork` e `allowed-tools` restrictions
4. Configurare MCP server in `.mcp.json` con env var expansion; server sperimentale in `~/.claude.json`
5. Testare plan mode vs direct execution su task di complessità variabile

### Exercise 3 — Structured Data Extraction Pipeline

*Domini: D4, D5*

1. Definire extraction tool con JSON schema (required/optional, enum con "other", nullable fields)
2. Implementare validation-retry loop con errori specifici nel follow-up
3. Aggiungere few-shot examples per documenti con formati variabili
4. Progettare batch processing su 100 documenti con failure handling per `custom_id`
5. Implementare human review routing con field-level confidence scores

### Exercise 4 — Multi-Agent Research Pipeline

*Domini: D1, D2, D5*

1. Coordinator con `allowedTools: ["Task"]`, subagent con contesto esplicitamente fornito nel prompt
2. Parallel subagent execution: coordinator emette multipli `Task` calls in singolo response
3. Output strutturato per subagent: claim + evidence excerpt + source URL + publication date
4. Error propagation: simulare timeout subagent, verificare che il coordinator riceva structured error context
5. Test con dati conflittuali da fonti credibili: preservare entrambi i valori con source attribution

---

## Appendice — Tecnologie e Concetti

### In-scope

| Categoria | Concetti |
|---|---|
| Agent SDK | AgentDefinition, agentic loops, `stop_reason`, hooks (`PostToolUse`, interception), subagent via `Task`, `allowedTools` |
| MCP | Server, tools, resources, `isError`, tool descriptions, tool distribution, `.mcp.json`, env var expansion |
| Claude Code | `CLAUDE.md` hierarchy, `.claude/rules/` con path-scoping, `.claude/commands/`, `.claude/skills/` con frontmatter (`context: fork`, `allowed-tools`, `argument-hint`), plan mode, direct execution, `/memory`, `/compact`, `--resume`, `fork_session`, Explore subagent |
| Claude Code CLI | `-p`/`--print`, `--output-format json`, `--json-schema` |
| Claude API | `tool_use` con JSON schema, `tool_choice` options, `stop_reason`, `max_tokens`, system prompts |
| Message Batches API | 50% cost savings, 24h processing window, `custom_id`, no multi-turn tool calling |
| JSON Schema | required/optional, enum, nullable, "other"+detail, strict mode |
| Pydantic | Schema validation, semantic validation errors, validation-retry loops |
| Prompt Engineering | Few-shot targeting, prompt chaining, context window management, progressive summarization, lost-in-the-middle |
| Session management | Resumption, `fork_session`, named sessions, context isolation |

### Out-of-scope

- Fine-tuning o training di modelli custom
- Claude API authentication, billing, account management
- Deployment o hosting di MCP server (infrastruttura, networking, container orchestration)
- Architettura interna di Claude, processo di training, model weights
- Constitutional AI, RLHF, safety training methodologies
- Implementazione dettagliata di linguaggi o framework specifici
