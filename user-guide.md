# ximeraExam 1.7.1 — User Guide

**Writing printable Ximera exams with points or standards**  
**Status:** Released version 1.7.1 (September 20, 2026).

`ximeraExam.sty` is a package used *with* `ximera.cls`, not a separate document class. It supplies exam questions and nested parts, imports existing Ximera activities, student/answer-key rendering, points or standards summaries, cover information, and customizable page furniture. It is intended for LuaLaTeX output; the PDF/UA-2 setup below requires a sufficiently recent LaTeX installation (tested by the project with TeX Live 2026).

## 1. Installation and first exam

Put `ximeraExam.sty` somewhere LaTeX can find it (for example, alongside your exam `.tex` file). Start a **points** exam with:

```latex
\DocumentMetadata{lang=en-US,pdfstandard=ua-2,tagging=on}
\documentclass{ximera}
\usepackage{unicode-math}
\usepackage[points]{ximeraExam}

\examtitle{Midterm 1}
\examcourse{MATH 101}
\examdate{July 18, 2026}
\examversion{Form A}
\noprintanswers % Optional: student mode is already the default.

\begin{document}
\makeexamcover
\begin{questions}
  \question
  \begin{parts}
    \part[2] Evaluate \(2+3=\answer{5}\).
    \part[3] Explain your reasoning.
  \end{parts}

  \question[5]
  Find the derivative of \(x^2\).
\end{questions}
\end{document}
```

The first question **inherits 5 points** from its parts; the second question has an explicit 5-point allocation. Run LuaLaTeX **twice** to populate a cover-page summary and page-count references. To create a standards exam, change `[points]` to `[standards]` and use standards codes instead of numeric point allocations (see §3).

**Build:** `lualatex exam.tex` twice. You may then validate the resulting PDF with `verapdf --defaultflavour ua2 exam.pdf`; see §12 for the limits of automated validation.

## 2. Questions and nested parts

`questions` establishes the top-level question sequence. Inside it use `\question[metadata]` or `\bonusquestion[metadata]`; the metadata argument is optional. These start items, **not environments**: write the item body after the command and start the next item when ready. In tagged PDF output, each visible **Question N** / **Bonus Question N** label is emitted as an H2 heading; nested `parts`, `subparts`, and `subsubparts` remain list structures.

```latex
\begin{questions}
  \question[6] Find the requested values.
  \begin{parts}
    \part[2] Part (a).
    \begin{subparts}
      \subpart[1] Subpart (i).
      \begin{subsubparts}
        \subsubpart[1] Subsubpart (A).
      \end{subsubparts}
    \end{subparts}
    \part[4] Part (b).
  \end{parts}

  \bonusquestion[2] Optional additional problem.
\end{questions}
```

Within `parts`, `subparts`, and `subsubparts` use `\part`, `\subpart`, and `\subsubpart` respectively. Each accepts optional metadata. A bonus question shares the normal question-number sequence; in points mode its points are tracked separately from the ordinary total. Use one `questions` environment for a complete exam: opening a new one resets the question counter and clears collected summary data.

The default visible labels are **Question 1**, `(a)`, `(i)`, `(A)`; a bonus is **Bonus Question 2**. The package's default point label is `[2 points]` and its default standards label is `[AL1]`. Explicit metadata prints beside the relevant item; metadata *computed by inheritance* does not print beside the parent item.

### Customize numbering and labels

Change these commands in the preamble with `\renewcommand`:

| Counter presentation | Default | Complete visible label | Default |
| --- | --- | --- | --- |
| `\questionnumber` | `\arabic{ximeraExamQuestion}` | `\questionlabel` | `\textbf{Question~\questionnumber}` |
| `\partnumber` | `\alph{ximeraExamPart}` | `\partlabel` | `(\partnumber)` |
| `\subpartnumber` | `\roman{ximeraExamSubpart}` | `\subpartlabel` | `(\subpartnumber)` |
| `\subsubpartnumber` | `\Alph{ximeraExamSubsubpart}` | `\subsubpartlabel` | `(\subsubpartnumber)` |

