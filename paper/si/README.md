# Standalone supplement build

Three outputs from one source. Nothing is duplicated between them.

| command | output | what it is |
|---|---|---|
| `quarto render paper/standard_model_short.qmd --to pdf` | `standard_model_short.pdf` | main text with the SI folded in, for internal review |
| `quarto render paper/standard_model_short.qmd --to docx` | `standard_model_short.docx` | main text alone, for portals that want a Word file |
| `quarto render paper/si/supplement.qmd` | `paper/si/supplement.pdf` | **supplementary information as a separate file**, S-numbered |

Journals take the supplement as its own file, so `supplement.qmd` builds the SI
standalone from the same source the review PDF folds in -- `paper/supplemental.qmd` --
and there is one copy of the SI, not two that can drift apart.

The SI stays **PDF** rather than Word, and that is what makes this cheap: it carries
thirteen kableExtra tables using `pack_rows`, `add_header_above` and `landscape`, none
of which survive a Word conversion. The main text carries no tables at all, so it
converts with nothing lost.

## Things that will bite if they are changed

- **`dev` must follow the format.** The setup chunk sets `cairo_pdf` for LaTeX and
  `ragg_png` otherwise. A docx built with `cairo_pdf` embeds `.pdf` figures that Word
  renders as blank placeholders -- the render succeeds and the figures are simply missing.
- **The SI include is wrapped in `.content-visible when-format="pdf"`.** Removing that
  puts the whole supplement inside the submission docx.
- **The S in `fig. S1` lives in the LaTeX counter,** not in `fig-title`. Quarto joins
  label and number with a hardcoded space, and for PDF that join happens inside LaTeX's
  caption machinery where a Lua filter on the Pandoc AST cannot reach it. Hence
  `\renewcommand{\thefigure}{S\arabic{figure}}` in `supplement.qmd`.
- **`paper/_setup_shared.R` is sourced by both documents.** It was extracted from the
  main .qmd's setup chunk because the SI quotes values (`n_excl`, `pct_excl`, `qc_mar`,
  `qc_nor`) that used to exist only inside that chunk -- which is why rendering
  `supplemental.qmd` on its own always failed with `object 'n_excl' not found`. Do not
  duplicate it back into either file.
- **Stale render artifacts do not announce themselves.** The built PDFs are only as
  current as their last render; re-render after any source change before submitting
  anything. This nearly shipped broken files twice.
