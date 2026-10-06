---
name: jaccard-signature-similarity
description: Calculate pairwise Jaccard similarity between microbial signatures after harmonizing their taxa to one specified taxonomic rank.
version: 1.0.0
authors:
  - name: Otto Infield-Harm
    orcid: 0009-0009-8629-6124
  - name: Levi Waldron
    orcid: 0000-0003-2725-0694
date: 2026-10-06
status: draft

license: CC-BY-4.0
type: atomic

protocol_citation: "10.1111/j.1469-8137.1912.tb05611.x"
method_origin_citation: "10.1111/j.1469-8137.1912.tb05611.x"

artifact_doi: ~
collection_doi: "10.5281/zenodo.22731694"

upstream_repositories:
  - "https://github.com/waldronlab/bugSigSimple"
  - "https://github.com/waldronlab/BugSigDBPaper"
database_urls: []

protocols_used: []
key_packages: []
category: "Statistical Analysis"
tags: [jaccard, similarity, signatures, taxonomic-level]
---

# Jaccard Similarity Between Microbial Signatures

This protocol produces a signature-by-signature matrix of Jaccard similarities
between microbial taxon sets. It first harmonizes every signature to one
prespecified taxonomic rank, then calculates intersection over union for every
eligible pair.

The protocol uses "signature" for one set of taxa reported in one direction for
one study or experiment. It treats taxa as set members: repeated occurrences of
the same taxon within a signature do not increase its weight.

## Materials

- **Input signatures**
  - A table or equivalent collection with one unique identifier per signature and
    a set of reported taxa for each signature.
  - Study identifiers, experiment identifiers, and direction labels for every
    signature. Direction must distinguish at least `increased` and `decreased`
    when those labels are present.
  - Taxonomic identifiers and a pinned taxonomy mapping capable of converting
    every input taxon to the selected rank.
- **Parameters**
  - `taxonomic_rank`: exactly one rank at which every signature is compared. The
    default is `genus`, matching the worked Jaccard analysis in the source
    material. Select another rank before harmonization when scientifically
    justified.
  - `minimum_signature_length`: the minimum number of distinct taxa a signature
    must contain after harmonization. The default is `2`.
  - `direction_pairing`: compare all eligible signatures regardless of direction
    by default. Alternatively, select only same-direction or only opposite-
    direction pairs before calculating the matrix.
  - `same_study_pairs`: include pairs from the same study by default. Excluding
    them is an alternative that must be selected and recorded before calculation.

## Steps

### Step 1: Define the comparison set

Specify the source data version, the taxonomy version and mapping, the selected
`taxonomic_rank`, `minimum_signature_length`, `direction_pairing`, and
`same_study_pairs` before inspecting similarity results. Record every parameter.

Require one unique identifier per input signature. Preserve the study,
experiment, direction, and original taxon identifiers as metadata; those fields
are needed to interpret matrix entries but are not part of the Jaccard
calculation.

Select the signatures to compare using rules defined before calculation. Do not
select, remove, or pair signatures based on their eventual similarity values.

### Step 2: Harmonize taxa to one rank

Map every taxon in every selected signature to the selected `taxonomic_rank`
using the pinned taxonomy mapping. Record the mapping version, input identifier,
mapped identifier, and any taxon that cannot be mapped.

Replace each signature's taxon list with the set of mapped identifiers. Remove
duplicate mapped identifiers within a signature. This deduplication is required:
Jaccard similarity compares sets, not counts or abundances.

Do not silently mix taxonomic ranks. A taxon that cannot be mapped to the
selected rank is excluded from that signature's harmonized set and recorded as
unmatched. If a signature has no mapped taxa, mark it empty rather than treating
it as a valid zero-length signature.

Rank harmonization discards information about the original taxonomic rank,
lineage detail below the selected rank, and any abundance or multiplicity
attached to the original record. Retain those original fields in provenance,
but do not use them in the Jaccard calculation.

### Step 3: Apply the minimum-length rule

