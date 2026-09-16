---
name: microbiome-diversity-analysis
description: Calculate within-sample and between-sample microbiome diversity and test whether diversity differs by a prespecified grouping variable.
version: 1.0.0
authors:
  - name: Otto Infield-Harm
    orcid: 0009-0009-8629-6124
  - name: Levi Waldron
    orcid: 0000-0002-2725-0694
date: 2026-09-16
status: draft
license: CC-BY-4.0
type: atomic

protocol_citation: "10.1038/s41467-025-66888-1"

artifact_doi: ~
collection_doi: ~

upstream_repositories: []
database_urls: []

protocols_used: []
key_packages: []
category: metagenomics
tags: [microbiome, diversity, shannon, beta-diversity, permanova, anosim]
---

# Microbiome Diversity Analysis

This protocol calculates alpha diversity and tests group-associated differences in
microbiome composition using beta diversity. It is independent of the software used
to calculate the metrics or tests.

## Materials

- A feature-by-sample abundance table with one consistent taxonomic or functional
  resolution.
- Sample metadata containing the prespecified grouping variable and any covariates or
  strata used in the analysis.
- A complete specification of the abundance scale, zero handling, distance metric,
  permutation scheme, and multiple-testing procedure.

## Steps

### Step 1: Define the analysis set

Specify the body site, participant population, study-design restrictions, repeated
sample rule, and exclusion criteria before calculating diversity. Retain a record of
every excluded sample and the reason for exclusion. Do not select samples based on
their diversity values or on a result of the group comparison.

Confirm that samples and metadata identifiers match uniquely. Resolve duplicate,
missing, or contradictory identifiers before analysis. Record the number of samples
and samples per group.

### Step 2: Validate and prepare abundances

Confirm that every abundance is numeric, finite, and non-negative. Specify whether
the input is counts, proportions, or another measurement scale. Use the same
preprocessing for every sample. Do not mix taxonomic resolutions or abundance scales.

If the metric requires proportions, normalize each sample by its total positive
abundance and record samples that cannot be normalized. Do not silently treat missing
values as zero. Preserve explicit zeros.

### Step 3: Calculate alpha diversity

For each sample, calculate Shannon entropy over its non-negative feature proportions:

```text
H = -sum(p_i * ln(p_i)) for all features with p_i > 0
```

Use the natural logarithm unless another base is prespecified. Define the treatment of
unobserved features, zero-total samples, and features removed during preprocessing.
Report one alpha-diversity value per sample together with the preprocessing and
effective feature count.

If comparing groups, prespecify the test, alternative hypothesis, covariate
adjustment, and multiple-testing correction. Report effect estimates, uncertainty, and
the number of observations used, not only a p-value.

### Step 4: Construct a beta-diversity matrix

Choose and record a dissimilarity metric appropriate to the abundance scale and
scientific question. Calculate one pairwise distance for every eligible pair of
samples. The metric, normalization, and any feature filtering must be identical for
all samples.

Verify that the matrix is square, symmetric, has zero diagonal, and retains unique
sample identifiers. Record the number of samples and the metric used.

### Step 5: Test group-associated composition differences

Use a permutation-based multivariate test such as PERMANOVA or ANOSIM only after
specifying:

- the model terms and their order,
- the grouping variable and covariates,
- the number and random-seed policy for permutations,
- whether permutations are unrestricted or constrained within study, participant,
  batch, or another stratum,
- the statistic and the direction of interpretation.

For PERMANOVA, also assess whether group differences in within-group dispersion could
explain an apparent location effect. Do not interpret a significant result as proof of
causation or transmission.

### Step 6: Report provenance and quality checks

Report the input dataset and version, sample-selection rules, feature resolution,
abundance preprocessing, alpha-diversity definition, distance metric, model,
permutation scheme, random-seed policy, missing-data handling, and software
implementation if one was used.

Confirm that all output values are finite and that the sample identifiers in every
output match the analysis set. Preserve the per-sample alpha-diversity table and the
beta-diversity matrix as versioned artifacts.

## Notes

Manghi et al. used Shannon entropy for alpha diversity and PERMANOVA and ANOSIM for
beta-diversity analyses in curated human metagenomes. Those choices are examples of
this protocol, not defaults for every dataset. The protocol does not define a
particular distance metric or a universal covariate set because those depend on the
study design.

## History & Reviews
<!-- Newest versions at the top -->

### Version 1.0.0 (2026-09-16)

#### Changes
- Initial protocol creation.

#### Reviews
*No reviews yet.*