`\bonusquestionlabel` defaults to `\textbf{Bonus Question~\questionnumber}`. For example, to use `1)`, `a)`, `i)`:

```latex
\renewcommand{\questionlabel}{\questionnumber)}
\renewcommand{\partlabel}{\partnumber)}
\renewcommand{\subpartlabel}{\subpartnumber)}
```

These commands change *printed appearance*, not question IDs or grading metadata.

## 3. Points versus standards

Choose **one** mode when loading the package: `\usepackage[points]{ximeraExam}` (the default if no option is given) or `\usepackage[standards]{ximeraExam}`. Both modes use the same question structure; only the meaning of `[metadata]`, its inheritance, and the summary command differ.

### Points mode

```latex
\begin{questions}
  \question % Automatically worth 7 points as the sum of the two parts.
  \begin{parts}
    \part[2] First task.
    \part[5] Second task.
  \end{parts}
  \question[10] Explicitly worth 10 points.
  \bonusquestion[2] Bonus task.
\end{questions}
```

When a parent has no explicit value, the package **sums its child values** recursively. If a parent has an explicit value, it takes precedence over any child sum. The grade table reports regular points and bonus points separately. Supply numeric point values; these are assessment metadata, not percentages.

The visible point annotation can be redefined, for example:

```latex
\renewcommand{\pointsLabel}[1]{(#1 pts)}
```

### Standards mode

```latex
\usepackage[standards]{ximeraExam}
% ...
\begin{questions}
  \question % Inherits AL1, AL2, AL3, in first-appearance order.
  \begin{parts}
    \part[AL1,AL2] First task.
    \part[AL2,AL3] Second task.
  \end{parts}
  \question[AL4] An explicitly assigned standard.
\end{questions}
```

When a parent has no explicit standards, it inherits an **ordered, deduplicated union** of its descendants' standards. An explicit parent list overrides that inferred union. Metadata may contain multiple comma-separated standard codes. The `\standardstable` summary uses the plain codes, not your visual formatting.

To format the full visible label or emphasize a *particular occurrence* of a code, use:

```latex
\renewcommand{\standardsLabel}[1]{[Standards: #1]}
\RenewDocumentCommand{\markStandard}{m}{*#1*}

\part[\markStandard{ALG2}] Evaluate \(f(3)\).
```

`\markStandard{...}` is a **presentation hook inside metadata**. Its formatting affects the visible label; the inherited standards and summary table still store the plain code (`ALG2` in this example). With the shown preamble setting, the part displays `[*ALG2*]` but the summary shows `ALG2`. The default hook simply prints its argument. Do not confuse the singular `\markStandard` with the plural `\standardsLabel`.

## 4. Student versions and answer keys

Student mode is **the default**. Write `\noprintanswers` to select it explicitly or `\printanswers` to select the answer key. Put these commands *after* `\usepackage{ximeraExam}`. If you issue both in the same scope, the last one takes effect.

In student mode an ordinary `\answer{...}` becomes Ximera's student answer blank, correct multiple-choice options are not marked, and `hint`, `feedback`, and `solution` bodies are hidden. In key mode the answers are printed, correct options are labeled **(correct)**, and explanatory blocks are displayed. An answer marked as *given* by Ximera remains visible in both modes. Use `\answer` **in math mode**, e.g. `\(x=\answer{3}\)`.

```latex
\noprintanswers
\begin{questions}
  \question[2] Student question: \(2+2=\answer{4}\).

  \begingroup
    \printanswers
    \question[2] Worked example: \(3+2=\answer{5}\).
    \begin{hint}Add the two integers.\end{hint}
    \begin{solution}The sum is 5.\end{solution}
  \endgroup

  \question[2] Back to student mode: \(4+2=\answer{6}\).
\end{questions}
```

