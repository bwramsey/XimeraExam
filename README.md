# ximeraExam

**Version 1.7.1 — Printable exams built from Ximera**

`ximeraExam.sty` is a LaTeX package for creating printable exams with the Ximera document class. It supports points-based and standards-based assessments, reusable Ximera activities, student and answer-key versions, and customizable exam layouts.

Use it **with `ximera.cls`**; `ximeraExam.sty` is not a separate document class.

## Requirements

- A working Ximera LaTeX installation.
- LuaLaTeX and a recent LaTeX installation. The project's PDF/UA-2 workflow uses TeX Live 2026.
- `ximeraExam.sty` from this release, placed beside your exam `.tex` file or somewhere LaTeX can find it.

## Quick start

Save this example as `exam.tex`:

```latex
\DocumentMetadata{lang=en-US,pdfstandard=ua-2,tagging=on}
\documentclass{ximera}
\usepackage{unicode-math}
\usepackage[points]{ximeraExam}

\examtitle{Midterm 1}
\examcourse{Math 101}
\examdate{September 2026}
\examversion{Form A}

\noprintanswers % Student version (the default).
% \printanswers % For a key, replace the line above with this one.

\begin{document}

\makeexamcover

\begin{questions}

  \question[5] Solve \(2x+3=11\). Show your work.

  \begin{solution}[2in]
    Subtract 3 from both sides, then divide by 2:
    \(x=4\).
  \end{solution}

  \question[3] Evaluate \(2+3\).

  Final answer: \answerline{}

  \begin{solution}[\stretch{1}]
    \(2+3=5\).
  \end{solution}

\end{questions}

\end{document}
```

Compile **twice** with LuaLaTeX to resolve exam summaries and page-count references:

```bash
lualatex exam.tex
lualatex exam.tex
```

To produce an answer key, replace `\noprintanswers` with `\printanswers` and compile again.

## Main features

- **Points or standards:** Assign metadata to questions and nested parts, with automatic inheritance from child items.
- **Reusable activities:** Import an existing Ximera activity with `\ximeraQuestion[metadata]{activity.tex}`. The exam supplies the question number; nested problem-like environments become exam parts and subparts.
- **Student and answer-key versions:** Hide ordinary answers and solutions in student copies and reveal them in the key.
- **Response space:** Reserve fixed or stretchable student writing space with `\begin{solution}[2in]` or `\begin{solution}[\stretch{1}]`. Long solutions can continue across pages in the answer key.
- **Answer lines:** Add inline or standalone writing lines and configure their width, standalone alignment, and answer-key visibility. Inline lines remain visible in keys; standalone labeled lines are hidden in keys by default.
- **Exam layout:** Build a default or custom cover and configure running headers and footers.
- **Print-friendly choices:** In PDFs, `\wordChoice` prints as a vertical, lettered list. Ximera's interactive HTML behavior remains unchanged.

## Examples and documentation

- `points-template.tex` — starting point for a points-based exam.
- `standards-template.tex` — starting point for a standards-based exam, including an example of marking an individual standard.
- `calculus-worked-example.tex` — illustrative Calculus I worked example.
- `user-guide.md` — detailed instructions, customization settings, and command reference.
- `CHANGELOG.md` — changes included in this release.
- `regression.tex` and `import-regression.tex` — regression test sources; the import test uses the included `Q1SP17REDUX.tex` activity.

Start with the template matching your assessment mode. Consult **`user-guide.md`** for the full explanation of question structure, answer lines, answer-key behavior, activity imports, and page layout.

## PDF accessibility

The package is designed for a tagged PDF/UA-2 workflow. To run automated validation on a compiled PDF:

```bash
verapdf --defaultflavour ua2 exam.pdf
```

Automated validation does not replace reviewing reading order, mathematical content, graph descriptions, and other material in the finished exam. Imported activities may require additional accessibility review.
