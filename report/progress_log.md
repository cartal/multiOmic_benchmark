Benchmarking methods for alignment of scRNA-sea and scATAC-seq data
================

## Dataset details

Incl. stats about dataset quality (median no. of fragments per cell,
perc. of fragments mapping to peaks, median no. of fragments in peaks
per cell)

## Data preprocessing

### ATAC-seq

ATAC-seq data is inherently sparse and noisy, more so than RNA-seq data.
In addition, methods for alignment with scRNA-seq data mostly require to
reduce accessibility signal to gene-level features. Different papers use
different strategies to preprocess ATAC-seq data and featurization
(benchmarked by Chen et al.
([2019](#ref-chenAssessmentComputationalMethods2019a))):

  - **Seurat pipeline**: raw 10x sequencing reads were preprocessed to
    call peaks with Cellranger (v …). Then all counts in peaks are
    summed up all counts in peaks within gene bodies + 2kb upstream
    (Welch et al. [2019](#ref-welchSingleCellMultiomicIntegration2019a);
    Stuart et al.
    [2019](#ref-stuartComprehensiveIntegrationSingleCell2019a)).
  - **SNAP-ATAC pipeline:** the genome is binned, then a cell x bin
    binary matrix is constructed. From this binary matrix a gene-level
    matrix is made (“Fast and Accurate Clustering of Single Cell
    Epigenomes Reveals Cis-Regulatory Elements in Rare Cell Types,”
    [n.d.](#ref-FastAccurateClusteringa)).
  - **PCA based on bulk:** some studies (Buenrostro et al.
    [2018](#ref-buenrostroIntegratedSingleCellAnalysis2018)) find
    principal components of differentiation/cell type from bulk ATAC-seq
    datasets and then project the single-cells on the same space.
    Arguably, this will lead to miss out on accessibility dynamics of
    rare cell populations.
  - **LSI method:** this procedure starts by making a bin x cell matrix.
    Then normalization and rescaling are done using term
    frequency-inverse document frequency transformation (something from
    text mining), then SVD is performed to obtain a PC x cell matrix.
    This is used to cluster cells and peaks are called on the clusters.
    Then the peak x cell matrix is used to do PCA again (I am getting
    dizzy).

### RNA-seq

Both datasets where preprocessed with std steps: removal of empty/lowQ
cells, normalization per cell coverage,

#### Feature selection

  - Highly-variable genes in RNA
  - genes that are expressed only in a few cells

## Tested integration methods

  - **Seurat CCA**:
      - select K?
  - **LIGER:**
      - K factors
      - Imputation strategy: projecting ATAC cells on NMF factor space.
        Testing if this is a valid imputation strategy using scRNA-seq
        data only
  - **SnapATAC pipeline:** it just wraps CCA alignment
  - **scGen:** requires cell type annotation also on the ATAC dataset
  - **totalVI:** not applicable as it assumes matching between cells
    (cite-Seq and RNA-seq from the same single-cells)
  - **BBKNN:** (how do you do an imputation step?)

## Uniform output for all methods

  - Impute transcriptomic data

## Metrics for comparison of integration models

Problems: finding optimal distance metric for gene accessibility and
expression (correlation doesn’t work, too sparse). (Or more general: how
to relate features from different datasets) Nearest neighbors in PCA?
How to make a nearest neighbor graph between modalities - MNN cosine
normalizes each batch and then calculates distances

  - PCA on integrated space and find genes with high eigen values
  - How to denoise the ATAC data??

<!-- end list -->

1)  Robustness to different methods of feature selection: HVGs in the
    RNA
2)  Robustness to different fractions of cells in ATAC dataset
3)  Leave-one-out approach for imputed data
4)  **Fraction of unassigned cells** (but how to distinguish unassigned
    and badly assigned?)
5)  **Joint clustering: purity of cell type annotation inside a cluster,
    mixing within the same cluster between different technologies**
6)  **Robustness to parameter picking (e.g. no. of factors)**
7)  Agreement meric defined by Welch et al. 2019: compare KNN graph of
    single datasets with KNN graph in integrated space, then calculate
    how many of each cell’s NNs in the single dataset graph are also NNs
    in the integrated graph. Welch et al. compare NN graphs built with
    different factor models to compare CCA and LIGER performance. I
    build for all methods KNN graphs from the PCA projection of imputed
    values.
8)  **Expression not at random of markers after integration:** is the
    structure that is clear in the RNA maintained after embedding w the
    ATAC? From idea of JP collaborator
      - Find marker genes from RNA only
      - Measure non-random expression in RNA only
9)  **pySCENIC on RNA only or integration w ATAC seq**

## Ideas for less “agnostic” integration

1)  Select only a certain lineage of cells (e.g. that you can align in
    pseudotime)