**Per-question switching is supported** `\begingroup` … `\endgroup` makes the mode change local; it leaves question numbering and metadata intact. The inverse also works: start a key with `\printanswers` and use a grouped `\noprintanswers` around a question to hide that question's answer material. Keep the **entire question body inside its group**, up to—but not including—the next question. The same approach works around `\ximeraQuestion` imports (see §6). These switches control all supported answer material, not just the text of `\answer`.

### Answer and key display hooks

Redefine these commands to adjust the printed appearance:

| Hook | Default purpose |
| --- | --- |
| `\studentanswerformat{...}` | Format student blanks through Ximera's `\handoutAnswerFormat`. |
| `\keyanswerformat{...}` | Format ordinary answers in keys through Ximera's `\answerFormatPlain`. |
| `\givenanswerformat{...}` | Format given answers in either mode. |
| `\correctchoicemark` | Text marker `(correct)` beside a correct key choice. |
| `\ximeraExamKeyBlock{label}{body}` | Format a printed hint or feedback block. |
| `\ximeraExamSolutionBox{body}` | Format a printed solution box. |

`\choice`, `\choiceEXP`, and inline word-choice variants follow the answer mode in PDF output. In answer keys, solutions appear in **breakable bordered boxes**: a long solution can continue onto another page, each segment has a border, and the **Solution:** label appears only at the start. The optional `solution` spacing argument reserves space only in student mode (see §5). Answer-mode overrides are for the **print/PDF build**; Ximera's native interactive HTML answer and choice commands are left alone.

### Printed multiple-choice layout

Ordinary `multipleChoice` questions print without a visible “Multiple Choice” heading and use capital-letter labels `(A)`, `(B)`, `(C)`, … so they do not clash with lowercase part labels. The default layout is vertical:

```latex
\begin{multipleChoice}
  \choice{First option}
  \choice[correct]{Second option}
  \choice{Third option}
\end{multipleChoice}
```

For choices that should run across the page, use `layout=horizontal`:

```latex
\begin{multipleChoice}[layout=horizontal]
  \choice{First option}
  \choice{Second option}
  \choice{Third option}
\end{multipleChoice}
```

A horizontal row is spread across the available line width with equal stretchable space before the first choice, between choices, and after the last choice. To limit how many choices appear on one row, add `max-per-row=<n>`:

```latex
\begin{multipleChoice}[layout=horizontal,max-per-row=3]
  \choice{One}
  \choice{Two}
  \choice{Three}
  \choice{Four}
  \choice{Five}
  \choice{Six}
\end{multipleChoice}
```

With `max-per-row=3`, the example prints as two independently spaced rows of three. Omitting `max-per-row` keeps all horizontal choices on one row when they fit. Correct choices remain unmarked in student mode and receive the configured `\correctchoicemark` in answer keys.


## 5. Answer lines and handwritten response space

### Answer lines: inline versus standalone

`\answerline[options]{label}` uses an **empty label** for an inline writing line and a **nonempty label** for a standalone labeled writing line:

```latex
% Inline: stays on the same line as surrounding text.
Final answer: \answerline{}.

% Standalone: starts its own row with a label and writing line.
\answerline{Final answer:}
```

**Important for answer keys: these forms behave differently.**

| Form | Student version | Answer key |
| --- | --- | --- |
| Inline, `\answerline{}` | Writing line shown | **Writing line always shown**, even with `\answerlineInKey{hide}`. |
| Standalone, `\answerline{Final answer:}` | Label and writing line shown | **Hidden by default**; shown only with `\answerlineInKey{show}`. |

Set the defaults in the preamble, after loading `ximeraExam`:

```latex
\answerlineWidth{2in}      % Default line length (default: 2in).
\answerlineAlign{left}     % Standalone alignment (default: left).
% \answerlineAlign{right}  % Or align standalone labeled lines to the right.
\answerlineInKey{hide}     % Hide standalone lines in keys (default).
% \answerlineInKey{show}   % Or keep standalone lines in keys.
```

