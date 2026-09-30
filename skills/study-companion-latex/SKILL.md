---
name: study-companion-latex
description: Generate study notes and multi-modal visualisations on any topic as LaTeX PDFs, then deliver them to the user. Use when a user says "study companion", "studievejleder", "lav noter om X", "forklar emnet på flere måder", "lav en studyguide", "lav visuelle forklarer", "explain X with diagrams/tables/examples", or wants the same topic explained as outline + table + step-by-step + example + summary. Sibling skill to `latex-compile-from-scratch` (which handles the actual PDF build). The two should be loaded together.
---

# Study Companion — LaTeX notes & visualisations

## What this skill does

You are a personal study advisor. The user gives you a topic. You return a short, well-structured PDF (1–4 pages) that explains the topic in **multiple complementary ways**:

1. **Outline** — bird's-eye view, the 5–8 core ideas
2. **Comparison table** — contrast the concepts side by side
3. **Step-by-step walkthrough** — a worked example end to end
4. **Mnemonic / pattern** — a memorable hook
5. **Quick-reference summary** — the 60-second version

This works because every learner has a preferred mode (visual, tabular, sequential, narrative), and the same idea explained five ways lets at least one stick.

## Workflow

### Step 1 — Clarify the topic

If the topic is vague, ask **one** question before proceeding: "What level are we at — beginner, intermediate, exam-prep, or research?" Never ask more than one round. Pick sensible defaults if the user does not respond in time.

### Step 2 — Pick the visualisations

For each topic, decide **which three or four of the five modes** are best:

| Topic shape | Best modes |
|-------------|------------|
| Has clear parts to compare | Outline + Table + Example + Summary |
| Is procedural ("how to do X") | Step-by-step + Worked example + Mnemonic |
| Has competing definitions | Outline + Table + Mnemonic + Summary |
| Is conceptual / abstract | Outline + Diagram (TikZ) + Mnemonic + Summary |
| Is memorisation-heavy | Table + Mnemonic + Quick-test (5 questions with answers hidden) |

### Step 3 — Compose the LaTeX

Use the skeleton below. Adapt section count and ordering to the topic, but always end with a **summary** and (when relevant) a **self-test**.

#### Skeleton (drop-in, customise the topic)

```latex
\documentclass[10pt,a4paper]{article}
\usepackage[utf8]{inputenc}
\usepackage[T1]{fontenc}
\usepackage[danish]{babel}
\usepackage[margin=1.6cm]{geometry}
\usepackage{parskip}
\usepackage{xcolor}
\usepackage{array}
\usepackage{tabularx}
\usepackage{booktabs}
\usepackage{enumitem}
\usepackage{tikz}

\definecolor{accent}{HTML}{7C5CFC}
\definecolor{muted}{HTML}{6B7280}

\pagestyle{empty}
\setlist[itemize]{leftmargin=1.2em,itemsep=1pt,topsep=2pt}

\title{\Huge <TOPIC>}
\author{}
\date{}

\begin{document}
\maketitle

\section*{Overblik -- de 6 vigtigste pointer}
% 5-7 bullet points, each one line, plain language

\section*{Sammenligning}
% A booktabs table contrasting 3-5 concepts on 3-4 axes

\section*{Sådan fungerer det -- trin for trin}
% Numbered list, 5-9 steps, plain language

\section*{Et konkret eksempel}
% A worked example end-to-end, 4-8 lines

\section*{Huskeregel}
% A short, memorable pattern. Avoid "fake" mnemonics; prefer
% a true structural hook ("sandsynlighed går ALTID mellem 0 og 1")

\section*{Selvtest}
% 3-5 short questions. Hide answers in a TikZ overlay OR
% put answers on the last page with a clear "Facit -- først når du
% har svaret" marker.

\section*{60-sekunders-resumé}
% 4-5 bullet points that an exhausted student can read in one minute

\end{document}
```

### Step 4 — Compile

Delegate to `latex-compile-from-scratch` (the sibling skill):

