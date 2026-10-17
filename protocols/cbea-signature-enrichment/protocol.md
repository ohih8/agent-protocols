---
name: "cbea-signature-enrichment"
description: "Test microbial signatures, such as those curated in BugSigDB, for taxonomic enrichment in each sample using competitive balances of compositional abundances (CBEA)."
version: "1.0.0"
authors:
  - name: "Otto Infield-Harm"
    orcid: "0009-0009-8629-6124"
  - name: "Levi Waldron"
    orcid: "0000-0003-2725-0694"
date: "2026-10-17"
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
  - "https://bugsigdb.org/"
key_packages:
  - "CBEA"
  - "BiocSet"
  - "bugsigdbr"
category: "Statistical Analysis"
tags: [enrichment, microbe-sets, cbea, bugsigdb, compositional-data]
---

# CBEA Enrichment of Microbial Signatures

This protocol calculates a competitive balance enrichment score and a
sample-level p-value for each microbial signature in each sample. A signature
is any named set of taxa at one taxonomic rank. The signatures curated in
BugSigDB are the worked example throughout, but the protocol applies to any
collection of signatures: sets from another database, taxa reported by a
previous study, or sets defined by the analyst before seeing the data. For each
signature and sample, the score contrasts the geometric mean relative abundance
of the signature taxa with the geometric mean relative abundance of all other
taxa in the declared background universe.

## Materials

- **Abundance data**
  - A taxa-by-sample matrix of strictly positive relative abundances: each
    column sums to a constant such as 1 or 100 before zero replacement, and no
    value is zero after it. Counts must be converted to relative abundances,
    and zeros must be handled, before this protocol begins.
  - The matrix must be on the linear proportion scale. Do not supply
    log-transformed, centered log-ratio (CLR), or other transformed values; the
    balance in Step 4 takes logarithms itself, and Step 2 may sum rows, which
    is meaningful only on the linear scale.
  - Taxon and sample identifiers must be unique. Taxon identifiers must carry
    the rank of each row, either in the identifier itself (for example a
    lineage string) or in an accompanying taxonomy table.
  - This protocol does not define sequencing quality control, normalization,
    or zero replacement. Record those decisions and the resulting abundance
    scale.
- **Signatures**
  - A collection of signatures, each identified by a stable signature
    identifier and containing taxa with a declared taxonomic rank. Any format
    that preserves the identifier and the member taxa is acceptable, such as a
    named list, a GMT file, or a `BiocSet`.
  - Record the source, version or release, retrieval date, and any filters used
    to select signatures.
  - **BugSigDB example.** Import a versioned BugSigDB release with
    `bugsigdbr::importBugSigDB(version = <release or Zenodo DOI>)` and extract
    signatures at the chosen rank with
    `bugsigdbr::getSignatures(tax.id.type = "taxname", tax.level = rank,
    exact.tax.level = FALSE)`. The last argument maps members curated below
    `rank` up to `rank`, which is the mapping Step 2 permits. The release
    identifier is the version to record.