`\answerlineWidth` controls the default length of **both** kinds of line. `\answerlineAlign` changes **standalone lines only**; it does not move inline lines. `\answerlineInKey` also controls **standalone lines only**; inline lines always appear in the key. Use the following optional settings to override the width for one line or the alignment for one standalone line:

```latex
\answerline[width=1in]{}
\answerline[width=3in,align=right]{Final answer:}
```

When a standalone label and writing line are too wide to fit together, the line moves below its label. For a short response you wish to print in the answer key, you can use a `solution` environment or ordinary text instead of relying on an inline writing line to disappear.

### Space for longer written work and solutions

Use the optional argument of `solution` to reserve answer space **only in the student version**. The answer key shows the authored solution *instead* of reserving that space:

```latex
\begin{solution}[2in]
  Show the calculation and explain why the answer is correct.
\end{solution}

\begin{solution}[\stretch{1}]
  A longer worked solution goes here.
\end{solution}
```

`[2in]` reserves a fixed height; `[\stretch{1}]` requests flexible vertical space, like `\vspace*{\stretch{1}}`. Stretchable space follows normal LaTeX page-building behavior; it does not guarantee a minimum height or start a new page. Without an optional argument, `solution` reserves no extra student writing space. Its response-space behavior respects local `\printanswers` / `\noprintanswers` switching (see §4).

In the answer key, a **long solution can break across pages**. The border closes at the bottom of a page and resumes on the next page, while “Solution:” appears only at the beginning. Ordinary `\vspace` remains appropriate for space that should appear in **both** the student version and the key.

## 6. Import an existing Ximera activity

`\ximeraQuestion[metadata]{file.tex}` starts **one exam question** and inserts the body of an existing, complete Ximera activity:

```latex
\begin{questions}
  \ximeraQuestion[5]{limitLaw9.tex}
  \ximeraQuestion[8]{relative/path/anotherActivity.tex}
\end{questions}
```

The package skips the imported activity's document class and preamble and treats the activity's outermost problem-like wrapper as the exam question's body. Nested problem-like wrappers are retained. An activity with multiple sibling exercises can still produce several exercises **within a single exam question**, so inspect the rendered result rather than assuming every exercise becomes its own separately numbered exam question.

Put packages, macros, colors, and TikZ styles required by the imported activity in the **parent exam preamble**, before the import; the activity's original preamble is not loaded as normal. File paths must resolve from the compilation context. The exam supplies the points or standards metadata, not the imported activity. For a locally answered import:

```latex
\begingroup
  \printanswers
  \ximeraQuestion[AL1]{limitLaw9.tex}
\endgroup
```

Use a numeric metadata value instead of `AL1` when in points mode.

## 7. Cover page and exam information

`\examtitle{...}`, `\examcourse{...}`, `\examdate{...}`, and `\examversion{...}` store exam information. Their expandable accessors are `\examtitletext`, `\examcoursetext`, `\examdatetext`, and `\examversiontext`; use them in your own cover, header, or footer. The title command also sets the document's PDF title property when the LaTeX PDF-management interface is available.

For a simple default cover, put `\makeexamcover` after `\begin{document}`. It prints the title, optional course/date/version, and a name prompt, then starts a new page. The visible title uses an **H1 heading tag** when tagging is active.

For a custom cover, use `examcover`:

```latex
\begin{examcover}
  \examcoverheading{\examtitletext}
  \noindent \examcoursetext\quad \examversiontext\par
  \bigskip
  Name: \rule{3in}{0.4pt}\par
  \bigskip
  \gradetable[h]
\end{examcover}
```

`examcover` leaves the first page's header/footer empty by default and starts a new page on exit. A custom cover does **not automatically print the title**, so include `\examcoverheading{...}` when you want a tagged H1. You can redefine the default-cover hooks `\examnameprompt`, `\examnamefield`, `\examcoverbeforetitle`, `\examcoveraftertitle`, and `\examcoverheadingformat{...}` without replacing the cover-building commands.

