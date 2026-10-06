---
name: "ward-linkage-signature-matrix-clustering"
description: "Cluster and display a signature similarity matrix using Ward minimum-variance hierarchical linkage."
version: "1.0.0"
authors:
  - name: "Otto Infield-Harm"
    orcid: "0009-0009-8629-6124"
  - name: "Levi Waldron"
    orcid: "0000-0003-2725-0694"
date: "2026-10-06"
status: draft
type: "atomic"

artifact_doi: ~
collection_doi: "10.5281/zenodo.22731694"
protocol_citation: "10.1080/01621459.1963.10500845"
method_origin_citation: "10.1080/01621459.1963.10500845"
license: "CC-BY-4.0"
protocols_used: []
category: "Statistical Analysis"
tags: [clustering, hierarchical-clustering, heatmap, dendrogram, similarity-matrix]
---

# Ward-Linkage Clustering of a Signature Similarity Matrix

This protocol orders and displays a signature-by-signature similarity matrix
using agglomerative hierarchical clustering with Ward minimum-variance
linkage. It is independent of how the similarities were calculated, so the
input may come from any similarity protocol that produces the required matrix
layout.

## Materials

- **Similarity matrix**
  - A square numeric matrix with one row and one column for every signature.
  - The row and column identifiers must be unique and must occur in the same
    order.
  - The matrix must be symmetric, with a documented diagonal and a documented
    interpretation of larger values.
- **Ward-compatible distances**
  - A distance representation for exactly the same signatures and ordering.
  - Distances must be non-negative, symmetric, have zero diagonal, and be
    appropriate for Ward's minimum-variance criterion. In particular, they
    must be Euclidean-compatible.
  - Any similarity-to-distance conversion or other repair of the input is
    completed and documented before this protocol begins; this protocol does
    not prescribe that preprocessing.
- **Signature annotations**
  - Study, condition, body site, and signature length for every signature when
    those metadata are available.
  - Annotation values must be joined by signature identifier, not by matrix
    position inferred from a separate ordering.

## Steps

### Step 1: Verify and record the inputs

Confirm that the similarity matrix and distance representation contain the
same signatures in the same order. Verify the matrix properties and the
distance prerequisites before clustering. Reject the input and report the
failed property if an identifier is duplicated or missing, the two
representations do not align, or the distance representation is not
Ward-compatible. Do not silently reorder, impute, symmetrize, or transform
the inputs.

Record the source and version of the similarity matrix, the distance
representation and its construction, the signature ordering, the number of
signatures, and the checks performed.

### Step 2: Perform Ward hierarchical clustering

Start with each signature as its own cluster and repeatedly merge the pair of
clusters that gives the smallest increase in within-cluster sum of squares
under Ward linkage. Continue until all signatures form one hierarchy.

Use a deterministic tie rule and record it when two candidate merges have the
same objective increase. Record the linkage method, distance representation,
tie rule, merge sequence or dendrogram, and any software and version used.

The dendrogram represents the sequence and relative objective cost of the
merges. A branch groups signatures according to the supplied distances; it
does not establish a biological taxon, causal relationship, statistical
significance, or condition enrichment.

### Step 3: Choose an ordering and display the matrix

Use the leaf order of the Ward dendrogram to order both rows and columns of
the original similarity matrix. Apply the identical order to every annotation
track. Do not replace the similarity values with distances or reorder rows
and columns independently.

Display the ordered similarity matrix with the dendrogram and annotation
tracks for the available study, condition, body-site, and signature-length
metadata. Include a legend or scale identifying the meaning and range of the
similarity values.

If discrete cluster labels are required, choose the number of clusters or
cutting rule before interpreting the annotations, apply it to the recorded
dendrogram, and report the rule and resulting membership. A dendrogram and
heatmap do not require a fixed number of clusters.

## Output

Produce:

- the original similarity matrix with rows and columns in the recorded Ward
  leaf order;
- the Ward dendrogram and its merge heights or objective increases;
- annotation tracks joined to the same ordered signature identifiers; and
- the recorded input checks, clustering parameters, tie rule, and any
  discrete cluster-cut rule.

The similarity values must remain unchanged apart from their row and column
ordering.


## Notes

Ward's paper introduced hierarchical grouping by minimizing an objective
function; this protocol specifies its minimum-variance linkage form. Ward
linkage is therefore a deliberate choice here, not an implementation default.
The Euclidean-compatibility requirement belongs to the clustering method, not
to any particular similarity measure.

## Out of scope

This protocol does not calculate similarity, harmonize taxa, convert
similarities to distances, repair invalid matrices, test clusters for
condition enrichment, or interpret biological associations. Those operations
must be specified separately.

## History & Reviews
<!-- Newest versions at the top -->

### Version 1.0.0 (2026-10-06)

#### Changes
- Initial protocol creation.

#### Reviews
*No reviews yet.*
