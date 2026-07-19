# Exploring Homes in CA, NJ, NY, and PA -- Statistical Analysis

![CI](https://github.com/dagron27/stat353-homes-price-analysis/actions/workflows/ci.yml/badge.svg)

**Course:** `STAT 353-54, Statistical Methods I for Engineering, Fall 2023`

**Assignment:** Project #3 -- Exploring Homes in CA, NJ, NY, and PA

Note on the course term: the assignment instructions (a D2L export) are
timestamped `11/28/23` and headed "Fa23 STAT 353-54"; the R Markdown
report's own front matter also self-dates `2023-11-29`. Both independently
point to **Fall 2023**, not Fall 2024.

## Assignment Intent

Using the "HomesForSale" dataset (Lock5 Statistics, 3rd edition -- 120
homes for sale across California, New Jersey, New York, and Pennsylvania
in 2019), answer five questions: (1) does home size affect price in
California; (2) does bedroom count affect price in California; (3) does
bathroom count affect price in California; (4) how do size, bedrooms, and
bathrooms jointly affect price in California; (5) do home prices differ
significantly across all four states. Grading weighted Analysis heaviest
(40%), then Accuracy and Writing (20% each), Structure (15%), and Sources
(5%).

**Confirmed implemented**, per `notebooks/Stat353Proj3v2.Rmd` (see Overview below):
all five questions are answered, each with a stated null/alternative
hypothesis, an R model (`lm()` for questions 1-4, `aov()` for question 5),
a plot, a written interpretation of the model output, and a conclusion
that explicitly accepts or rejects the null hypothesis. Question 5 goes
beyond a bare ANOVA: it includes Tukey HSD pairwise post-hoc comparisons
and residual diagnostics (a residuals-vs-fitted plot, a histogram of
residuals with a normal curve overlaid, a Q-Q plot, and a Shapiro-Wilk
normality test) -- checking the ANOVA's own assumptions rather than
taking a significant p-value at face value. The report closes with a
Summary section (key takeaways, suggested future investigations, and an
overall conclusion) and a Resources section citing the data source.

## Overview

`notebooks/Stat353Proj3v2.Rmd` is a single R Markdown report (knits to both Word and
PDF; `docs/Report.pdf` is the committed rendered output) structured as
Background / Task / Data / Analysis (one subsection per question) /
Summary / Resources. Findings, verified against the report's own R output:

1. **Size vs. Price (CA):** significant (p = 0.0005). R-squared 35.94%.
2. **Bedrooms vs. Price (CA):** not significant (p = 0.255).
3. **Bathrooms vs. Price (CA):** significant (p = 0.0041). R-squared 25.88%.
4. **Size + Bedrooms + Bathrooms jointly (CA):** the overall model is
   significant (F = 5.568, p = 0.0044, R-squared 39.12%), but only the
   Size coefficient is individually significant within that model (p =
   0.0259) -- Bedrooms and Bathrooms are not, once the other two
   predictors are controlled for.
5. **Price differences across all four states (ANOVA):** significant (p =
   0.000148). Tukey HSD shows California is significantly different from
   all three other states; New Jersey and Pennsylvania are not
   significantly different from each other. The report also honestly
   flags that the ANOVA's own assumptions are questionable here:
   heteroscedasticity is visible in the residual plot, and the
   Shapiro-Wilk test rejects normality of the residuals -- so the
   conclusion notes this as a caveat rather than treating the significant
   p-value as the final word.

## Dependencies

R packages used (declared via `library()` in the Rmd's setup chunk):

- `ggplot2`
- `dplyr`
- `psych`

Base R itself provides everything else used (`lm`, `aov`, `plot`,
`boxplot`, `TukeyHSD`, `shapiro.test`, `qqnorm`, `hist`, `curve`).
Knitting to Word or PDF output additionally requires `rmarkdown` and, for
the PDF target specifically, a LaTeX distribution (e.g. `tinytex`).

## Environment Setup

```r
install.packages(c("rmarkdown", "ggplot2", "dplyr", "psych"))
# Only needed if knitting to PDF and no LaTeX distribution is installed:
install.packages("tinytex")
tinytex::install_tinytex()
```

Knit the report:

```r
rmarkdown::render("notebooks/Stat353Proj3v2.Rmd", output_format = "pdf_document")
```

The report fetches its data live over HTTPS from
`https://www.lock5stat.com/datasets3e/HomesForSale.csv` at knit time --
see Known Issues below -- so an internet connection is required to
reproduce it, and the local `data/HomesForSale*.csv`/`.xlsx` files in this repo
are not read by the report at all.

## Continuous Integration

A GitHub Actions workflow (`.github/workflows/ci.yml`) runs on every push
and can also be triggered manually via `workflow_dispatch`. It sets up R,
installs the packages listed above, and knits `notebooks/Stat353Proj3v2.Rmd` to PDF
end to end -- a real functional run (fetching the live dataset and
re-fitting every model), not a syntax-only check. The rendered PDF is
uploaded as a downloadable build artifact and is never committed back to
the repository.

Because the report fetches data from an external site
(`lock5stat.com`) at render time, this CI job -- and reproducing the
report locally -- depends on that site remaining available and the
dataset at that URL remaining unchanged. If `lock5stat.com` is ever
unreachable or the CSV's structure changes, the knit will fail for
reasons unrelated to this repository's own code.

## Known Issues

### Dead Code / Hygiene

- **The committed local dataset files are not used by the report.**
  `data/HomesForSale.csv`, `data/HomesForSale.xlsx`, `data/HomesForSaleCA.csv`,
  `data/HomesForSaleCanton.csv`, and `data/HomesForSaleNY.csv` are all present in
  this repository, but `notebooks/Stat353Proj3v2.Rmd`'s only data-loading call is
  `read.csv("https://www.lock5stat.com/datasets3e/HomesForSale.csv")` --
  it fetches fresh data from the web rather than reading any of these
  local files. They were most likely downloaded for manual inspection of
  the dataset (column names, per-state subsets) while developing the
  analysis, then left in the repository. All five files are small (2KB-
  13KB) and are kept here as a record of that exploration rather than
  deleted, since they don't cost anything to keep and may be useful
  reference for the data's shape without needing network access.
- **Reproducibility depends on an external site staying available and
  unchanged.** See Continuous Integration above.

### CI Failure: Unescaped `$` Broke LaTeX Compilation -- Fixed

CI failed with `Error: LaTeX failed to compile ... Stat353Proj3v2.tex` --
the knit step (Rmd to `.tex`) succeeded, but the LaTeX-to-PDF compilation
of that generated file failed. The `.Rmd`'s "Elements" section contains
the line `Price Asking price (in $1,000's)`, in plain prose text, not
inside an R code chunk. R Markdown's underlying pandoc conversion treats
`$...$` as inline math by default (the `tex_math_dollars` extension);
this was the only unescaped `$` anywhere in this document's prose, so
pandoc found no matching closing `$` and this stray math-mode delimiter
was the cause of the malformed LaTeX in the generated `.tex` file.
Escaped it to `\$1,000's` (a literal dollar sign, not math-mode).

### CI Failure: Literal Unicode Characters Unsupported by `pdflatex` -- Fixed

After the `$` fix above, the next CI run progressed further (through the
knit step and partway through LaTeX compilation) but failed with a new
error: `! LaTeX Error: Unicode character β (U+03B2) not set up for use
with LaTeX.` The default PDF engine (`pdflatex`, an 8-bit engine) can't
render arbitrary Unicode bytes that pass through pandoc unconverted. The
document's five hypothesis-statement sections used literal Unicode `β`
and `≠` characters (e.g. `H0:βSize=0`, `H1:βSize≠0`) instead of LaTeX
math syntax; these were rewritten as proper inline math (`H0:
$\beta_{Size} = 0$`, `H1: $\beta_{Size} \neq 0$`, and equivalently for
Beds, Baths, the joint-regression hypothesis, and the states-ANOVA
hypothesis). A full scan of the document then also turned up several
other non-ASCII typographic characters in prose that could have caused
the same class of failure on a later run -- em dashes (`—`), curly
apostrophes (`'`), a Unicode minus sign (`−`), and multiplication signs
(`×`) -- all replaced with LaTeX/pandoc-safe ASCII equivalents (`---`
markdown em-dash syntax, straight apostrophes, hyphens, and `x`
respectively).

**Verification**: `pdflatex`/`xelatex` (TeXLive) were available locally,
so the exact math syntax used in the fix (`$\beta_{Size} = 0$`,
`\neq`, subscripted Greek letters) was compile-tested in a standalone
`.tex` file and the resulting PDF visually confirmed to render correctly
before being applied to the real document -- a stronger verification
than the first `$` fix above, which could only be reasoned about from
the error text since no LaTeX toolchain was available at the time.

### Security

No findings. This is a local statistical analysis script with one
outbound HTTPS request to a fixed, hardcoded, well-known academic dataset
URL (no user input, no credentials) and no other file, network, or
subprocess activity.

## Status

Complete. All five assigned questions are answered with a stated
hypothesis, model, plot, interpretation, and conclusion; the state-level
ANOVA additionally includes post-hoc comparisons and a full set of
residual diagnostics that honestly report where the model's assumptions
are questionable (heteroscedasticity, non-normal residuals) rather than
omitting that discussion. `docs/Report.pdf` is the rendered output of
the current `.Rmd`.
