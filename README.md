# Accelerating returns in word learning

Code, model fits, and manuscript for **"Children, but not language models, show
accelerating returns in word learning."**

Children's vocabulary growth is modelled as an accumulator in which latent ability rises
with the log of accumulated input, and a word is produced once ability exceeds that word's
difficulty:

> theta_i(t) = xi_i + kappa_i * log(t/a0) + log H,  P(produce j) = logit^-1(theta_i(t) - delta_j)

kappa = 1 is the pure accumulator, in which learning is proportional to input and nothing
about the learner changes. Across five longitudinal CDI samples in three languages children
come out near 11-13. GPT-2 models trained on CHILDES at increasing data budgets come out
near 1.2.

This is analysis code for a paper rather than a software package, so the sections
below map the usual code-availability checklist onto it: the **demo** builds the
manuscript from committed intermediate results, **instructions for use** fit the model
to a new longitudinal vocabulary dataset, and **reproducing the paper** walks back from
the manuscript to the public raw data.

## 1. System requirements

What you need depends on how far back you want to go:

| task | software | hardware |
|---|---|---|
| Build the manuscript (the demo) | R, Quarto, TinyTeX, 18 R packages | any desktop |
| Fit the model to a dataset | + CmdStan, `cmdstanr`, `posterior`, `loo` | a desktop for small datasets; a multi-core cluster node for the paper's full samples |
| Rebuild the model inputs from Wordbank | + `wordbankr`, `redivis`, a free Redivis account | any desktop |
| Retrain the language models | Python, PyTorch, Hugging Face `transformers` | NVIDIA GPUs |

### Versions tested

Manuscript build and local model fitting were tested on **macOS 26.6.2** (Apple M2 Max,
32 GB RAM) with:

- R 4.5.1; Quarto 1.8.25 (with its bundled Pandoc 3.6.3); TinyTeX (TeX Live 2026)
- **XQuartz — required on macOS.** R's `cairo_pdf` device, which draws every figure in
  the PDF, links against XQuartz's libraries on macOS.
- R packages: broom 1.0.10, cowplot 1.2.0, dplyr 1.2.1, forcats 1.0.0, ggplot2 4.0.2,
  ggrepel 0.9.6, gridExtra 2.3, here 1.0.2, kableExtra 1.4.0, knitr 1.50,
  latex2exp 0.9.8, patchwork 1.3.2, ragg 1.5.0, readr 2.1.5, rmarkdown 2.29,
  scales 1.4.0, stringr 1.6.0, tidyr 1.3.2
- Model fitting: CmdStan 2.38.0, cmdstanr 0.9.0, posterior 1.6.1, loo 2.8.0
- Data preparation: wordbankr 2.0.0 (GitHub `langcog/wordbankr`, commit `da15c48`),
  redivis 0.12.12 (GitHub `redivis/redivis-r`, commit `3e06033`)
- Used by individual analysis scripts: gamlss 5.5.0, lme4 1.1.37, matrixStats 1.5.0

The fits reported in the paper were run on Stanford's Sherlock cluster (Linux, Slurm)
with CmdStan 2.38.0, as recorded in the sampler diagnostics saved with the later fits. The language
models were trained on Linux with Python 3.12, PyTorch 2.4 (CUDA 12.4 builds),
`transformers` >=4.40,<4.50, `datasets` >=2.20,<4, `accelerate` >=0.30,<2, `tokenizers`
>=0.15,<0.22, `numpy` >=2.0,<2.3 and `pandas` >=2.2,<3 — the ranges pinned in
`cluster/marlowe/stage_marlowe.sh`. Exact resolved versions on the clusters were not
recorded. The manuscript build has not been tested on Windows or Linux.

### Non-standard hardware

None for the demo, or for fitting small datasets. The paper's full samples are another
matter: the largest (Norwegian, 3.2 million observations) took about 40 hours for the
full model on 16 cores, and `studies/bayes_long/fit.slurm` requests 32 cores, 64 GB and
two days per fit. Retraining a language model needs an NVIDIA GPU: one GPT-2-small run used about
31 GB of GPU memory at our batch size. The models were trained on A40 (48 GB), H100 and L40S GPUs; a full-corpus run takes
7–8 hours on an L40S, and the 55-run developmental ladder took about 38 GPU-hours on A40s.

## 2. Installation guide

