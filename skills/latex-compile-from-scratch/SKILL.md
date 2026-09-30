---
name: latex-compile-from-scratch
description: Compile any LaTeX document to PDF in this environment with high reliability. Use whenever the user wants a `.tex` file turned into a `.pdf`, including Danish text, TikZ icons, simple tables, and one-page summaries. Covers toolchain detection, package-availability fallbacks, multi-pass compilation for references, and verification. Skip if the user is asking for academic-journal submission, Beamer presentations, or complex TikZ animations — those have their own skills.
---

# LaTeX Compile-from-Scratch (this environment)

## When to use

- User hands you a `.tex` file or asks for one and wants a PDF
- Danish/nordic text (`[danish]{babel}` / `[utf8]{inputenc}` / `[T1]{fontenc}`)
- A4 or letter, single-pass, one to ten pages
- Light TikZ (icons, simple shapes), tables, lists
- No external image files; no `\input{}` of user-supplied PDFs

If the request involves **Beamer slides**, **academic-journal submission**, or **shell-escape required TikZ**, defer to the matching specialized skill instead.

## Toolchain reality check

This environment may or may not have:

| Engine    | Binary      | Status                              |
|-----------|-------------|-------------------------------------|
| pdfTeX    | `pdflatex`  | Almost always present (TeX Live)    |
| XeTeX     | `xelatex`   | Sometimes                           |
| LuaTeX    | `lualatex`  | Sometimes                           |
| BibTeX    | `bibtex`    | Yes, but only needed with `.bib`    |

**First call before doing anything else:**

```bash
which pdflatex xelatex lualatex bibtex
```

Default to `pdflatex` — it is the most reliable engine in this environment and the only one guaranteed to produce a stable PDF without font-substitution warnings.

## Mandatory preamble for Danish text

```latex
\documentclass[10pt,a4paper]{article}
\usepackage[utf8]{inputenc}
\usepackage[T1]{fontenc}
\usepackage[danish]{babel}
\usepackage[margin=1.6cm]{geometry}
\usepackage{parskip}
\usepackage{xcolor}
```

If you skip any of these four (`inputenc`, `fontenc`, `babel`, `geometry`), Danish characters and line wrapping will silently break.

## Packages that are almost always available

`array`, `tabularx`, `booktabs`, `multicol`, `enumitem`, `tikz`, `xcolor`, `geometry`, `parskip`, `hyperref`, `graphicx`, `titlesec`, `babel`, `inputenc`, `fontenc`.

## Packages that are NOT installed (learned the hard way)

- `fontawesome5` — **not present**. Do not use. Inline TikZ icons instead.
- `fontawesome` (legacy) — also unreliable.
- `emoji` package — **not present**.
- `material-icons` — **not present**.

If you need icons, **always draw them with TikZ** — five lines per icon is enough for a circle, a square, an arrow, a play triangle, a lock, a cloud, a Sigma symbol, etc.

## Verification protocol — do not skip

```bash
cd /path/to/file
pdflatex -interaction=nonstopmode -halt-on-error file.tex  # strict first pass
pdflatex -interaction=nonstopmode file.tex                 # second pass for refs
ls -la file.pdf
pdfinfo file.pdf | grep -E "Pages|File size"
pdftotext file.pdf - | head -30                              # confirm text is extractable
```

If `pdflatex` exits non-zero:
1. Read the FIRST error in the log, not the last one. TeX errors often cascade.
2. Check for `! LaTeX Error: File 'X' not found.` — package missing, swap to a TikZ alternative.
3. Check for `! Undefined control sequence.` — typo in command or unsupported package option.
4. Never re-run blindly — fix the root cause first.

## Common pitfalls and their fixes

**Danish characters come out as `�` or wrong glyphs.** Missing `[utf8]{inputenc}` and/or `[T1]{fontenc}`. Add both.

**Babel warning: "No hyphenation patterns were loaded for (babel) Danish".** TeX Live base install has Danish patterns but you must declare `[danish]{babel}` in the preamble and pass `[utf8]` encoding through inputenc.

**PDF is generated but page count is 0.** Compilation hit a fatal error after the previous page was finished. Look for the FIRST error in the log, fix it, recompile from scratch (delete `.aux` and `.log`).

**`! LaTeX Error: File 'fontawesome5.sty' not found.`** Switch to inline TikZ icons. See "icon library" below.

**Compile fails with `! Emergency stop.` or runaway argument.** Long unbraced argument to a fragile command. Wrap the text in `{}` or use `\protect`.

## Inline TikZ icon library (drop-in replacements for common emojis)

Use these as drop-in `\newcommand` blocks at the top of the document. They render at any zoom level, are color-bound to `\color{accent}`, and look clean in print.