- **Software**
  - `CBEA` (https://github.com/qpmnguyen/CBEA), the Bioconductor implementation
    of the method. Function names below refer to this package; an
    implementation in another language must reproduce the operations named in
    Step 4, not only the function's description.
- **Parameters**
  - `rank`: exactly one taxonomic rank shared by the abundance table and all
    signatures in an execution.
  - `analysis_scope`: exactly one of `pooled` or `per_dataset`.
  - `min_set_size`: 2 matched taxa.
  - `max_set_size`: `|U| - 1`, where `U` is the execution's background
    universe.
  - `n_perm`: 100 random taxon subsets for the null distribution.
  - `distribution`: `norm`, a normal distribution fitted to the null scores.
    `mnorm`, the two-component normal mixture from the same paper, is the only
    permitted alternative and must be recorded if chosen.
  - `correlation_adjustment`: enabled (`adj = TRUE` in `CBEA::cbea()`).
  - `multiple_testing_method`: Benjamini-Hochberg false-discovery-rate
    adjustment, with the family defined in Step 5.
  - `significance_threshold`: adjusted p-value less than 0.05.
  - `random_seed`: a recorded integer seed used for the random taxon subsets.

## Steps

### Step 1: Confirm the abundance matrix and analysis scope

Verify that the abundance matrix is taxa by sample, that identifiers are
unique, that the values are relative abundances on the linear scale, and that
every value is finite and strictly positive. Reject the input and report the
failed check if any value is zero, negative, missing, or non-finite; do not
silently add a pseudocount or replace a value. A pseudocount or
zero-replacement method must be selected and applied before this protocol,
with its value or algorithm recorded. Note that `CBEA::cbea()` adds a
pseudocount of `1e-5` with only a warning when it finds zeros; this protocol's
check must run before that function so the silent replacement never happens.

Select `pooled` or `per_dataset` before examining enrichment results. Use
`pooled` only when datasets have compatible taxonomic processing, abundance
scales, and sample-level metadata. In pooled mode, combine the compatible
datasets and define one universe across the combined table. In per-dataset
mode, keep datasets separate and repeat Steps 2 through 5 independently for
each dataset, defining a separate universe and set of tested signatures for
each one. Do not treat a pooled analysis as an aggregation of independent
dataset results.

Record the number of datasets, samples, taxa, the abundance representation,
the upstream preprocessing including the zero-replacement method, the selected
scope, and the random seed.

### Step 2: Harmonize taxonomy and define the background universe

Choose one `rank`, such as species or genus, before matching signatures.
Express the taxon identifiers in the abundance matrix and in the signatures at
that rank using the same naming convention and the same taxonomy. The vignette
this protocol was extracted from used NCBI taxonomy names at the genus and
species ranks on both sides.

**Abundance rows.** Prefer an abundance table already at `rank`. When the
table is at a lower rank, aggregate it to `rank` by summing the relative
abundances of all rows that map to the same `rank` taxon, within each sample.
Summing is valid only because the input is on the linear proportion scale. Do
not relabel rows without summing: two species rows relabeled to one genus
would leave duplicate identifiers and would be scored as two members of the
complement. Exclude rows that cannot be mapped to a single taxon at `rank`,
including rows whose classification stops above `rank` and rows whose
identifier is ambiguous, and emit a warning naming each excluded row and the
abundance it carried, because such rows may indicate incompatible or
incorrect taxonomic labeling. After aggregation and exclusion, confirm again
that identifiers are unique and that every value is strictly positive. Zero
replacement should have been applied to the table at `rank`; if it was
applied before aggregation, record that, since the summed pseudocounts then
differ between taxa.

**Signature members.** Map each member to `rank` using the same taxonomy. A
member curated below `rank` maps up to its ancestor at `rank` (a species to
its genus). A member curated above `rank`, or one with no identity at `rank`,
is unmappable: exclude it and record it. Do not infer a lower-rank identity
from a higher-rank name. Then retain only members that match a taxon in the
harmonized abundance matrix. Record every unmatched member and its reason for
exclusion. In the `CBEA` package, `BiocSet::filter_elementset()` performs the
matching step; `CBEA::cbea()` would also drop unmatched members silently, but
the protocol requires them to be recorded, so match explicitly.

**Universe.** Define `U` as the set of all unique harmonized taxa remaining in
the abundance matrix for the selected analysis scope. This is what
`CBEA::get_raw_score()` uses: the complement of a signature `S` is every other
column of the matrix, `U \ S`. It is not the set of taxa appearing in other
signatures, and it is not restricted to taxa appearing in any signature.

Record the rank, the taxonomy and mapping rules, the number of abundance rows
aggregated and excluded, the size and identifiers of `U`, and the matched and
unmatched members of every signature. Discard an empty signature before
testing and record that reason.

### Step 3: Apply the set-size bounds

For each signature, let `S` be its unique matched taxa and let `k = |S|`.
Retain the signature only when:

```text
2 <= k <= |U| - 1
```

The lower bound excludes singleton signatures, for which the protocol does
not define a stable set-level balance. The upper bound guarantees a non-empty
complement. If `|U| < 3`, no signature can be tested. Remove duplicate
signature members before calculating `k`; do not count repeated source rows
as distinct taxa.

Record the original member count, matched member count, duplicate count,
unmatched members, `|U|`, the final set size, and the exclusion reason for
every discarded signature. Do not change the bounds after looking at the
results.

### Step 4: Calculate competitive balance scores and sample-level inference

For every retained signature `S` and sample, calculate the CBEA competitive
balance from the positive relative abundances:

```text
score(S, sample) =
  sqrt(|S| * |U \ S| / |U|) *
  (mean(log(abundance in S)) - mean(log(abundance in U \ S)))
```

This is the isometric log-ratio contrast between the signature subcomposition
and its complement, computed by `CBEA::get_raw_score()`. A positive score
means that the signature taxa have a higher geometric-mean abundance than the
complement in that sample; a negative score means the reverse. Do not replace
this contrast with a sum or arithmetic mean of signature abundances.

Construct the null distribution for each signature as `CBEA::fit_scores()`
does, pooling across samples rather than building a separate null per sample:

1. Draw `n_perm = 100` random subsets of `U`, each of size `k`, without
   replacement within a subset, using the recorded seed. In the `CBEA`
   package (version 0.99.4), the exported `cbea()` methods accept a
   `parallel_backend` argument but do not forward it to the internal
   fitter, which seeds a serial backend from the current R random stream.
   Call `set.seed(random_seed)` immediately before `CBEA::cbea()` and do
   not rely on the `parallel_backend` argument for reproducibility.
2. Apply every subset to every sample with the formula above. This yields
   `n_perm` times the number of samples null scores for the signature,
   pooled into one vector.
3. Fit the chosen `distribution` to the pooled null scores by maximum
   likelihood (`CBEA::estimate_distr()`, which calls
   `fitdistrplus::fitdist()` for `norm`).
4. With `correlation_adjustment` enabled, fit the same distribution to the
   observed scores of the signature across all samples, then combine the two
   fits (`CBEA::combine_distr()`): for `norm`, the final null keeps the mean
   of the permutation fit and takes the standard deviation from the fit to
   the observed scores. This widens the null to account for correlation among
   taxa, which the random subsets do not preserve. Because the observed fit
   needs several samples, do not run this protocol on a single sample or on a
   dataset too small to fit the distribution; record the number of samples
   per execution.
5. Convert each observed score to an upper-tail p-value under the final null,
   `p = 1 - F(score)` where `F` is the fitted cumulative distribution
   function (`CBEA::scale_scores()` with `method = "pval"`). A small p-value
   therefore means the signature is enriched, not depleted. Depletion is not
   tested by this protocol; if it is of interest, record that the lower tail
   was used instead, for every result.

In the `CBEA` package the call that performs Steps 4.1 through 4.5 is:

```r
set.seed(random_seed)
CBEA::cbea(obj = abundance, set = signatures, output = "pval",
           abund_values = "relative_abundance", distr = "norm",
           adj = TRUE, n_perm = 100)
```

where `abundance` is a `TreeSummarizedExperiment` whose named assay holds the
harmonized, strictly positive relative abundances, and `signatures` is the
`BiocSet` of retained signatures. The package documentation describes
`n_perm` as bootstrap resamples; the implementation draws random taxon
subsets as described above, and the protocol follows the implementation.

The null hypothesis is competitive: within a sample, the taxa in the signature
have the same balance against their complement as a random set of the same
size drawn from the whole measured universe `U`, signature members included,
has against its own complement. It is not a null that every
signature taxon has no association with the sample condition, and it is not a
test that every member of the signature is individually differentially
abundant.

Record the CBEA implementation and version, all parameters, the random seed,
the number of samples, the number of null scores per signature, the fitted
null parameters per signature, and any signature-sample result that could not
be computed.

### Step 5: Adjust across signatures and report the results

Within each execution (the pooled table or one per-dataset table), apply the
Benjamini-Hochberg procedure separately within each sample, taking the family
to be the unadjusted p-values of all retained signatures in that sample. Use
the same family definition for every sample and do not adjust only the
signatures that were significant before correction. Mark a signature-sample
result significant when its adjusted p-value is less than 0.05. Neither the
CBEA paper nor the BugSigDB application adjusts the sample-level p-values;
the adjustment is a requirement of this protocol so that a set of per-sample
calls has a declared false-discovery rate.

Produce a result table containing the dataset or pooled-scope identifier,
sample identifier, signature identifier, matched taxa, complement size, CBEA
score, unadjusted p-value, adjusted p-value, and significance decision. Also
produce an execution manifest containing the rank, universe, set-size bounds,
preprocessing description, aggregation and exclusion counts from Step 2,
inference parameters, number of samples, and software versions.

## Notes

The CBEA paper describes this method as a single-sample taxon-set enrichment
method based on the competitive null hypothesis and the isometric log-ratio
transformation. The BugSigDB paper's `Figure3.Rmd` supplies the application
context for running CBEA on pooled and individual microbiome datasets with
BugSigDB signatures, but it does not replace the method's primary citation,
and nothing in this protocol depends on the signatures coming from BugSigDB.

This protocol requires positive relative abundances because the balance uses
logarithms and because aggregation to `rank` sums rows. The choice of zero
replacement, pseudocount, normalization, and taxonomic database is upstream
and must be specified separately. The `0.5` added to percentage-scale
relative abundances in the BugSigDB vignette is one such choice, not a
prescription of this protocol; if it is used, record its scale and rationale
before running this protocol.

The vignette requested `output = "zscore"` from `CBEA::cbea()` and passed the
per-sample z-scores to a between-group linear model. That downstream analysis
is out of scope here, and this protocol requests `output = "pval"` so that the
sample-level inference is the deliverable. The z-score is the observed score
standardized by the final null's mean and standard deviation, so it is
available from the same fit if a downstream protocol needs it.

A significant result licenses only the conclusion that the signature's
within-sample balance is unusually high under the declared competitive null
and preprocessing. It does not establish causality, biological activity,
disease mechanism, differential abundance of every member, or generalization
beyond the analyzed samples.

This protocol does not produce the abundance input, retrieve or curate
signatures, compare CBEA with over-representation analysis or PADOG, perform
downstream between-group differential analysis of the CBEA scores, or
interpret a signature's biological meaning. Those operations require separate
protocols.

## History & Reviews
<!-- Newest versions at the top -->

### Version 1.0.0 (2026-10-17)

#### Changes
- Initial protocol creation.
- Specified taxonomy harmonization with aggregation by summing relative
  abundances, the background universe, set-size bounds, the competitive
  balance, the pooled cross-sample null and its correlation adjustment,
  upper-tail p-values, per-sample Benjamini-Hochberg adjustment, and pooled
  versus per-dataset execution.
- Generalized from BugSigDB signatures to any collection of microbial
  signatures, with BugSigDB retained as the worked example.

#### Reviews
*No reviews yet.*