## 8. Grade and standards tables

Use `\gradetable` in points mode and `\standardstable` in standards mode. They read **the preceding LaTeX run's question data** and may appear on the cover, *before* the actual `questions` environment. On a first compile the summary may say it is available after the next run.

```latex
\gradetable                  % Horizontal; Earned row; 4 question columns/block.
\gradetable[v]               % Vertical table.
\gradetable[h][noearned][3]  % Horizontal, omit Earned, 3 question columns/block.

\standardstable              % Horizontal; Earned row; 3 question columns/block.
\standardstable[v][noearned] % Vertical, omit Earned column.
\standardstable[h][noearned][4]
```

The **first** optional argument chooses `h` (horizontal, default) or `v` (vertical). The **second** is `noearned` to omit the earned row/column (default: include it). The **third** is a positive integer maximum number of *question columns per horizontal block*; its defaults are 4 for points and 3 for standards. The third argument has no useful effect on a vertical table. A horizontal table can wrap across multiple blocks. The horizontal points table includes a regular **Total** column and, when bonus points exist, a separate **Bonus** column immediately to its right in the final block. The vertical points table shows the bonus total in a separate row. The regular total excludes bonus points; the standards table lists codes without a numerical total. Do not mix `\gradetable` with standards mode or `\standardstable` with points mode.

## 9. Headers, footers, and page styles

The page-furniture commands take **three mandatory arguments**, in **left, center, right** order:

```latex
\firstpageheader{Left}{Center}{Right}
\firstpagefooter{Left}{Center}{Right}
\runningheader{\examcoursetext}{\examtitletext}{\examversiontext}
\runningfooter{\examdatetext}{}{Page~\thepage\ of~\pageref{LastPage}}
```

The `ximeraExamRunning` style is selected automatically at the beginning of the document. The default running header is course / title / version, and the default running footer is date / blank / `Page X of Y`. The first-page style, `ximeraExamFirst`, starts empty and is selected **explicitly** with `\thispagestyle{ximeraExamFirst}` when desired; it is not automatically applied to the exam cover. The `examcover` environment uses an empty page style.

Use `{}` for any field you want blank. Header/footer content is treated as an **artifact** in tagged PDF, rather than repeated as substantive reading content on every page; put essential instructions or information in the document body as well.

## 10. Worked points-exam pattern

This short pattern combines a custom cover, inherited points, a key-ready answer, and a bonus:

```latex
\DocumentMetadata{lang=en-US,pdfstandard=ua-2,tagging=on}
\documentclass{ximera}
\usepackage{unicode-math}
\usepackage[points]{ximeraExam}
\examtitle{Sample Exam}
\examcourse{MATH 101}
\examversion{Form A}
\noprintanswers

\begin{document}
\begin{examcover}
  \examcoverheading{\examtitletext}
  \noindent Name: \rule{3in}{0.4pt}\par
  \bigskip
  \gradetable[h][noearned][4]
\end{examcover}
\begin{questions}
  \question
  \begin{parts}
    \part[2] Compute \(\lim_{x\to 2}(x+1)=\answer{3}\).
    \part[3] Explain the limit law used.
  \end{parts}

  \bonusquestion[1] Give another example.
\end{questions}
\end{document}
```

For a **standards** example, see the §3 standards question pattern and use `\standardstable` instead of `\gradetable`. Starter templates for points mode and standards mode, plus a Calculus I worked example, are available separately as `.tex` files in the project’s templates collection.

## 11. Command reference