```latex
\usepackage{tikz}

% Book
\newcommand{\iconbook}{%
\begin{tikzpicture}[baseline=-0.5ex,line width=0.6pt]
\draw[accent,fill=accent!85] (0,0) rectangle (0.7,0.7);
\draw[white] (0.35,0.05) -- (0.35,0.65);
\draw[white] (0.1,0.1) -- (0.3,0.1);
\draw[white] (0.1,0.2) -- (0.3,0.2);
\draw[white] (0.1,0.3) -- (0.3,0.3);
\draw[white] (0.4,0.1) -- (0.6,0.1);
\draw[white] (0.4,0.2) -- (0.6,0.2);
\draw[white] (0.4,0.3) -- (0.6,0.3);
\end{tikzpicture}}

% Money / coin
\newcommand{\iconmoney}{%
\begin{tikzpicture}[baseline=-0.5ex]
\draw[accent,fill=accent,line width=0.6pt] (0,0) circle (0.35);
\node[white,font=\bfseries\small] at (0,0) {\$};
\end{tikzpicture}}

% Arrow right
\newcommand{\iconarrow}{%
\begin{tikzpicture}[baseline=-0.5ex,line width=1.2pt,accent,->]
\draw (0,0) -- (0.7,0);
\end{tikzpicture}}

% Play triangle
\newcommand{\iconplay}{%
\begin{tikzpicture}[baseline=-0.5ex]
\fill[accent] (0,0) -- (0.7,0.35) -- (0,0.7) -- cycle;
\end{tikzpicture}}

% Sigma (math)
\newcommand{\iconsigma}{%
\begin{tikzpicture}[baseline=-0.5ex]
\node[accent,font=\Large\bfseries] at (0,0) {$\Sigma$};
\end{tikzpicture}}

% Eye
\newcommand{\iconeye}{%
\begin{tikzpicture}[baseline=-0.5ex,line width=0.8pt]
\draw[accent] (0,0.25) ellipse (0.4 and 0.22);
\fill[accent] (0,0.25) circle (0.12);
\fill[white] (-0.05,0.28) circle (0.04);
\end{tikzpicture}}

% Refresh / loop (two opposing arcs)
\newcommand{\iconloop}{%
\begin{tikzpicture}[baseline=-0.5ex,line width=1.2pt,accent]
\draw[->] (0.6,0.35) arc (0:-180:0.3);
\draw (0,0.05) arc (-180:0:0.3);
\end{tikzpicture}}

% RSS / wifi waves
\newcommand{\iconrss}{%
\begin{tikzpicture}[baseline=-0.5ex,line width=1pt,accent,fill=accent]
\draw[fill=none] (0,0.1) arc (180:0:0.3);
\draw[fill=none] (0,0.1) arc (180:0:0.2);
\draw[fill=none] (0,0.1) arc (180:0:0.1);
\fill[accent] (0,0.1) circle (0.06);
\end{tikzpicture}}

% Padlock
\newcommand{\iconlock}{%
\begin{tikzpicture}[baseline=-0.5ex,line width=0.8pt]
\draw[accent] (0.15,0.3) -- (0.15,0.5) arc (180:0:0.2) -- (0.55,0.3);
\draw[accent,fill=accent] (0.05,0) rectangle (0.65,0.35);
\fill[white] (0.35,0.05) rectangle (0.37,0.18);
\end{tikzpicture}}

% Cloud
\newcommand{\iconcloud}{%
\begin{tikzpicture}[baseline=-0.5ex]
\fill[accent] (0.1,0.15) circle (0.22);
\fill[accent] (0.5,0.18) circle (0.26);
\fill[accent] (0.8,0.15) circle (0.2);
\fill[accent] (0,0.08) rectangle (0.95,0.18);
\end{tikzpicture}}
```

## Output delivery

After successful compile, copy the PDF to `~/` with a clean filename:

```bash
cp doc.pdf ~/<descriptive-name>.pdf
ls -la ~/<descriptive-name>.pdf
pdfinfo ~/<descriptive-name>.pdf | grep -E "Pages|File size"
```

Then send to the user with `MEDIA:` so Telegram delivers it as a native attachment.

## Anti-patterns (do not)

- ❌ Use `fontawesome5` — not installed in this environment
- ❌ Use emoji characters directly — they render as missing-glyph boxes
- ❌ Compile with `lualatex` for a document that does not need it — slower and the env may not have it
- ❌ Skip verification — a PDF with 0 pages or watermark means the compile silently failed
- ❌ Wrap icons in `\fbox` or `\framebox` — they look terrible
- ❌ Use `\usepackage[utf8x]{inputenc}` (the legacy one) — `utf8` is correct

## Cross-skill links

- For academic-paper workflow (data → figures → journal-style PDF): use `research/academic-paper-writer`
- For Beamer slides: there is no first-class skill in this archive yet; fallback to the same `pdflatex` toolchain with `\documentclass{beamer}` and `[danish]{babel}`
- For studying topic X with multiple visualisations: use `education/study-companion-latex` (sibling skill)