1. Install [R](https://cran.r-project.org) (4.5 or later) and [Quarto](https://quarto.org),
   and on macOS [XQuartz](https://www.xquartz.org).
2. Install TinyTeX with `quarto install tinytex`.
3. Clone this repository and install the R packages:

```bash
git clone https://github.com/mcfrank/acceleration.git
cd acceleration
Rscript -e 'install.packages(c("broom","cowplot","dplyr","forcats","ggplot2","ggrepel","gridExtra","here","kableExtra","knitr","latex2exp","patchwork","ragg","readr","rmarkdown","scales","stringr","tidyr"), repos = "https://cloud.r-project.org")'
```

That list is complete for the demo: it was tested by installing it into an empty R
library and building all three documents from a fresh clone.

**Install time.** The 18 packages and their dependencies — 78 in all — installed in
26 seconds as CRAN binaries on the test machine. On Linux, CRAN supplies source packages,
which must be compiled and take considerably longer. Quarto and TinyTeX are single
installers and were not timed.

To fit models, and to rebuild the model inputs from Wordbank, you also need:

```r
install.packages("cmdstanr", repos = c("https://stan-dev.r-universe.dev", getOption("repos")))
install.packages(c("posterior", "loo", "remotes"))
cmdstanr::install_cmdstan(version = "2.38.0")   # compiles CmdStan; 45 s on 8 cores here
remotes::install_github(c("langcog/wordbankr@da15c48", "redivis/redivis-r@3e06033"))
```

On macOS, compiling CmdStan needs Apple's command-line tools (`xcode-select --install`).

## 3. Demo

```bash
quarto render paper/standard_model_short.qmd --to pdf    # main text + supplement, for review
quarto render paper/standard_model_short.qmd --to docx   # main text only
quarto render paper/si/supplement.qmd                    # supplement only
```

These build from the intermediate results committed in `paper/cache/`, so they need no
model fits, no data download and no cluster.

**Expected output.**

- `paper/standard_model_short.pdf` — 30 pages: the main text followed by the supplement
- `paper/standard_model_short.docx` — the main text, with figures embedded
- `paper/si/supplement.pdf` — the 19-page standalone supplement

Word labels in the figures are placed by `ggrepel`, whose positions vary slightly from
one build to the next; everything else is identical.

**Run time.** Measured on the test machine, building from a fresh clone: 72 seconds for
the PDF with a freshly installed package library (15 seconds with an established one),
then 12 seconds for the docx and 10 seconds for the supplement.

## 4. Instructions for use

### Fitting the model to your own data

The model needs longitudinal vocabulary checklist data: one row per child ×
administration × word.

| column | meaning |
|---|---|
| `child` | child identifier, the same across that child's administrations |
| `age` | age in months at the administration |
| `item` | the word (checklist item) |
| `produced` | 1 if the child produces the word, else 0 |

Separating efficiency from acceleration needs repeated measurements: at least two
administrations per child, and three or more for per-child acceleration (see the
supplement). From the repository root, convert the table to the bundle the fitting
script reads:

```r
library(dplyr)
d <- read.csv("my_cdi_data.csv")        # columns: child, age (months), item, produced (0/1)

d <- d |> mutate(ii = match(child, unique(child)), jj = match(item, unique(item)))
admins <- d |> distinct(ii, age) |> arrange(ii, age) |> mutate(aa = row_number())
d <- d |> left_join(admins, by = c("ii", "age"))

bundle <- list(
  stan_data = list(N = nrow(d), A = nrow(admins), I = max(d$ii), J = max(d$jj),
                   aa = d$aa, jj = d$jj, y = as.integer(d$produced),
                   admin_to_child = admins$ii, admin_age = admins$age,
                   log_H = log(365), a0 = 18),
  child_ix = distinct(d, ckey = child, ii),
  item_ix  = distinct(d, item, jj))
saveRDS(bundle, "fits/bayes_long/bundle_mydata.rds")
```

Then fit:

```bash
Rscript studies/bayes_long/01_fit.R mydata m3
```

The second argument picks the rung of the model ladder: `m0` (κ fixed at 1, the pure
accumulator), `m1` (population κ), `m2` (+ per-child efficiency) or `m3` (+ per-child
acceleration, the paper's main model). `m32pl` and `m3lin` are the 2PL and linear-age
variants from the supplement. Sampler settings come from the environment: `STAN_CHAINS`,
`STAN_WARMUP` and `STAN_ITER` (defaults 4, 1000 and 1000) and `STAN_THREADS` (threads per
chain, default 4).

Output goes to `fits/bayes_long/summaries/`:

- `mydata_m3.summary.rds` — posterior summary of every parameter; `kappa_pop` is
  population acceleration, κ
- `mydata_m3_child.csv` — per-child efficiency (`xi`) and acceleration (`kappa`):
  medians and 90% intervals
- `mydata_m3_psi.csv` — per-word difficulty (`delta_j`) with its interval and
  convergence diagnostics
- `mydata_m3.draws.rds`, `.loo.rds`, `.diag.rds` — draws of the scalar parameters,
  PSIS-LOO, and sampler diagnostics

**Worked example.** Thirty children from the Japanese sample (90 administrations, 447
words, 40,010 rows), fitted with `STAN_CHAINS=2 STAN_WARMUP=200 STAN_ITER=200
STAN_THREADS=2`, ran in 71 seconds on the test machine including model compilation,
with no divergent transitions. It estimated κ at 12.9 (90% interval 11.9–13.9), against
11.6 from the full sample. The paper's full samples need a cluster (see
[Non-standard hardware](#non-standard-hardware)).

### Reproducing the paper

Each step regenerates what the step above it starts from, working back to public sources.

1. **The manuscript, from committed intermediate results** — the demo above.
2. **The intermediate results, from the model fits.** The fits are archived publicly on
   Redivis at **<https://redivis.com/datasets/datapages.acceleration:a1c7>** (v1.0), in
   three tables: `stan_fits` (Stan data bundles, posterior summaries, draws, diagnostics,
   per-child and per-word exports), `lm_ladders` (per-word language-model learning
   curves) and `loo_objects` (pointwise LOO, about 1 GB, needed only for the LOO
   tables). This fetches the first two, 58 MB, into the layout the scripts expect:

   ```r
   library(redivis)
   ds  <- redivis$organization("datapages")$dataset("acceleration:a1c7", version = "v1.0")
   out <- "fits/bayes_long/summaries"
   ds$table("stan_fits")$download_files(path = out)
   bundles <- list.files(out, "^bundle_", full.names = TRUE)
   file.rename(bundles, file.path("fits/bayes_long", basename(bundles)))
   ds$table("lm_ladders")$download_files(path = "fits/llm")
   ```

   The Redivis client asks you to log in on first use (a free account is enough);
   non-interactive scripts can set `REDIVIS_API_TOKEN` instead. The cache-building
   scripts, the order to run them in and their traps are listed claim by claim in
   [`journal/ACCELERATION_PROVENANCE.md`](journal/ACCELERATION_PROVENANCE.md). One
   input is not yet in the archive: `paper/build_cache_short.R` also reads the CHILDES
   word frequencies behind Fig. 2 (`fits/english_word_freq.rds`).
3. **The fits, from the model inputs.** On a Slurm cluster, `studies/bayes_long/fit.slurm`
   runs one dataset × model; `fcv.slurm` and `pool.slurm` run the forward
   cross-validation and the pooled fit, and `cluster/sherlock/setup_R.R` sets up R and
   CmdStan on the cluster.
4. **The model inputs, from Wordbank.** `MIN_ADMINS=3 Rscript
   studies/bayes_long/00_prepare_bundles.R` rebuilds the five main-analysis bundles from
   Wordbank (drop `MIN_ADMINS=3` for the two-administration bundles used in the
   robustness analyses). `wordbankr` reads Wordbank's Redivis release, so it needs the
   same login. Tested: all five bundles rebuilt byte-for-byte identical to the archived
   ones, in 160 seconds.
5. **The language models.** `studies/llm/train_gpt2_childes.py`, launched with the
   recipes in `cluster/ccn2/` (an 8×A40 workstation) and `cluster/marlowe/` (Slurm).
   The CHILDES corpus and tokenizer come from the public
   [`styfeng/TinyDialogues`](https://github.com/styfeng/TinyDialogues) repository. The
   trained models are also available at
   <https://huggingface.co/mcxfrank/childes-gpt2-ladder> and
   <https://huggingface.co/mcxfrank/gpt2-composition-control>.

## Layout

| path | what |
|---|---|
| `paper/` | manuscript, supplement, cache-building scripts, submission targets |
| `studies/bayes_long/` | the M0-M3 model ladder, 2PL variant, forward cross-validation |
| `studies/llm/` | language-model training and learning-curve analysis |
| `studies/gamlss/` | non-parametric quantile comparison |
| `cluster/` | Sherlock (Stan) and ccn2 / Marlowe (GPU) job recipes |
| `journal/` | experiment logs, provenance, orientation for a new session |
| `fits/` | model fits — gitignored; download from Redivis (above) |

## History

This repository was split out of
[langcog/standard-model-2](https://github.com/langcog/standard-model-2) at commit
`8584c00`, and begins there. Commit history predating the split — including the model
family's development, the identifiability work, and the language-model experiments — lives
in that repository, which remains public. `git log` there for anything earlier than this
repo's first commit.

For the reasoning rather than the diffs, start with
[`journal/STANDARD_MODEL_CONTEXT.md`](journal/STANDARD_MODEL_CONTEXT.md) and the two
experiment logs beside it; they carry what `git blame` would otherwise be used to find.

Split commit: `8584c00b3e2dba049b9c79734a354e9f78b8a039`
