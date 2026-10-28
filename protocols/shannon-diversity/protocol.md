---
name: "shannon-diversity"
description: "Calculate the Shannon diversity index for each sample in a feature-by-sample abundance table."
version: "1.0.0"
authors:
  - name: "Otto Infield-Harm"
    orcid: "0009-0009-8629-6124"
  - name: "Levi Waldron"
    orcid: "0000-0003-2725-0694"
reviews:
  - name: "Levi Waldron"
    orcid: "0000-0003-2725-0694"
    date: "2026-10-27"
    protocol_version: "1.0.0"
    status: approved
date: "2026-10-26"
status: draft
type: "atomic"

artifact_doi: ~
collection_doi: "10.5281/zenodo.22731694"
protocol_citation: "10.1002/j.1538-7305.1948.tb01338.x"
method_origin_citation: "10.1002/j.1538-7305.1948.tb01338.x"
license: "CC-BY-4.0"
protocols_used: []
upstream_repositories:
  - "https://github.com/vegandevs/vegan"
  - "https://github.com/scikit-bio/scikit-bio"
database_urls: []
key_packages:
  - "vegan"
  - "scikit-bio"
category: "Statistical Analysis"
tags:
  - alpha-diversity
  - shannon
  - microbiome
---

# Shannon diversity of a single sample

Calculate one Shannon diversity value for each sample in a feature-by-sample abundance
table. The calculation is descriptive: this protocol does not rarefy, subsample, or
otherwise normalize samples to a common sequencing depth, and it does not compare groups,
test hypotheses, or interpret the resulting values.

## Materials

- **Input data:**
  - `data` — a table with one row per feature and one column per sample. The first
    column must contain unique, non-missing feature identifiers, and there must be at
    least one sample column. Every other cell must be a non-negative number, and no cell
    may be missing: a missing value must not be imputed, converted to zero, or omitted.
  - When `input_scale` is `proportions`, each sample column must sum to 1 within an
    absolute tolerance of $10^{-6}$. A sample outside that tolerance is not renormalized.
  - When `input_scale` is `counts`, samples should be rarefied or otherwise normalized to
    a common sequencing depth, so that every sample column has the same total. If the
    totals differ, proceed, and warn that the Shannon values depend on sequencing depth
    and are not comparable across samples, and advise rarefying to a common depth before
    comparing them. Proportions carry no depth, so this condition cannot be checked on
    them.
- **Parameters:**
  - `input_scale` — whether `data` holds `counts` or `proportions`. Required, with no
    default, because the scale changes Step 1 and cannot be told reliably from the values.
    It must be one of those two words.
  - `log_base` — the base $b$ of the logarithm in Step 2. The default is `e`, the natural
    logarithm, which gives the index in nats; `2` gives bits. It must be `e` or a number
    greater than 1.
- **Software & Repositories:**
  - `vegan` (https://github.com/vegandevs/vegan) — R package; `diversity()` computes
    Step 2.
  - `scikit-bio` (https://github.com/scikit-bio/scikit-bio) — Python package;
    `skbio.diversity.alpha.shannon()` computes Step 2.
  - Either is optional. Any implementation that follows Steps 1 and 2 is acceptable.

## Steps

### Step 1: Convert abundances to proportions

For each sample independently, let $x_{ij}$ be the abundance of feature $i$ in
sample $j$.

For `counts`, calculate

$$
p_{ij} = \frac{x_{ij}}{\sum_{i} x_{ij}}.
$$

If a count sample's total is zero, its proportions are undefined: stop, report which
sample it is, and produce no output. For `proportions`, use the values as given:
$p_{ij} = x_{ij}$.

Zero-valued features remain part of the table but make no contribution to the index.

### Step 2: Calculate Shannon diversity

For each sample $j$, calculate

$$
H_j = -\sum_{i:p_{ij}>0} p_{ij}\log_b(p_{ij}),
$$

where $b$ is `log_base`. Terms with $p_{ij}=0$ are omitted, because their limiting
contribution is zero.

The same calculation, for a feature-by-sample matrix `x` with feature identifiers as row
names:

- R: `vegan::diversity(x, index = "shannon", MARGIN = 2, base = exp(1))`. `MARGIN = 2`
  treats columns as samples; the function's default, `MARGIN = 1`, treats rows as samples.
- Python: `skbio.diversity.alpha.shannon(x[:, j], base=math.e)` for each sample column `j`.

Replace `exp(1)` or `math.e` with $b$ when `log_base` is not `e`.

### Step 3: Return the result

Create a table with exactly two columns, `sample_id` and `shannon`, and one row
for each input sample. Preserve the input sample-column order. Do not add feature
summaries, group labels, confidence intervals, tests, plots, or interpretations.
Record `input_scale`, `log_base`, and any warning given under Materials alongside the
execution metadata, not as additional rows or columns in this output table.

## Notes

The Shannon index was introduced by Shannon (1948), which is cited as both the
source of this protocol's calculation and the method origin. Manghi et al.
(*Nature Communications*, 2025, DOI: 10.1038/s41467-025-66888-1) is an application
of the method, not the method-origin citation.

Both `vegan::diversity()` and `skbio.diversity.alpha.shannon()` divide each sample by its
own total, so they accept counts and proportions alike; neither checks the conditions in
Materials, which must be checked first. `vegan::diversity()` returns `NA` for a sample
with a missing value rather than stopping. Pass the logarithm base explicitly to both:
the default of `shannon()` has differed between scikit-bio releases.

## History & Reviews
<!-- Newest versions at the top -->

### Version 1.0.0 (2026-10-26)

#### Changes
- Initial protocol for calculating per-sample Shannon diversity.

#### Reviews

**Review by Levi Waldron ([0000-0003-2725-0694](https://orcid.org/0000-0003-2725-0694))**
- **Date:** 2026-10-27
- **Status:** `approved`
- **Notes:** Review given on waldronlab/agent-protocols#53.