2)  Annotation of cell types also in ATAC-seq data (e.g. to use scGen)
3)  Considering enhancer accessibility (matching them to genes??)

## Does adding the ATAC information improve the inference of gene regulatory networks?

  - running SCENIC on full integrated data VS just on RNA

## Other random things

  - How does ATAC improve the RNA? Can we detect cell
    clusters/populations that are highly homogeneous in the RNA but
    display significant variability at the accessibility level? But what
    is variability in super noisy ATAC seq?
  - Studying pioneering TFs: temporal relationship between TF expression
    and accessibility (are they really opening chromatin?) Could be done
    on the time series data Svensson and Pachter
    ([2019](#ref-svenssonInterpretableFactorModels2019))

## Bibliography

<div id="refs" class="references">

<div id="ref-buenrostroIntegratedSingleCellAnalysis2018">

Buenrostro, Jason D., M. Ryan Corces, Caleb A. Lareau, Beijing Wu,
Alicia N. Schep, Martin J. Aryee, Ravindra Majeti, Howard Y. Chang, and
William J. Greenleaf. 2018. “Integrated Single-Cell Analysis Maps the
Continuous Regulatory Landscape of Human Hematopoietic Differentiation.”
*Cell* 173 (6): 1535–1548.e16.
<https://doi.org/10.1016/j.cell.2018.03.074>.

</div>

<div id="ref-chenAssessmentComputationalMethods2019a">

Chen, Huidong, Caleb Lareau, Tommaso Andreani, Michael E. Vinyard, Sara
P. Garcia, Kendell Clement, Miguel A. Andrade-Navarro, Jason D.
Buenrostro, and Luca Pinello. 2019. “Assessment of Computational Methods
for the Analysis of Single-Cell ATAC-Seq Data.” *bioRxiv*, August,
739011. <https://doi.org/10.1101/739011>.

</div>

<div id="ref-FastAccurateClusteringa">

“Fast and Accurate Clustering of Single Cell Epigenomes Reveals
Cis-Regulatory Elements in Rare Cell Types.” n.d., 41.

</div>

<div id="ref-stuartComprehensiveIntegrationSingleCell2019a">

Stuart, Tim, Andrew Butler, Paul Hoffman, Christoph Hafemeister,
Efthymia Papalexi, William M. Mauck, Yuhan Hao, Marlon Stoeckius, Peter
Smibert, and Rahul Satija. 2019. “Comprehensive Integration of
Single-Cell Data.” *Cell* 177 (7): 1888–1902.e21.
<https://doi.org/10.1016/j.cell.2019.05.031>.

</div>

<div id="ref-svenssonInterpretableFactorModels2019">

Svensson, Valentine, and Lior Pachter. 2019. “Interpretable Factor
Models of Single-Cell RNA-Seq via Variational Autoencoders.” Preprint.
Bioinformatics. <https://doi.org/10.1101/737601>.

</div>

<div id="ref-welchSingleCellMultiomicIntegration2019a">

Welch, Joshua D., Velina Kozareva, Ashley Ferreira, Charles Vanderburg,
Carly Martin, and Evan Z. Macosko. 2019. “Single-Cell Multi-Omic
Integration Compares and Contrasts Features of Brain Cell Identity.”
*Cell* 177 (7): 1873–1887.e17.
<https://doi.org/10.1016/j.cell.2019.05.006>.

</div>

</div>