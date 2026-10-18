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

This protocol converts a signature-by-signature similarity matrix to
distances, clusters the signatures by agglomerative hierarchical clustering
with Ward minimum-variance linkage, and displays the similarity matrix in the
resulting order. It is independent of how the similarities were calculated,
so the input may come from any similarity protocol that produces the required
matrix layout, such as `jaccard-signature-similarity`.

## Materials

- **Similarity matrix**
  - A square numeric matrix with one row and one column for every signature.
  - The row and column identifiers must be unique and must occur in the same
    order.
  - The matrix must be symmetric, with every value between 0 and 1
    inclusive, a diagonal of 1, and larger values meaning more similar.
    Jaccard similarity satisfies these conditions.
  - Every entry must be present. A matrix with missing entries, for example
    one produced with pair restrictions, cannot be clustered by this protocol.
- **Parameters**
  - `tie_rule`: when two or more candidate merges have exactly the same
    objective increase, merge the pair whose members have the smallest
    original input indices, comparing the smaller index of each pair first
    and then the larger. This is the default and the only rule defined here.
- **Signature annotations**
  - Study, condition, body site, and signature length for every signature when
    those metadata are available.
  - Annotation values must be joined by signature identifier, not by matrix
    position inferred from a separate ordering.

## Steps

### Step 1: Verify the input and convert similarity to distance

Verify the matrix properties before clustering. Reject the input and report
the failed property if an identifier is duplicated or missing, the matrix is
not square or symmetric, any entry is missing or outside 0 to 1, or the
diagonal is not 1. Do not silently reorder, impute, symmetrize, or transform
the input.

Convert the similarity matrix to a distance matrix by subtracting every
entry from 1:

```text
d(i, j) = 1 - s(i, j)
```

The result is non-negative, symmetric, and zero on the diagonal. Use it only
for clustering; the similarity values are what is displayed in Step 3.

Record the source and version of the similarity matrix, the signature
ordering, the number of signatures, and the checks performed.

### Step 2: Perform Ward hierarchical clustering

Start with each signature as its own cluster and repeatedly merge the pair of
clusters whose merge gives the smallest increase in the within-cluster sum of
squared distances. Continue until all signatures form one hierarchy.

Apply Ward's criterion to the squared distances. In the Lance-Williams
update, after merging clusters `A` and `B`, the squared distance from the
merged cluster to any other cluster `C` is:

```text
d²(A∪B, C) = ((|A| + |C|) d²(A, C) + (|B| + |C|) d²(B, C) - |C| d²(A, B))
             / (|A| + |B| + |C|)
```

where `|X|` is the number of signatures in cluster `X`. The merge height
reported for each step is the distance, not the squared distance. In R this
is `hclust(as.dist(d), method = "ward.D2")`. Do not use `method = "ward.D"`,
which applies the same update to unsquared distances and produces a
different hierarchy from the same input.

Break ties with `tie_rule`. Record the linkage method, the tie rule, whether
any tie occurred, the merge sequence with its heights, the leaf order of the
resulting dendrogram, and the software and version used.

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
function; this protocol specifies its minimum-variance linkage form applied
to squared distances, which is the criterion Ward described. Ward linkage is
therefore a deliberate choice here, not an implementation default. The
source analyses applied it to one minus Jaccard similarity, which is a
metric but not in general a Euclidean distance; the criterion is applied to
it regardless, as a documented choice.

The leaf order of a dendrogram is not unique: either child of a merge can be
drawn first without changing the hierarchy, and implementations differ in
the convention they use. This protocol does not prescribe one. It requires
the leaf order to be recorded so that the displayed matrix can be
reproduced.

## Out of scope

This protocol does not calculate similarity, harmonize taxa, repair invalid
matrices, test clusters for condition enrichment, or interpret biological
associations. Those operations
must be specified separately.

## History & Reviews
<!-- Newest versions at the top -->

### Version 1.0.0 (2026-10-06)

#### Changes
- Initial protocol creation.

#### Reviews
*No reviews yet.*
