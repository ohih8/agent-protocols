---
name: "cbea-bugsigdb-enrichment"
description: "Test harmonized BugSigDB microbial signatures for taxonomic enrichment with competitive balances of compositional abundances."
version: "1.0.0"
authors:
  - name: "Otto Infield-Harm"
    orcid: "0009-0009-8629-6124"
  - name: "Levi Waldron"
    orcid: "0000-0003-2725-0694"
date: "2026-10-14"
status: draft
type: "atomic"

artifact_doi: ~
collection_doi: "10.5281/zenodo.22731694"
protocol_citation: "10.1371/journal.pcbi.1010091"
method_origin_citation: "10.1371/journal.pcbi.1010091"
license: "CC-BY-4.0"
protocols_used: []
upstream_repositories:
  - "https://github.com/qpmnguyen/CBEA"
  - "https://github.com/waldronlab/BugSigDBPaper"
database_urls:
  - "https://bugsigdbr.bioconductor.org/"
key_packages:
  - "CBEA"
  - "bugsigdbr"
  - "BiocSet"
category: "Statistical Analysis"
tags: [enrichment, microbe-sets, cbea, bugsigdb, compositional-data]
---

# CBEA Enrichment of BugSigDB Microbial Signatures

This protocol calculates competitive balance enrichment scores and sample-level
significance for BugSigDB microbial signatures. For each signature and sample,
the score contrasts the geometric mean abundance of the signature taxa with the
geometric mean abundance of all other taxa in the declared background universe.

## Materials

- **Abundance data**
  - A taxa-by-sample matrix of strictly positive relative abundances, or another
    positive compositional representation whose scale and construction are
    recorded. Counts must be converted to a positive compositional representation
    before this protocol begins.
  - The matrix must have unique taxon and sample identifiers. This protocol does
    not define sequencing quality control, normalization, zero replacement, or
    the choice of an upstream transformation. Record those decisions and the
    resulting abundance scale.
- **BugSigDB signatures**
  - A collection of signatures, each identified by a stable signature identifier
    and containing taxa at a declared taxonomic rank.
  - Obtain the signatures from BugSigDB or a versioned export and record the
    release, retrieval date, source URL, and any filters used to select them.
- **Parameters**
  - `rank`: exactly one taxonomic rank shared by the abundance table and all
    signatures in an execution.
  - `analysis_scope`: exactly one of `pooled` or `per_dataset`.
  - `min_set_size`: 2 matched taxa.
  - `max_set_size`: `|U| - 1`, where `U` is the execution's harmonized
    background universe.
  - `n_perm`: 100 permutations for the empirical null distribution.
  - `distribution`: normal parametric fit to the permutation scores.
  - `variance_inflation_adjustment`: enabled to account for inter-taxa
    correlation.
  - `multiple_testing_method`: Benjamini-Hochberg false-discovery-rate
    adjustment across all retained signatures tested in the same execution.
  - `significance_threshold`: adjusted p-value less than 0.05.
  - `random_seed`: a recorded integer seed used for permutation reproducibility.

## Steps

### Step 1: Confirm the abundance matrix and analysis scope

Verify that the abundance matrix is taxa by sample, that identifiers are unique,
and that every value supplied to CBEA is finite and strictly positive. Reject
the input and report the failed check if any value is zero, negative, missing,
or non-finite; do not silently add a pseudocount or replace a value. A
pseudocount or zero-replacement method must be selected and applied before this
protocol, with its value or algorithm recorded.

Select `pooled` or `per_dataset` before examining enrichment results. Use
`pooled` only when datasets have compatible taxonomic processing, abundance
scales, and sample-level metadata. In pooled mode, combine the compatible
datasets and define one universe across the combined table. In per-dataset
mode, keep datasets separate and repeat Steps 2 through 5 independently for
each dataset, defining a separate universe and set of tested signatures for
each one. Do not treat a pooled analysis as an aggregation of independent
dataset results.

Record the number of datasets, samples, taxa, the abundance representation,
the upstream preprocessing, the selected scope, and the random seed.

### Step 2: Harmonize taxonomy and define the background universe

Choose one `rank`, such as species or genus, before matching signatures. Convert
the taxon identifiers in the abundance matrix and the BugSigDB signatures to
that rank using the same naming convention and taxonomy mapping. Do not infer a
lower-rank identity from a higher-rank name, and do not combine identifiers
whose rank is ambiguous.

