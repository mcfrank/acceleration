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

## Building

```bash
quarto render paper/standard_model_short.qmd --to docx   # main text, for submission
quarto render paper/si/supplement.qmd                    # supplement, for submission
quarto render paper/standard_model_short.qmd --to pdf    # both together, for review
```

The manuscript renders from committed caches in `paper/cache/`, so none of this needs
cluster access or a fit download. To rebuild those caches from the fits, see
[`journal/ACCELERATION_PROVENANCE.md`](journal/ACCELERATION_PROVENANCE.md), which maps
every claim in the paper and supplement to the cache, script, and fits behind it — and
records the traps (one script needs a live Wordbank connection; another silently truncates
if run without arguments).

## Layout

| path | what |
|---|---|
| `paper/` | manuscript, supplement, cache-building scripts, submission targets |
| `studies/bayes_long/` | the M0-M3 model ladder, 2PL variant, forward cross-validation |
| `studies/llm/` | language-model learning-curve analysis |
| `studies/gamlss/` | non-parametric quantile comparison |
| `cluster/` | Sherlock (Stan) and ccn2 / Marlowe (GPU) job recipes |
| `journal/` | experiment logs, provenance, orientation for a new session |
| `fits/` | model fits — gitignored; see below |

## Fits

Fits are not in git. They are archived publicly on Redivis:

**<https://redivis.com/datasets/datapages.acceleration:a1c7>** (v1.0)

- `stan_fits` — bundles, posterior summaries, draws, diagnostics, per-child and per-item exports
- `lm_ladders` — per-word language-model learning curves
- `loo_objects` — pointwise LOO log-likelihoods (needed only to re-derive the LOO comparisons)

The first two are ~55 MB and enough to rebuild every cache except the LOO tables.

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
