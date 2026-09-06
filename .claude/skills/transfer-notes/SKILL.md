---
name: transfer-notes
description: Port Nikhil's handwritten annotations (blue notes + yellow highlights) from the annotated v1 PDF into the v3 LaTeX source of "The Principles of Diffusion Models", one chapter at a time. Use when asked to port/transfer notes or highlights for a chapter, or to compare a chapter's PDF annotations against the tex.
---

# transfer-notes

Port annotations from `principles_gen_models/v1_annotated.pdf` (arXiv 2510.21890 **v1**) into
`principles_gen_models/arXiv-2510.21890v3/` (**v3** tex). Argument: a chapter/appendix name.
Be concise in every report: tables and one-liners, no narration.

## Ground rules
- **v3 text always wins.** Authors' v1→v3 rewording is never a conflict and is never reported.
- **Conflict = Nikhil's annotation targets text that v3 changed or removed.** Report these (with the v3 analog sentence if one exists, or "drop") and wait for his decision. Never resolve silently.
- Two phases: **report first, edit only after go-ahead.**
- Port highlights too, not just handwritten notes. Don't highlight Nikhil's own notes.
- Lightly fix slips in his notes (variable names, swapped labels) and mention each fix in one line.

## Style rules for ported notes (in priority order)
1. **Expand equalities in place.** When a note justifies a step in an existing derivation, insert the intermediate equalities directly into the original display (blue lines/members inside the authors' align) — never a separate "Why? ..." block after it.
2. **No parentheses around notes.** Write inline notes as regular sentences, not "(...)" asides, unless a parenthetical is genuinely the natural form (e.g., a one-word gloss).
3. **Formal explanations get a \textit{Proof.} block**, ending with $\blacksquare$ — never a "Why?" label. Standalone blocks are only for self-contained lemmas/proofs the text doesn't already contain.
4. **Minimize words in derivations.** Prefer chains of math; add prose only where a step is genuinely confusing. A trailing half-sentence naming the facts used beats interleaved narration.
5. **Never abridge his derivations.** Every step from the handwritten chain goes in, in his order (only slips corrected, each mentioned in one line of the report).

- Done: Ch.2 (Variational: VAEs→DDPMs), Ch.3 (Score-Based: EBMs→NCSN), Appendix B, Appendix C, D.1/D.2.1 (earlier session), D.2.3, D.2.4, D.2.5. Pattern for the rest is identical.
- Pitfall: curly quotes/apostrophes (’) inside `\nkh{...}` break soul under inputenc — use ASCII ' inside highlights. Deterministic-analog / divergence-theorem notes now live in Appendix E — skip them when they appear in the margins.

## Macros (defined in `math_commands.tex`)
| Macro | Use | Pitfall |
|---|---|---|
| `\nk{...}` | inline blue note (full sentences, per style rule 2) | no leading space inside (`\color` eats it): write `word \nk{Note ...}` |
| `\begin{nknote}...\end{nknote}` | blue block: paragraphs, displays, tikz | fine inside `\exm{}{}` bodies; **not** directly after a run-in `\paragraph{...}` (its `\par` pushes the heading into the margin) — use `\begingroup\color{notesblue} ... \endgroup` there |
| `\nkh{...}` | yellow text highlight (`soul \hl`, breaks across lines) | no `\Cref`/footnotes inside; `$...$` is fine |
| `\nkhm{...}` | yellow box around a piece of display math | math mode only |
- Book macros: `\rvx \rvw \rvz \rmG \rmI_D \Tr \diff \E`; **no `\rvW`** (use `\mathbf{W}`).

## Workflow
1. **Locate.** Tex file: `grep -rn "\\chapter{<title>" *.tex`. PDF pages: scan headers
   `pdftotext -f N -l N v1_annotated.pdf -` over a range (v1 PDF index ≈ printed page + 9).
   Read PDF pages with the Read tool (`pages`, ≤20 per call) — annotations are handwritten, so look at the images.
2. **Report** (no edits): (a) list of notes by page, (b) list of highlighted phrases, (c) conflicts table
   (annotation → what v3 did → recommended resolution), (d) any slips you'd fix. Stop and wait.
3. **Apply** with a Python script of assert-anchored string replacements (`assert s.count(old)==1`), after
   copying the file to the scratchpad. Never blind-edit. Sketches → small inline tikz (`color=notesblue`).
4. **Build & verify** from the v3 dir:
   `export PATH=/Library/TeX/texbin:$PATH; latexmk -pdf -interaction=nonstopmode -file-line-error -outdir=build main_v2.tex`
   Check the log **with Python, not `grep -c`** (grep -c prints blank in this shell): lines starting `!` or matching
   `\.tex:\d+: ` = errors; `Overfull` lines between this file's marker and the next file's = layout to fix
   (split long `align*` lines; move inline `\det(\cdot)` into a display). Then Read the changed pages of
   `build/main_v2.pdf` (locate them by `pdftotext` scan) and eyeball blue/yellow placement and margins.
5. **Report**: one table "annotation → where it landed (file line / PDF page)", build stats, fixes made.

## Reference: how Appendix B/C were done
- Notes ported as `nknote` "Why?"/"Proof"/"Lemma" blocks right after the equation they justify; labels like "A."/"B." in the margin become such blocks.
- Nikhil removed §B.2.1 (physical box argument) entirely; B.2.2 kept. Read removal requests at the subsection level carefully.
- Preferred proof style: e.g. Itô product rule via stacked `z=(x;y)`, Hessian `[[0,I],[I,0]]`, block quadratic covariation — parameterization-free, no diffusion matrix `G`.
- Sign/measure conventions in Girsanov: reference measure = denominator; Brownian motion is the one under the reference measure.