For each signature, retain only members that match a taxon in the abundance
matrix after harmonization. Record every unmatched signature member and its
reason for exclusion. Define `U` as the set of all unique harmonized taxa
remaining in the abundance matrix for the selected analysis scope. The
complement for a signature `S` is `U \ S`; it is not the set of taxa appearing
in other BugSigDB signatures.

Record the rank, taxonomy mapping or normalization rules, the size and
identifiers of `U`, and the matched and unmatched members of every signature.
Discard an empty signature before testing and record that reason.

### Step 3: Apply the set-size bounds

For each signature, let `S` be its unique matched taxa and let `k = |S|`.
Retain the signature only when:

```text
2 <= k <= |U| - 1
```

The lower bound excludes singleton signatures, for which the protocol does not
define a stable set-level balance. The upper bound guarantees a non-empty
complement. If `|U| < 3`, no signature can be tested. Remove duplicate
signature members before calculating `k`; do not count repeated BugSigDB rows as
distinct taxa.

Record the original member count, matched member count, duplicate count,
unmatched members, `|U|`, the final set size, and the exclusion reason for every
discarded signature. Do not change the bounds after looking at the results.

### Step 4: Calculate competitive balance scores and sample-level inference

For every retained signature `S` and sample, calculate the CBEA competitive
balance using the positive abundances:

```text
score(S, sample) =
  sqrt(|S| * |U \ S| / |U|) *
  (mean(log(abundance in S)) - mean(log(abundance in U \ S)))
```

This is the isometric log-ratio contrast between the signature subcomposition
and its complement. A positive score means that the signature taxa have a
higher geometric-mean abundance than the complement in that sample; a negative
score means the reverse. Do not replace this contrast with a sum or arithmetic
mean of signature abundances.

Estimate the empirical null by permuting the taxon labels within each sample
`n_perm = 100` times while preserving the sample's abundance values and the
signature size. Use the recorded random seed. Fit the specified normal
distribution to the permutation scores and use the resulting sample-level
null inference. Apply the variance-inflation adjustment for inter-taxa
correlation. Return the CBEA score, the unadjusted p-value, and the adjusted
p-value for every retained signature and sample.

The null hypothesis is competitive: within a sample, the taxa in the signature
have the same balance-level abundance as the measured taxa outside the
signature. It is not a null that every signature taxon has no association with
the sample condition, and it is not a test that every member of the signature
is individually differentially abundant.

Record the CBEA implementation and version, all parameters, the random seed,
the number of successful permutations, and any signature-sample result that
could not be computed.

### Step 5: Adjust across signatures and report the results

Within each execution (the pooled table or one per-dataset table), apply the
Benjamini-Hochberg procedure to the complete family of unadjusted p-values for
the retained signatures. Use the same family definition for every sample in
that execution and do not adjust only the signatures that were significant
before correction. Mark a signature-sample result significant when its adjusted
p-value is less than 0.05.

Produce a result table containing the dataset or pooled-scope identifier,
sample identifier, signature identifier, matched taxa, complement size, CBEA
score, unadjusted p-value, adjusted p-value, and significance decision. Also
produce an execution manifest containing the rank, universe, set-size bounds,
preprocessing description, inference parameters, exclusion counts, and
software versions.

## Notes

The CBEA paper describes this method as a single-sample taxon-set enrichment
method based on the competitive null hypothesis and isometric log-ratio
transformation. The BugSigDB paper's `Figure3.Rmd` supplies the application
context for running CBEA on pooled and individual microbiome datasets, but it
does not replace the method's primary citation.

This protocol requires positive compositional abundances because the balance
uses logarithms. The choice of zero replacement, pseudocount, normalization,
taxonomic database, and differential-abundance input is upstream and must be
specified separately. The `0.5` addition shown in the application vignette is
not silently prescribed here; if it is used, record its scale and rationale
before running this protocol.

A significant result licenses only the conclusion that the signature's
within-sample balance is unusual under the declared competitive null and
preprocessing. It does not establish causality, biological activity, disease
mechanism, differential abundance of every member, or generalization beyond
the analyzed samples.

This protocol does not produce the abundance input, retrieve or curate
BugSigDB signatures, compare CBEA with ORA or PADOG, perform downstream
between-group differential analysis of the CBEA scores, or interpret a
signature's biological meaning. Those operations require separate protocols.

## History & Reviews
<!-- Newest versions at the top -->

### Version 1.0.0 (2026-10-14)

#### Changes
- Initial protocol creation.
- Specified taxonomy harmonization, background universe, set-size bounds,
  competitive balance calculation, permutation inference, multiple-testing
  adjustment, and pooled versus per-dataset execution.

#### Reviews
*No reviews yet.*
