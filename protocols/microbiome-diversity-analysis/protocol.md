---
name: microbiome-diversity-analysis
description: Calculate within-sample and between-sample microbiome diversity and test whether diversity differs by a prespecified grouping variable.
version: 1.1.0
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

This protocol calculates one within-sample diversity value (alpha diversity) for
each sample and one between-sample dissimilarity value (beta diversity) for each pair
of samples. It can then test whether alpha diversity or beta diversity differs
between prespecified groups. It is independent of the software used to calculate the
metrics or tests.

The protocol uses "sample" for one biological observation and "feature" for one
taxon or functional category. It does not require a particular file format, but the
same sample and feature identifiers must be retained in every input and output.

## Materials

- An abundance table in which each value identifies one feature in one sample. The
  table may have samples in rows or columns, but its orientation must be recorded.
  Use one consistent taxonomic or functional resolution throughout.
- A metadata table with exactly one row per sample and a unique sample identifier.
  It must contain the prespecified grouping variable and any covariates or strata
  used in the analysis.
- A complete specification of the abundance scale, zero handling, distance metric,
  permutation scheme, and multiple-testing procedure.

## Steps

### Step 1: Define the analysis set and analysis objectives

Specify the body site, participant population, study-design restrictions, repeated
sample rule, and exclusion criteria before calculating diversity. Retain a record of
every excluded sample and the reason for exclusion. Do not select samples based on
their diversity values or on a result of the group comparison.

State which outputs are required: alpha diversity, beta diversity, alpha-diversity
group testing, beta-diversity group testing, or all four. Alpha diversity is
calculated separately for each sample. Beta diversity is calculated for every
eligible pair of samples. A group test is performed only when a grouping variable
and a prespecified test are provided.

Confirm that every abundance-table sample identifier appears exactly once in the
metadata and that every metadata sample selected for analysis appears in the
abundance table. Resolve duplicate, missing, or contradictory identifiers before
analysis. Record the number of samples and samples per group.

### Step 2: Validate and prepare abundances

Confirm that every abundance is numeric, finite, and non-negative. Specify whether
the input is raw counts, proportions, or another measurement scale. Use the same
preprocessing for every sample. Do not mix taxonomic resolutions or abundance scales.

For each sample, calculate its total abundance as the sum of all valid feature
values. If the input is counts or another non-proportional scale, convert it to
proportions by dividing each feature value by that sample total before calculating
Shannon entropy. If the input is already proportions, verify that the values are
non-negative and that each sample sums to one within a stated numerical tolerance;
renormalize only if that rule was prespecified. Record the tolerance and any
renormalization.

Do not silently treat missing values as zero. Preserve explicit zeros. Exclude a
sample with a missing or invalid feature value, or apply a prespecified feature-level
missing-data rule consistently before calculating any metric. Exclude and report a
sample whose total abundance is zero or cannot be calculated.

### Step 3: Calculate alpha diversity

For each eligible sample, calculate Shannon entropy over the feature proportions
created in Step 2:

```text
H = -sum(p_i * ln(p_i)) for all features with p_i > 0
```

Use the natural logarithm unless another base is prespecified. Include every retained
feature with positive proportion in the sum; zero-proportion features contribute zero
and do not need to be listed in the calculation. An absent feature is treated as
zero only when the abundance table represents a complete profile and the absence is
not a missing measurement.

Report one alpha-diversity value per sample, the number of retained features, the
number of positive-abundance features, and the preprocessing decisions used for that
sample.

If comparing groups, prespecify the test, alternative hypothesis, covariate
adjustment, and multiple-testing correction. Report effect estimates, uncertainty, and
the number of observations used, not only a p-value.

### Step 4: Construct a beta-diversity matrix

Choose and record one dissimilarity metric appropriate to the abundance scale and
scientific question. Examples include Bray-Curtis for non-negative abundance
profiles and Jaccard for presence/absence profiles. Calculate one pairwise distance
for every eligible pair of samples, including pairs from the same group. The metric,
normalization, and any feature filtering must be identical for all samples.

If converting abundances to presence/absence, define the positive-abundance
threshold before calculating the matrix. Do not choose a threshold separately for
different groups or samples.

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

For alpha diversity, use the prespecified univariate test from Step 3 and report its
effect estimate separately from the beta-diversity test. For beta diversity,
interpret a significant PERMANOVA result as evidence that group centroids differ in
the selected distance space, not that every sample differs or that one particular
feature caused the result. Interpret ANOSIM according to its selected rank-based
statistic and report that statistic.

For PERMANOVA, also assess whether group differences in within-group dispersion could
explain an apparent centroid/location effect. Do not interpret a significant result
as proof of causation or transmission.

### Step 6: Report provenance and quality checks

Report the input dataset and version, table orientation, sample-selection rules,
feature resolution, abundance preprocessing, alpha-diversity definition, distance
metric, model,
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

### Version 1.1.0 (2026-09-16)

#### Changes
- Clarified required inputs, table orientation, analysis objectives, identifier joins,
  normalization, missing values, absent features, distance choices, and interpretation
  of alpha- and beta-diversity tests.

#### Reviews
*No reviews yet.*

### Version 1.0.0 (2026-09-16)

#### Changes
- Initial protocol creation.

#### Reviews
*No reviews yet.*
