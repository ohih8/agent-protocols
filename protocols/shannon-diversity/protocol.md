---
name: "shannon-diversity"
description: "Calculate the Shannon diversity index for each sample in a feature-by-sample abundance table."
version: "1.0.0"
authors:
  - name: "Otto Infield-Harm"
    orcid: "0009-0009-8629-6124"
  - name: "Levi Waldron"
    orcid: "0000-0003-2725-0694"
reviews: []
date: "2026-10-20"
status: draft
type: "atomic"

artifact_doi: ~
collection_doi: "10.5281/zenodo.22731694"
protocol_citation: "10.1002/j.1538-7305.1948.tb01338.x"
method_origin_citation: "10.1002/j.1538-7305.1948.tb01338.x"
license: "CC-BY-4.0"
protocols_used: []
upstream_repositories: []
database_urls: []
key_packages: []
category: "Statistical Analysis"
tags:
  - alpha-diversity
  - shannon
  - microbiome
---

# Shannon diversity of a single sample

Calculate one Shannon diversity value for each sample in a feature-by-sample abundance
table. The calculation is descriptive: this protocol does not compare groups, test
hypotheses, or interpret the resulting values.

## Materials

- **Input table:** A CSV file with one row per feature and one column per sample. The
  first column contains unique feature identifiers, and the remaining columns contain
  non-negative numeric abundances.
- **Input scale:** A required parameter, either `counts` or `proportions`.
- **Logarithm base:** An optional parameter, defaulting to the natural logarithm (`e`).
  The base must be greater than 1.
- **Depth normalization:** `none`. This protocol does not rarefy or otherwise
  normalize samples to a common sequencing depth. The supplied abundances and this
  setting must be recorded with the result.

## Steps

### Step 1: Validate the table and parameters

Read the input as a feature-by-sample table and preserve the sample column names.
Reject the input if the feature identifiers are missing or duplicated, if there are
no sample columns, if an abundance is non-numeric or negative, or if any abundance
is missing. Missing values must stop the calculation; they must not be imputed,
converted to zero, or silently omitted.

Require the input scale to be explicitly declared as `counts` or `proportions`.
Reject any other value. Require the logarithm base to be greater than 1.

### Step 2: Convert abundances to proportions

For each sample independently, let $x_{ij}$ be the abundance of feature $i$ in
sample $j$.

For `counts`, calculate

$$
p_{ij} = \frac{x_{ij}}{\sum_{i} x_{ij}}.
$$

Reject a count sample whose total is zero. For `proportions`, require the values to
be non-negative and to sum to 1 within a numerical tolerance of $10^{-12}$;
reject a sample outside that tolerance rather than silently renormalizing it.

Do not rarefy, subsample, or otherwise alter the sample totals. Zero-valued
features remain part of the table but make no contribution to the index.

### Step 3: Calculate Shannon diversity

For each sample $j$, calculate

$$
H_j = -\sum_{i:p_{ij}>0} p_{ij}\log_b(p_{ij}),
$$

where $b$ is the requested logarithm base. Terms with $p_{ij}=0$ are omitted,
because their limiting contribution is zero.

### Step 4: Write the result

Write a CSV table with exactly two columns, `sample_id` and `shannon`, and one row
for each input sample. Preserve the input sample-column order. Do not add feature
summaries, group labels, confidence intervals, tests, plots, or interpretations.
Record the input scale, logarithm base, and `depth_normalization: none` alongside
the execution metadata, not as additional rows or columns in this output table.

## Notes

The Shannon index was introduced by Shannon (1948), which is cited as both the
source of this protocol's calculation and the method origin. Manghi et al.
(*Nature Communications*, 2025, DOI: 10.1038/s41467-025-66888-1) is an application
of the method, not the method-origin citation.

This protocol intentionally does not perform rarefaction. Because Shannon diversity
can depend on sequencing depth, comparisons across samples with different depths
must be treated as potentially confounded unless depth normalization is handled by
a separate, explicitly documented procedure.

## History & Reviews
<!-- Newest versions at the top -->

### Version 1.0.0 (2026-10-20)

#### Changes
- Initial protocol for calculating per-sample Shannon diversity.

#### Reviews
*No reviews yet.*
