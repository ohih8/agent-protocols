---
name: prevalence-filtering
description: Remove microbiome features detected in fewer than a prespecified fraction of analysis samples before a separately specified downstream analysis.
version: 1.0.0
authors:
  - name: Otto Infield-Harm
    orcid: 0009-0009-8629-6124
  - name: Levi Waldron
    orcid: 0000-0002-2725-0694
date: 2026-09-22
status: draft

license: CC-BY-4.0
type: atomic

protocol_citation: "10.12688/f1000research.8986.1"
method_origin_citation: "10.12688/f1000research.8986.1"

artifact_doi: ~
collection_doi: "10.5281/zenodo.22731694"

upstream_repositories: []
database_urls: []

protocols_used: []
key_packages: []
category: "Data Transformation"
tags: [prevalence-filter, filtering, microbiome, preprocessing]
---

# Prevalence Filtering of Microbiome Features

This protocol removes features detected in fewer than a prespecified fraction of
samples before a separately specified transformation, modelling procedure, or other
downstream analysis. It defines prevalence and the filtering operation without
requiring a particular data format or software implementation.

The protocol uses "sample" for one biological observation and "feature" for one
taxon or functional category. A feature is retained when its prevalence is at
least the selected threshold; features below the threshold are removed.

## Materials

- **Input data**
  - A feature-by-sample abundance table with unique feature and sample identifiers.
    The table may use the opposite orientation, but its orientation must be recorded.
  - Sample metadata identifying the prespecified cohort and any study, batch, body
    site, or other grouping used to define the analysis set.
- **Parameters**
  - `prevalence_threshold`: the minimum fraction of eligible samples in which a
    feature must be detected to be retained. The default is `0.01` (1%).
  - `detection_rule`: a feature is detected when its abundance is greater than zero.
    This is the default rule. A positive detection floor may be selected instead,
    but its value and units must be recorded.
  - `prevalence_scope`: compute prevalence after selecting the cohort and other
    analysis-set restrictions. This is the default. Computing prevalence before
    subsetting is an alternative that must be selected and recorded before filtering.
  - `multi_dataset_scope`: apply filtering separately within each dataset. This is
    the default. Pooled filtering across datasets is an alternative that must be
    selected and recorded before filtering.

## Steps

### Step 1: Define the analysis set and filtering parameters

Specify the cohort, inclusion and exclusion criteria, repeated-sample rule, and
dataset membership before calculating prevalence. Select the
`prevalence_threshold`, `detection_rule`, `prevalence_scope`, and
`multi_dataset_scope` before evaluating downstream results. The threshold must be
between 0 and 1 inclusive. Record the selected values and their units.

For the default `prevalence_scope`, select the cohort first and calculate
prevalence only over the resulting eligible samples. If prevalence is calculated
before cohort subsetting, define that broader sample set explicitly and use it
consistently for every feature.

For the default `multi_dataset_scope`, perform the operation independently for
each dataset and retain a feature if it passes within that dataset. If pooled
filtering is selected, define the pooled sample set and apply one threshold to that
set. Do not change between per-dataset and pooled filtering after examining the
results.

### Step 2: Validate samples, features, and abundances

Confirm that feature and sample identifiers are unique and that every sample in the
selected analysis set has a corresponding abundance column or row. Resolve
duplicate, missing, or contradictory identifiers before filtering. Record the
number of eligible samples and features for each dataset or pooled analysis set.

Confirm that abundances are numeric, finite, and non-negative. Preserve explicit
zeros. Do not silently treat missing values as zero. Exclude samples or features
with invalid values according to a prespecified missing-data rule, or stop and
resolve the invalid input. Record every exclusion and its reason.

If a detection floor is used, confirm that all abundance values are on a comparable
scale and record the floor and units. Under the default rule, a feature is detected
when its abundance is greater than zero. A zero-total sample is still included in
the prevalence denominator when it is otherwise eligible; if it is excluded, record
that decision before calculating prevalence.

### Step 3: Calculate feature prevalence

For each eligible feature, count the eligible samples in which the feature satisfies
the selected detection rule. Let `n_detected` be this count and `n_samples` be the
number of samples in the selected prevalence scope. Calculate:

```text
prevalence = n_detected / n_samples
```

Use the same denominator and detection rule for every feature within an analysis
set. If `n_samples` is zero, do not filter; stop and report that the analysis set
contains no eligible samples. Record `n_detected`, `n_samples`, and prevalence for
every feature, including features with prevalence zero.

### Step 4: Apply the prevalence threshold

Retain a feature when `prevalence >= prevalence_threshold` and remove it when
`prevalence < prevalence_threshold`. Do not round prevalence before making this
comparison. At the default threshold of `0.01`, a feature must be detected in at
least 1% of eligible samples. The threshold is a parameter rather than a universal
constant: Callahan et al. used 5% in their workflow, whereas the curated
metagenomic application motivating this protocol used 1%.

If a threshold expressed as a fraction does not correspond to an integer number of
samples, apply the comparison to the unrounded fraction rather than silently
rounding the threshold. Equivalently, a feature passes when
`n_detected >= prevalence_threshold * n_samples`. Record the number of retained and
removed features and the smallest retained and largest removed prevalence values
when those sets are non-empty.

### Step 5: Produce the filtered data and provenance record

Remove the selected features while preserving the original values and identifiers
of retained features and samples. Do not transform, normalize, impute, or otherwise
alter retained abundances as part of this protocol.

Output the filtered abundance table and a feature-level report containing each
feature's `n_detected`, `n_samples`, prevalence, detection rule, detection floor if
used, threshold, filtering scope, dataset scope, and retain/remove decision. Also
report the analysis-set definition, excluded samples, input and output feature
counts, and the software and version used if an implementation was used.

Pass the filtered data to a separately specified downstream analysis. If the
filtered data will be used for centred log-ratio transformation or another
transformation, record that filtering occurred first because the retained feature
set changes the subsequent calculation.

## Notes

Testing of this protocol using GPT 5.6-Luna passed all generated test cases, many of which were designed to probe edge cases.
The test cases are not included here as they were not created in the explicit test format nor run by the official tester.

Callahan et al. describe prevalence as the number or fraction of samples in which a
taxon appears at least once and motivate filtering as a way to avoid effort spent on
rare taxa and artifacts of heuristic OTU clustering. Their workflow uses a 5%
threshold. A 1% default is used here because it matches the motivating curated
metagenomic application, but it is not canonical and should be justified for the
dataset and feature-resolution method. The rationale for filtering may be weaker
for sequence-resolved profiles than for heuristically clustered OTUs, which is
another reason to record the selected threshold rather than treating it as fixed.

This protocol is distinct from [independent-filtering-variance](../independent-filtering-variance/protocol.md),
which filters features by an overall variance or mean rather than by the fraction
of samples in which they are detected.

## Out of scope

This protocol does not define or perform abundance transformation, including
centred log-ratio transformation; normalization; imputation; variance- or
mean-based filtering; differential abundance testing; hypothesis testing;
multiple-testing adjustment; modelling; or interpretation of downstream results.

## History & Reviews
<!-- Newest versions at the top -->

### Version 1.0.0 (2026-09-22)

#### Changes
- Initial protocol creation.

#### Reviews
*No reviews yet.*
