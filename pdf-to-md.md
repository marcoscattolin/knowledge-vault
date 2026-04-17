---
name: pdf-to-md
description: >
  Converts a PDF into a structured Markdown note following the knowledge-vault standard.
  Use this skill whenever the user asks to convert, summarize, or import a PDF
  (paper, article, thesis, report) into the vault — even with casual phrasing like
  "aggiungi questo PDF", "converti", "importa il paper", "fai la nota di questo articolo",
  or when they drop a file path. Trigger also when the user mentions raw/pdf/,
  "clipping", or wants to create an Obsidian note from a document.
---

# PDF → Markdown Conversion

## 1. Estrai il testo dal PDF

Esegui questo script dalla directory in cui vuoi che venga salvato il markdown:

```python
import sys
import os
sys.stdout.reconfigure(encoding='utf-8')
from pypdf import PdfReader

os.makedirs('raw/pdf', exist_ok=True)

pdf_path = 'raw/pdf/nome_file.pdf'

r = PdfReader(pdf_path)
print('Pages:', len(r.pages))
for i, p in enumerate(r.pages):
    print(f'--- PAGE {i+1} ---')
    print(p.extract_text())

# cleanup: rimuovi il PDF e la cartella raw/pdf/ dopo l'estrazione
import shutil
os.remove(pdf_path)
shutil.rmtree('raw/pdf')
print('Rimossi: PDF e cartella raw/pdf/')
```

**Note:**
- Testo vuoto → PDF scansionato, non estraibile (richiede OCR). Annota nel frontmatter e salta la conversione.
- Output > 25 K token → leggi a blocchi con `offset` + `limit`.
- Paper > 30 pagine → estrai per sezioni: prima pagina (abstract/autori), heading strutturali, conclusioni.

---

## 2. Frontmatter YAML

```markdown
---
title: "Titolo esatto del paper/documento"
source: "URL arxiv/doi/web — lascia vuoto se non disponibile"
author:
  - "[[Nome Cognome]]"
  - "[[Nome Cognome2]]"
published: YYYY-MM-DD
created: YYYY-MM-DD
description: "Una frase: contributo principale + contesto applicativo"
tags:
  - "clippings"
  - "tag-topic-1"
  - "tag-topic-2"
---
```

**Regole:**
- `title`: copia esatto dal paper, non abbreviare
- `source`: preferisci URL arxiv (`https://arxiv.org/abs/XXXX.XXXXX`); se assente usa `""`
- `author`: `[[Nome Cognome]]` per ogni autore (Obsidian link)
- `published`: se solo anno noto → `YYYY-01-01`
- `description`: max 1 riga, focus su *cosa fa* e *a cosa serve*
- `tags`: sempre `"clippings"` come primo tag; aggiungi tag tematici in kebab-case

---

## 3. Struttura del corpo

```markdown
## Titolo Completo del Paper

### Autori (Istituzione/Azienda) — Anno

---

## Abstract

[Paragrafo sintetizzato: problema, metodo, risultato principale]

---

## 1 Introduzione

[Contesto, motivazione, gap che il paper risolve]

## 2 Sezione…

[Concetti chiave — NON trascrivere tutto, estrarre le idee principali]

…

## Conclusioni / Takeaway

[Messaggi principali, limiti, future work]

---

## Key References

- Autore (anno) — *Titolo breve* (ruolo nel paper)
```

**Regole:**
- Sezioni `##` seguono la numerazione originale del paper; sottosezioni `###`
- Formule: semplificate in testo leggibile, non LaTeX raw; usa `>` blockquote per le formule chiave
- Tabelle risultati: includi sempre se presenti nel paper
- Algoritmi: pseudocodice in blocco `code` o lista numerata
- Lunghezza target: 300–800 righe secondo la complessità del paper

---

## 4. Dove salvare i file

| Tipo file | Cartella | Nome file |
|-----------|----------|-----------|
| PDF originale | `raw/pdf/` | `Titolo_Breve.pdf` (snake_case, max ~40 char) |
| Markdown convertito | directory di esecuzione (`.`) | `Titolo_Breve.md` (stesso nome del PDF) |

---

## 5. Tag di riferimento

```
causal-inference, econometrics, DiD, uplift-modeling, treatment-effects,
bandits, contextual-bandits, reinforcement-learning, dynamic-pricing,
price-elasticity, airline, recommendation-systems, meta-learning,
multi-objective, machine-learning, marketing, lecture-notes
```

---

## 6. Casi speciali

| Caso | Azione |
|------|--------|
| PDF scansionato (testo vuoto) | Annota in `description` che non è estraibile; salta la conversione |
| Tesi/report senza URL | `source: ""` |
| Paper senza data esatta | `YYYY-01-01` |
| Abstract molto lungo | Sintetizza in 3–5 frasi (problema, metodo, risultato principale) |
| Paper con molte appendici | Copri solo quelle rilevanti, accenna le altre in una riga |

---

## 7. Checklist prima di salvare

- [ ] Frontmatter completo (tutti i campi valorizzati o esplicitamente vuoti)
- [ ] `description` cattura il contributo principale in una riga
- [ ] Tutte le sezioni principali del paper sono presenti
- [ ] Formule chiave incluse (semplificate)
- [ ] Tabella risultati inclusa se presente
- [ ] Key References alla fine
- [ ] Tag tematici coerenti con gli altri file nella vault
