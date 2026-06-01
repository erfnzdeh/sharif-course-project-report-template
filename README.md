# Course Project Report Template (Sharif / TexReady)

The generic LaTeX template for a **course project report**, taken from
[latex.sharif.edu](https://latex.sharif.edu/). It was created by the
**TexReady Group** for Computer Science students and was originally published
on TexReady. It is built on **XeLaTeX + XePersian**.

I couldn't compile it locally on a current TeX Live, so this repository keeps
the original template (first commit) and the fixes that make it build again
(second commit). The problems and their fixes are documented below.

> **Compile with XeLaTeX, not pdfLaTeX** (it is a XePersian template).
>
> ```bash
> latexmk -xelatex Seminar.tex
> # or manually:
> xelatex Seminar.tex && bibtex Seminar && xelatex Seminar.tex && xelatex Seminar.tex
> ```

The template ships with everything it needs (fonts under `styles/fonts/`,
figures under `figs/`), so it builds out of the box on a current TeX Live
(2025/2026) installation.

---

## Why it didn't compile, and how it was fixed

The template was written for an older TeX Live and accumulated a few latent
bugs. On a current distribution it failed to produce any output. This repo
contains the original files (first commit) and the fixes (second commit). The
problems and their fixes are below.

### 1. Fatal — obsolete `xepersian` option (the main blocker)

`styles/common.tex` loaded:

```latex
\usepackage[localise=on,extrafootnotefeatures]{xepersian}
```

In **xepersian v26 (2026)** the option was renamed from the British spelling
`localise` to the American `localize`. The unknown option was forwarded to the
`bidi` package, producing a hard stop:

```
! LaTeX Error: Unknown option 'localise' for package bidi.
No pages of output.
```

**Fix:** `localise=on` → `localize=on`.

### 2. Fatal — fonts referenced by system name instead of the bundled files

The active font commands were:

```latex
\settextfont[Scale=1.166666666666666]{XBNiloofar}
\setdigitfont{Yas}
```

These look up fonts **installed on the machine**. Anyone without `XB Niloofar`
/ `Yas` installed got:

```
! I can't find file `XBNiloofar'.
! Emergency stop.
```

The fonts were actually shipped in `styles/fonts/` but never wired up.

**Fix:** load the bundled fonts via `Path`/`Extension` so no system install is
required:

```latex
\settextfont[
  Scale=1.166666666666666,
  Extension=.ttf,
  Path=styles/fonts/,
  BoldFont=XB NiloofarBd,
  ItalicFont=XB NiloofarIt,
  BoldItalicFont=XB NiloofarBdIt
]{XB Niloofar}
```

(and the same for `\setdigitfont`).

### 3. Document-class mismatch (`article` vs. chapter-based styles)

`Seminar.tex` uses `\documentclass{article}` and the body uses `\section`, but
the styles assumed a chapter-based class. This caused errors such as:

```
! LaTeX Error: No counter 'chapter' defined.
! LaTeX Error: Command \bibname undefined.
! Undefined control sequence.   % \فصل (=\chapter) in the appendix
```

**Fix (kept the `article` class, aligned everything to sections):**

- `styles/common.tex`: theorem counters `[chapter]` → `[section]`
- `styles/common.tex`: `\renewcommand{\bibname}` → `\renewcommand{\refname}`
- `chapters/appendix.tex`: `\فصل` (`\chapter`) → `\قسمت` (`\section`)
- `bibs/bibs.tex`: `\addcontentsline{toc}{chapter}` → `{section}`

### 4. Stray right-to-left control characters

Every line of `chapters/context.tex` (and `chapters/appendix.tex`) began with
an invisible **`U+202B` RIGHT-TO-LEFT EMBEDDING** character. XeTeX tried to
typeset it (`Missing character: There is no ‫ ...`), which injected text before
the first `\item` and inside tables, breaking list and table environments:

```
! LaTeX Error: Something's wrong--perhaps a missing \item.
! Misplaced \noalign.
```

XePersian already manages bidirectional text, so these marks were pure noise.

**Fix:** stripped all `U+202B` characters from the content files.

### 5. Undefined macro `\EnglishCourseTitle`

`front/template/title.tex` uses `\EnglishCourseTitle`, but `front/info.tex`
never defined it (`! Undefined control sequence`).

**Fix:** added a definition in `front/info.tex` (edit it with your own title):

```latex
\newcommand{\EnglishCourseTitle}
{Course Project Report Title}
```

---

## Result

After the fixes, the template builds cleanly:

- `latexmk -xelatex Seminar.tex` exits 0 with no errors
- `bibtex` resolves all references
- a complete PDF is produced

## Project layout

```
Seminar.tex            Main file (\input/\include of the parts below)
front/info.tex         Fill in your names, course, professor, title, etc.
front/template/        Title page, logo, "besmellah"
chapters/context.tex   Body of the report
chapters/appendix.tex  Appendix
bibs/                  Bibliography (.bib) and bibs.tex
figs/                  Figures
styles/                common.tex, custom.tex, fonts, packages, alg-fa.tex
```

## Getting started

1. Edit `front/info.tex` (Persian fields) and `\EnglishCourseTitle`.
2. Write your content in `chapters/context.tex`.
3. Add references to `bibs/full.bib` / `bibs/refs.bib`.
4. Build with `latexmk -xelatex Seminar.tex`.
