# PDF → Markdown Conversion Template

Questo file documenta lo standard per convertire PDF in markdown nella knowledge vault.
Usarlo come riferimento in ogni nuova sessione prima di convertire PDF.

---

## 1. Struttura del frontmatter YAML

```markdown
---
title: "Titolo esatto del paper/documento"
source: "URL arxiv/doi/web — lascia vuoto se non disponibile"
author:
  - "[[Nome Cognome]]"
  - "[[Nome Cognome2]]"
published: YYYY-MM-DD          # data di pubblicazione/submission
created: YYYY-MM-DD            # data di creazione della nota (oggi)
description: "Una frase che cattura il contributo principale e il contesto applicativo"
tags:
  - "clippings"
  - "tag-topic-1"              # es. causal-inference, bandits, dynamic-pricing
  - "tag-topic-2"
---
```

**Regole frontmatter**:
- `title`: copia esatto dal paper, non abbreviare
- `source`: URL arxiv preferito (es. `https://arxiv.org/abs/XXXX.XXXXX`); se tesi/report senza URL lascia `""`
- `author`: usa `[[Nome]]` per ogni autore (Obsidian link)
- `published`: data paper; se solo anno noto usa `YYYY-01-01`
- `description`: massimo 1 riga, focus su *cosa fa* e *a cosa serve*
- `tags`: sempre `"clippings"` come primo tag; aggiungi tag tematici in kebab-case

---

## 2. Struttura del corpo del documento

```markdown
## Titolo Completo del Paper

### Autori (Istituzione/Azienda) — Anno

---

## Abstract

[Paragrafo fedele all'abstract originale, sintetizzato se lungo]

---

## 1 Introduzione

[Contesto, motivazione, gap che il paper risolve]

## 2 Sezione...

[Contenuto sintetizzato — NON trascrivere tutto, estrarre i concetti chiave]

...

## Conclusioni / Takeaway

[Messaggi principali, limiti, future work]

---

## Key References

- Autore (anno) — *Titolo breve* (ruolo nel paper: es. "metodo esteso")
```

**Regole corpo**:
- Titolo `##` = sezioni principali del paper (segui la numerazione originale)
- Sottosezioni con `###`
- Formule: semplificate in testo leggibile, non LaTeX raw; usa `>` blockquote per le formule chiave
- Tabelle risultati: includi sempre se presenti nel paper
- Algoritmi: pseudocodice indentato in blocco `code` o lista numerata
- Lunghezza target: 300–800 righe a seconda della complessità

---

## 3. Come estrarre il testo dal PDF

```python
import sys
sys.stdout.reconfigure(encoding='utf-8')
from pypdf import PdfReader

r = PdfReader('raw/pdf/nome_file.pdf')
print('Pages:', len(r.pages))
for i, p in enumerate(r.pages):
    print(f'--- PAGE {i+1} ---')
    print(p.extract_text())
```

Eseguire da `C:\Users\Scattolin Marco\repos\PERSONAL_REPOS\knowledge-vault`.

**Note**:
- Se il testo è vuoto → PDF scansionato, non estraibile con pypdf (richiede OCR)
- Output > 25K token: salvato automaticamente in file temporaneo; leggerlo a blocchi con `offset` + `limit`
- Per paper molto lunghi (>30 pagine): estrarre per sezioni (prima pagina per abstract/autori, ricerca heading per struttura, pagine conclusioni)

---

## 4. Dove salvare i file

| Tipo file | Cartella | Nome file |
|-----------|----------|-----------|
| PDF originale | `raw/pdf/` | `Titolo_Breve.pdf` (snake_case, max ~40 char) |
| Markdown convertito | `raw/clippings/` | `Titolo_Breve.md` (stesso nome del PDF) |

---

## 5. Checklist prima di salvare

- [ ] Frontmatter completo (tutti i campi valorizzati o esplicitamente vuoti)
- [ ] `description` cattura il contributo principale in una riga
- [ ] Tutte le sezioni principali del paper sono presenti
- [ ] Formule chiave incluse (semplificate)
- [ ] Tabella risultati inclusa se presente
- [ ] Key References alla fine
- [ ] Tag tematici coerenti con gli altri file in `raw/clippings/`

---

## 6. Esempi di tag usati in questa vault

```
causal-inference, econometrics, DiD, uplift-modeling, treatment-effects,
bandits, contextual-bandits, reinforcement-learning, dynamic-pricing,
price-elasticity, airline, recommendation-systems, meta-learning,
multi-objective, machine-learning, marketing, lecture-notes
```

---

## 7. Casi speciali

| Caso | Cosa fare |
|------|-----------|
| PDF scansionato (testo vuoto) | Annotare nel frontmatter `description` che è non estraibile; saltare conversione |
| Tesi/report senza URL | Lasciare `source: ""` |
| Paper senza data esatta | Usare `YYYY-01-01` |
| Abstract molto lungo | Sintetizzare in 3-5 frasi mantenendo: problema, metodo, risultato principale |
| Paper con molte appendici | Coprire solo appendici rilevanti, accennare le altre in una riga |