Count the distinct mapped taxa in each harmonized signature. Retain a signature
only when its count is at least `minimum_signature_length`. With the default of
`2`, remove signatures with zero or one distinct mapped taxa, including
signatures emptied by harmonization.

Record every removed signature and the reason: empty after mapping, below the
minimum length, or another prespecified input exclusion. Do not calculate a
similarity involving an ineligible signature. If fewer than two signatures
remain, output an empty matrix with its dimensions and reason rather than
inventing pairwise values.

### Step 4: Select eligible pairs

Construct pairs from the retained signatures according to the selected
`direction_pairing` and `same_study_pairs` rules. The default compares every
unordered pair of retained signatures, including increased with decreased
signatures and signatures from the same study.

If same-direction or opposite-direction pairing is selected, define direction
labels that do not match either selected class and record how they are handled.
If same-study pairs are excluded, compare study identifiers after resolving
them to the pinned study identity; do not infer study identity from a display
label.

Pair restrictions affect off-diagonal entries only. Every retained signature
still receives a diagonal entry of `1.0`.

### Step 5: Calculate Jaccard similarity

For two retained signatures with harmonized taxon sets `A` and `B`, calculate:

```text
J(A, B) = |A ∩ B| / |A ∪ B|
```

Because eligible signatures contain at least two mapped taxa, the union is
non-empty and the denominator is positive. Report values between `0` and `1`.
Identical sets have similarity `1.0`; disjoint sets have similarity `0.0`.

Calculate one value for every permitted unordered pair and copy it to both
corresponding matrix positions. Do not calculate a value for a prohibited pair
and then interpret it as zero; represent prohibited pairs according to the
matrix convention selected in Step 6.

### Step 6: Produce the similarity matrix

Output a square numeric matrix with one row and one column for every retained
signature. Use the same ordered signature identifiers for rows and columns,
record that ordering, set the diagonal to `1.0`, and make permitted entries
symmetric. The matrix must contain no duplicate row or column identifiers.

For pairs excluded by `direction_pairing` or `same_study_pairs`, use missing
values and provide a pair-level reason rather than zero. If the downstream
consumer requires a complete matrix, do not silently substitute a value;
instead, report that the selected pair restrictions are incompatible with that
consumer's requirement.

Also output a pair metadata table containing the two signature identifiers,
study and direction metadata, whether the pair was permitted, intersection
size, union size, similarity, and any exclusion reason. Preserve the
harmonized taxon sets and the signature-exclusion table as provenance.

## Notes

The 1912 Jaccard article, *The Distribution of the Flora in the Alpine Zone*,
is the DOI-indexed English-language primary source used here
([10.1111/j.1469-8137.1912.tb05611.x](https://doi.org/10.1111/j.1469-8137.1912.tb05611.x)).
Jaccard also published an earlier 1901 French work associated with the
coefficient; the 1912 article is retained because it is the citable source
identified for the coefficient in the repository issue and has a verified DOI.

The source analyses use rank harmonization and a minimum signature length of
two taxa. Those choices are defaults here, not properties of the Jaccard
coefficient itself. Taxonomic harmonization is a lossy preprocessing decision,
so results at different ranks are not interchangeable.

Internal practical evaluation used small, duplicate, mixed-rank,
empty-after-harmonization, direction-restricted, same-study, diagonal, symmetry,
and large sparse cases. A fresh executor calculated all supplied concrete
similarities correctly. Five cases lacked a pinned taxonomy mapping version and
one large sparse case lacked actual taxon memberships; these were evaluation-case
defects, not protocol failures. The test data and expected results are not
included in this protocol.

## Out of scope

This protocol only calculates Jaccard similarity after single-rank harmonization.
It does not define taxonomy assignment, mixed-rank semantic similarity,
clustering, distance conversion, visualization, ordination, or statistical
testing.

## History & Reviews
<!-- Newest versions at the top -->

### Version 1.0.0 (2026-10-06)

#### Changes
- Initial protocol creation.

#### Reviews
*No reviews yet.*