| Public command or environment | Purpose |
| --- | --- |
| `\usepackage[points]{ximeraExam}` / `[standards]` | Select assessment mode. |
| `\ximeraExamMode` | Expand to the current assessment mode name (`points` or `standards`). |
| `questions`; `\question[metadata]`; `\bonusquestion[metadata]` | Define the main question sequence, ordinary questions, and bonus questions; visible top-level question labels are H2 headings in tagged PDF output. |
| `parts` / `\part[metadata]`; `subparts` / `\subpart[metadata]`; `subsubparts` / `\subsubpart[metadata]` | Nest assessment items. |
| `\ximeraQuestion[metadata]{file.tex}` | Import a complete Ximera activity as one exam question. |
| `\printanswers`; `\noprintanswers` | Choose key or student rendering, globally or within a TeX group. |
| `multipleChoice[layout=vertical or horizontal,max-per-row=n]`; `\choice[correct]{...}` | Print capital-letter multiple-choice options; horizontal mode can limit and evenly spread choices by row. |
| `\answerline[width=...,align=...]{label}` | Print an inline line with an empty label, or a standalone line with a nonempty label. |
| `\answerlineWidth{length}`; `\answerlineAlign{left or right}`; `\answerlineInKey{hide or show}` | Set line length, standalone alignment, and standalone visibility in keys. |
| `solution` / `\begin{solution}[space]` | Hide solutions and reserve optional fixed/stretchable response space for students; print a breakable boxed solution in keys. |
| `\pointsLabel{metadata}`; `\standardsLabel{metadata}`; `\markStandard{code}` | Customize printed metadata labels and individually marked standards. |
| `\questionnumber`, `\partnumber`, `\subpartnumber`, `\subsubpartnumber` | Customize numbering presentation. |
| `\questionlabel`, `\bonusquestionlabel`, `\partlabel`, `\subpartlabel`, `\subsubpartlabel` | Customize visible item labels. |
| `\examtitle{...}`, `\examcourse{...}`, `\examdate{...}`, `\examversion{...}` | Store cover/page-furniture information. |
| `\examtitletext`, `\examcoursetext`, `\examdatetext`, `\examversiontext` | Retrieve stored information. |
| `\makeexamcover`; `examcover`; `\examcoverheading{...}` | Produce default/custom cover and tagged cover heading. |
| `\gradetable[h or v][earned or noearned][max]` | Points-mode summary. |
| `\standardstable[h or v][earned or noearned][max]` | Standards-mode summary. |
| `\firstpageheader{L}{C}{R}`, `\firstpagefooter{L}{C}{R}` | Set fields for explicitly selected first-page style. |
| `\runningheader{L}{C}{R}`, `\runningfooter{L}{C}{R}` | Set running-page fields. |
| `\thispagestyle{ximeraExamFirst}` | Explicitly select the first-page layout. |

## 12. Accessibility and current limitations

Start `\DocumentMetadata` **before** `\documentclass`; use LuaLaTeX with `unicode-math`, and validate the completed PDF with veraPDF's `ua2` profile. The package's built-in heading structure uses the exam cover title as H1 and each visible top-level **Question N** / **Bonus Question N** label as H2; nested parts remain list items. The project has successfully compiled and automatically validated a five-page historical exam with a graph, a piecewise function, nested questions, an array/table of values, and a cover grade table. Automated validation is **not equivalent** to checking mathematical reading order, graph descriptions, meaningful table headers, visual contrast, and usability with assistive technology; inspect these separately for each real exam.

Known limits: summary data require another compile; the imported activity's own preamble is skipped; the package does not generate model answers you have not authored; formatting of large diagrams, response spaces, and custom covers remains the exam author's responsibility. Although this package creates an accessible tagged structure for its supported exam features, it cannot guarantee that arbitrary custom LaTeX or TikZ in an imported problem is accessible without author review.


### Imported activity hierarchy and printable word choices (1.7.1)

For `\ximeraQuestion`, the outermost problem-like environment supplies the exam question body. Nested problem-like environments become exam parts and subparts rather than additional “Exercise” headings. In PDF output, `\wordChoice` presents its options as a vertical, lettered multiple-choice list; the short list stays together, and an immediately following period in the activity source is suppressed so it does not appear on a separate line. The correct option receives the existing `(correct)` marker in answer keys. Ximera’s HTML word-choice behavior is unchanged; given-word mode continues to use the original rendering.