1. Detect `pdflatex`.
2. Run `pdflatex -interaction=nonstopmode file.tex` twice (second pass resolves refs).
3. Verify with `pdfinfo` + `pdftotext | head` + visual spot-check via `pdftoppm` + `vision_analyze`.
4. Copy to `~/` and deliver via `MEDIA:` to Telegram.

### Step 5 — Offer follow-ups

Always end the response with one concrete next step the user can opt into:

- "Want me to extend this with [worked examples / common mistakes / a 20-question drill]?"
- "Should we add a section on [deeper sub-topic]?"
- "Want a one-page cheat-sheet version too?"

## Topic coverage recipes

These are starting points. Adapt freely.

### Algorithms & data structures

- Outline: complexity classes, common operations
- Table: time/space complexity for each operation
- Step-by-step: trace one full execution on a small input
- Example: realistic problem solved
- Mnemonic: which data structure maps to which problem pattern

### Mathematics (calc, linear algebra, stats)

- Outline: definitions, theorems
- Table: when to use which technique
- Step-by-step: derivation or worked problem
- Example: applied problem from real life
- Mnemonic: the single most useful identity

### Languages (vocab, grammar, conjugation)

- Outline: 5 most common patterns
- Table: tense / case / gender grid
- Step-by-step: build one sentence from scratch
- Example: 3 example sentences, each highlighting a different pattern
- Mnemonic: a memorable phrase that triggers the rule

### History / social science

- Outline: chronology of 5–8 events
- Table: actors, motivations, outcomes
- Step-by-step: causal chain for one event
- Example: a primary-source excerpt (if available) with close reading
- Mnemonic: a date or quote you cannot forget

### Programming language / library

- Outline: data types, control flow, I/O
- Table: method cheatsheet (5–10 most-used methods)
- Step-by-step: build a 10-line program from zero
- Example: realistic mini-project
- Mnemonic: a rule-of-thumb ("ask forgiveness, not permission" for Python)

### Concepts / theory (philosophy, biology, economics)

- Outline: the 5–8 core claims
- Table: opposing schools side by side
- Step-by-step: trace one mechanism end to end
- Example: a concrete case study
- Mnemonic: the one-line statement that captures the whole

## Pitfalls

- **Information overload** — 4 pages is the upper bound. If the topic needs more, split into "Part 1" and "Part 2" PDFs.
- **English drift** — if the user is writing Danish, write Danish. Detect language from the most recent user message; do not mix.
- **Cute-but-empty mnemonics** — never use fake rhyme-mnemonics that don't actually help. Prefer a structural hook.
- **Tables that don't fit** — 5 columns × 8 rows max. Anything larger, split.
- **TikZ overuse** — diagrams are great, but if they need shell-escape or complex node positioning, replace with a table or list.
- **Self-test without answers** — a self-test without answers is just noise. Always include an answer key (ideally below a fold-line or on a separate page).

## Delivery checklist

- [ ] Topic clearly named in the title and at top of each section
- [ ] At least three of the five modes present
- [ ] Self-test has an answer key
- [ ] 60-second summary present
- [ ] PDF compiled cleanly (no missing-glyph boxes, no orphan pages)
- [ ] PDF ≤ 4 pages
- [ ] PDF ≤ 200 KB (lighter is easier to share)
- [ ] Sent via `MEDIA:` to Telegram
- [ ] User given a concrete follow-up offer

## Cross-skill links

- **Always pair with `education/latex-compile-from-scratch`** — that skill knows the local toolchain quirks (no `fontawesome5`, what `pdflatex` flags to use, how to spot a silently-failed compile).
- For visualisation-heavy topics (graphs, charts), consider matplotlib → TikZ/PGF import via `pgfplots` if available; otherwise use SVG-to-PDF via `cairosvg` + `pdfpages`.
- For memory-of-this-session continuity, persist topic progress in `~/.hermes/memory/` so the next session can resume the same study thread.
