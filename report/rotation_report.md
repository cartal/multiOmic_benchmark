---
title: "Optimizing integration of scRNA-seq and scATAC-seq datasets"
output: 
  bookdown::word_document2: 
    keep_md: yes
  bookdown::pdf_document2:
bibliography: references.bib
biblio-style: alphadin
link-citations: true
toc: false
---

# Introduction {-}

Single-cell Assays for Transposase-Accessible Chromatin by sequencing (scATAC-seq) enable profiling of the chromatin accessibility landscape of thousands of single-cells, and identification of *cis-* and *trans-* acting regulatory elements in heterogeneous cell populations [@cusanovichMultiplexSinglecellProfiling2015; @cusanovichSingleCellAtlasVivo2018]. However, common analysis tasks for single-cell RNA-seq (scRNA-seq) datasets, such as clustering and annotation of cell types, are challenging for scATAC-seq data, because of it's extreme sparsity and limited prior knowledge of cell-type specific accessibility patterns.
Integration of accessibility datasets with comprehensively annotated scRNA-seq datasets can allow denoising of biological signal, guide cell type annotation and disentangle causal relationships between biological layers of information [@buenrostroIntegratedSingleCellAnalysis2018; @granjaSinglecellMultiomicAnalysis2019]. 
Different methods for integration of multi-modal single-cell datasets have been recently proposed [@barkasJointAnalysisHeterogeneous2019; @lopezJointModelUnpaired2019; @stuartComprehensiveIntegrationSingleCell2019; @welchSingleCellMultiomicIntegration2019]. These require scATAC-seq data to be reduced to represent accessibility of genes, usually by simply counting the number of accessible regions overlapping with the promoter and gene body [ @stuartComprehensiveIntegrationSingleCell2019; @welchSingleCellMultiomicIntegration2019]. Then, assuming positive correlation between gene accessibility and expression, a common latent space projection is inferred, to find similarities between cells in different data modalities.

Currently there is no consensus on best practices for integration of scATAC-seq and scRNA-seq, regarding methods or preprocessing steps such as the featurization to genes. 
<!-- Methods for simultaneous profiling of biological layers are starting to emerge [refs], but these are mostly low-throughput and labor intensive. -->
<!-- In most cases, molecular profiles will be measured in parallel from cells sampled from the same tissue/cellular population. -->
<!-- Consequently, aligning different datasets requires an at least partial correspondence between profiled features across omics. This is true when integrating different scRNA-seq datasets or scRNA-seq data with spatial transcriptomics. For omic types that measure molecular features that are different than genes, data is usually preprocessed to generate a matrix of gene level features e.g. measuring gene activity from ATAC accessibility peaks [https://www.ncbi.nlm.nih.gov/pubmed/30078726]. Different integration methods use different inference algorithms for the latent space projection (e.g. canonical correlation analysis, non-negative matrix factorization, variational autoencoders), but all allow the mapping of a gene expression (or other molecular feature) vector x on a latent space vector z, via a vector of feature loadings w. Inspection of loadings can distinguish features that allow alignment across datasets and potentially indicate cell and omic specific contributions to the overall data variation. -->

Here we implemented an analysis workflow to optimize label transfer from scRNA-seq datasets to scATAC-seq datasets. We defined metrics to compare performance of three published integration methods (Seurat CCA anchoring, hereafter CCA [@stuartComprehensiveIntegrationSingleCell2019], Conos [@barkasJointAnalysisHeterogeneous2019] and Liger [@welchSingleCellMultiomicIntegration2019]). We then applied our workflow to optimize integration of scRNA-seq and scATAC-seq datasets generated from developing human thymus.

# Results {-}

We started by comparing performance of methods for label transfer on a publicly available dataset of Peripheral Blood Mononuclear Cells (PBMC) from 10XGenomics. 

We first run label prediction including a growing number of scATAC-seq cells to evaluate method run time and robustness. Conos is the fastest method, with speed minimally affected by the number of query cells (Fig.\@ref(fig:label-comp)A). Liger showed the highest variability in label predictions for subsets of cells (Fig.\@ref(fig:label-comp)B).   
A crucial step to reconstruct consensus chromatin profiles of cell types is to assess whether cells of the same predicted cell type are also similar in genome-wide accessibility. Visualizing the predicted labels on the embedding of genome-wide scATAC-seq profiles suggests that all methods reconstruct the expected clustering structure (Fig.\@ref(fig:label-comp)A). To quantify this agreement we formulate a KNN purity statistic that measures the fraction of neighbors that share the same label against a permutation based null (see Methods). On this dataset, we found that Conos and Liger score slightly higher than CCA (Fig.\@ref(fig:label-comp)G).

<!-- Cell type proportions are in line with those found in the cell population measured with scRNA-seq (Fig.\@ref(fig:label-comp)B). -->
<!-- All methods associate labels with a prediction score, that allows to remove labels with high uncertainty (Fig.\@ref(fig:label-comp)D). We found that different methods score with high confidence similar clusters (Fig.\@ref(fig:label-comp)E). We clustered scRNA-seq cells with increasing resolution (i.e. from bigger few clusters to many smaller clusters) and compared the distribution of prediction scores. We found that Conos predicts labels with higher confidence with larger (low-resolution) clusters (Fig.\@ref(fig:label-comp)F) -->
<!-- Setting a threshold on the label prediction score allows to remove from downstream analysis cells with a low confidence prediction label, and mark them as unassigned. We found that at the cutoff of 0.5, suggested by @stuartComprehensiveIntegrationSingleCell2019a, Liger excludes the least amount of cells, while Conos scores the most cells with higher confidence (Fig.\@ref(fig:predict-score)B).  -->
<!-- To reconstruct consensus chromatin profiles of cell types, it is crucial that label predictions in genome-wide accessibility. -->
<!-- We wanted to quantify if cells that get annotated with the same label also have a similar genome-wide bin accessibility profile. This is crucial to allow reconstruction of consensus chromatin accessibility profiles of predicted cell types. We devised a KNN purity statistic  -->
<!-- We calculate the fraction of nearest neighbors per cell that share the predicted label. For each prediction, we calculate a null distribution of this fraction by randomly shuffling the assigned labels. We define a KNN purity statistic as the deviation between the null and the true distribution (KS test, see Methods \@ref(knn-purity)).  -->
<!-- We found that Conos and Liger performed slightly better on this dataset than CCA (Fig.\@ref(fig:label-comp)G). -->
<!-- Looking at the distributions of KNN scores, we find that Liger maintains the original connectivity structure better, followed by Conos and finally CCA, even if the differences are not that substantial. Also the KNN score is dependent on the assigned cell type. -->
<!-- We checked accessibility of PBMC marker genes in the called cell types. We found that cell populations called with CCA show accessibility of expected marker genes from transcriptomics, for NK cells, monocytes, CD8+ cells, and the platelets even. -->
<!-- Because of the assumption of correlation between gene accessibility and expression,  -->
While all methods predict similar labels in a steady-state cell population such as the PBMCs, datasets capturing differentiating cell states, where the chromatin landscape undergoes extensive remodeling, are more challenging to integrate.
<!-- We next focused on optimization of label transfer on a biological system with differentiating cell states, where the chromatin landscape might be undergoing remodeling. -->
We analyzed scRNA-seq and scATAC-seq datasets generated from one fetal thymus at 10 weeks post-conception.
We identified 15 cell populations by clustering the transcriptomes, including components of the thymic microenvironment and a main cluster representing T cells undergoing maturation, from the Double Negative (DN) state, to Double Positive (DP) to CD4+ or CD8+ Single Positive (SP) T cells (Fig.\@ref(fig:opt-thymus)A). Visualizing the scATAC-seq cells we also find a large central cluster and smaller cell populations (Fig.\@ref(fig:opt-thymus)B).
Running our label transfer workflow we noticed that, while all methods consistently assigned the smaller clusters, only with CCA transferred annotations for the T cells resembled chromatin accessibility clusters, as also quantified with the KNN purity score (Fig.\@ref(fig:opt-thymus)C). We observed that the transformation from genome-wide to gene accessibility significantly increases noise in the T cell cluster (Fig.\@ref(fig:opt-thymus)D). 
<!-- To obtain gene accessibility profiles that best represents the signal in the genome-wide profiles, we binarized gene accessibility (see Methods) and  -->
We reasoned that summing up sparse accessibility profiles over gene bodies might lead to artifactual inflation of differences between cells that are similar genome-wide.
We found that binarizing the gene accessibility profiles allows to maintain the clustering structure found in genome-wide accessibility (Fig.\@ref(fig:opt-thymus)F) and that using the binary gene matrix for label transfer greatly increased the KNN purity for all methods (Fig.\@ref(fig:opt-thymus)E). 

Our comparison of label transfer outcomes indicated that integration with a binary cell-by-gene accessibility matrix and using CCA is the best integration strategy, by our metrics. 
Using these indications, we performed integration of scATAC-seq and scRNA-seq cells focusing on the putative T cell populations in the thymic datasets (Fig.\@ref(fig:tcells)A-B). We modeled the differentiation trajectory of T cells using pseudotime analysis (Fig.\@ref(fig:tcells)C). This ordered cell populations in both datasets according to the expected order for conventional T cell maturation (Fig.\@ref(fig:tcells)D). We were then able to explore changes in accessibility patterns along the inferred differentiation trajectory. We observed that genome-wide accessibility tends to decrease during T cell maturation (Fig.\@ref(fig:tcells)E), as expected for cells moving towards a "primed" state [@gaspar-maiaOpenChromatinPluripotency2011]. We then performed motif accessibility analysis with ChromVAR [@schepChromVARInferringTranscriptionfactorassociated2017], which measures enrichment or depletion of accessibility at transcription factor (TF) motifs in scATAC-seq profiles. Among the top variable in our dataset, we identify motifs for many TFs that have been shown to be involved in T cell maturation (Fig.\@ref(fig:tcells)), such as TCF, RUNX, REL and TBX transcription factors [@hosoyaGlobalDynamicsStagespecific2018, Park et al. 2019, unpublished]. These show distinct changes in accessibility along pseudotime, especially at the very early and final stages of maturation. Data integration enabled us to compare patterns of TF motif accessibility with patterns of TF expression along pseudotime. We identified instances in which TF expression correlates with motif accessibility, as well as cases in which motif accessibility decreases as TF expression increases (\@ref(fig:tcells)G).

# Discussion {-}
We have developed the first uniform framework to compare performance of different methods for label transfer. This could lay the ground for a more systematic benchmark of integration methods across datasets and modalities. Ideally this could use recently published multi-omic datasets generated from high-throughput joint profiling of scRNA-seq and scATAC-seq in the same cells [@chenHighthroughputSequencingTranscriptome2019], as a ground-truth for correlations between modalities.
<!-- We found that while different methods perform similary well on a dataset with well defined cluster structure, performance drops for alignment of a dataset representing developing cells. This is expected, to some extent, since all methods are built on the assumption that gene expression correlates with gene accessibility: this might not be true for cells undergoing a differentiation process, where the chromatin landscape is subject to remodelling [ref]. -->

We show that the strategy used for aggregation of accessibility profiles at the gene level can significantly impact performance of integration methods. 
Counting accessible regions that overlap the gene body might be an adequate solution for datasets with strongly distinct accessibility patterns between cell populations. More refined models estimate accessibility of sites linked to specific genes also accounting for action of distal regulatory elements [@plinerCiceroPredictsCisRegulatory2018]. Our framework could be used to compare integration performance using similar models for featurization to genes. 

We have demonstrated how optimized integration allows to align multi-omic datasets along a common differentiation trajectory, such as for T cell maturation. This allows to relate patterns of accessibility to patterns of gene expression along maturation. Interestingly, we observed instances of TFs that showed coordinated, but anti-correlated, changes between expression and motif accessibility. This can indicate a repressive function of the TF on regulatory elements [@berestQuantificationDifferentialTranscription2019].

# Methods {-}

### Datasets {-}
**PMBC:** this dataset is composed of Peripheral Blood Mononuclear Cells (PBMCs) from one donor. Raw scRNA-seq counts were downloaded with the Seurat R package [[@satijaSpatialReconstructionSinglecell2015]] (download link: https://www.dropbox.com/s/3f3p5nxrn5b3y4y/pbmc_10k_v3.rds?dl=1). Raw scATAC-seq fragments were downloaded with the SnapATAC R package [@fangFastAccurateClustering2019] (download link: http://renlab.sdsc.edu/r3fang//share/github/PBMC_ATAC_RNA/atac_pbmc_10k_nextgem.snap). A total of 5607 scRNA-seq cells and 8690 scATAC-seq cells were used for analysis after QC. 

**Thymus dataset:** this dataset consists of cells from one fetal thymus at 10 post-conception weeks (Park et al. unpublished, Dominguez Conde et al. unpublished). A total of 8321 scRNA-seq cells and 5793 scATAC-seq cells were used for analysis after QC.

### Data preprocessing and dimensionality reduction {-}

We preprocessed and normalized scRNA-seq using the standard pipeline from the R package Seurat (v3.1.1). We clustered cells using the leiden algorithm [@traagLouvainLeidenGuaranteeing2019] and annotated populations based on expression of marker genes from the literature. ScATAC-seq reads were aligned and preprocessed using CellRanger (10X genomics). We used the SnapATAC pipeline for quality control and preprocessing [@fangFastAccurateClustering2019]. 
This generates genome-wide single-cell accessibility profiles by binning the genome into fixed-size windows (selected bin size: 5 kb) and constructing a cell-by-bin binary matrix, estimating accessibility for each bin.

For dimensionality reduction, we used Principal Component Analysis for scRNA-seq datasets and Latent Semantic Indexing for dimensionality reduction of accessibility matrices, as proposed by @cusanovichMultiplexSinglecellProfiling2015. For data visualization, we used Uniform Manifold Approximation and Projection (UMAP) algorithm [@mcinnesUMAPUniformManifold2018] on low-dimensional data representation.
<!-- ells with high expression of mitochondrial genes were filtered out. Raw counts $c$ were normalized by total cell coverage and converted to $log(c + 1)$ as a variance stabilizing transformation. -->

### Label transfer {-}

**Accessibility in gene-level features:** We generated cell-by-gene count matrices by aggregating bin coverage over the gene bodies and promoters (2kb upstream of transcriptional start site). Count gene matrices were converted to binary gene matrices by substituting counts with 1 if counts > 0.

**Feature selection:** unless otherwise specified, we select genes for label transfer by taking the union of the most highly variable genes in scRNA-seq (using the Seurat function `FindVariableGenes`, with `selection.method = "mvp"`) and the most frequently covered genes in the scATAC-seq datasets (covered in at least 10% of cells). We found that integration using joint feature selection performs significantly better that selection based on the reference dataset only (data not shown).
<!-- Other studies have used only the HVGs from the reference dataset (in most cases the scRNA-seq). We found that these two feature selection strategies gave similar label transfer results and we opted for the union to avoid selecting only genes that have very low coverage in the ATAC-seq data.  -->

**Data integration methods:** details of published integration methods for single-cell data and the rationale behind exclusion from the benchmark are detailed in Table 1.   

<!-- ### Label transfer {#transfer-label} -->
<!-- One of the key tasks for integration methods is to be able to transfer cell type annotations learnt from a reference to a query dataset. This is especially useful if the query is a scATAC-seq dataset, where calling of cell types based on prior knowledge on marker genes is often not possible. Different models are adapted to transfer discrete cell state labels derived from gene expression to cells measured with scATAC-seq. -->

<!-- ##### Seurat CCA -->

<!-- Identified anchor pairs are weighted based on the query cell local neighboorhood (the k nearest anchors) and the anchor score. The obtained reference cells x query cells weight matrix is then multiplied by the matrix of annotation x reference cells, to generate a query cell x annotation matrix. This returns a prediction score for each class for every cell in the query dataset, ranging from 0 to 1 [@stuartComprehensiveIntegrationSingleCell2019a]. -->

<!-- ##### LIGER -->
<!-- While the authors do not describe a method for transferring discrete labels, I adapted their strategy for feature imputation. I build a cross-dataset KNN graph in the aligned factor space, then I assign each query cell to the most abundant label between the k nearest neighbors in the reference dataset (k=30). The prediction score for label $l$ is computed as the fraction of nearest neighbors that have the predicted label. -->
<!-- $$ -->
<!-- score = \frac{count(l)}{k} -->
<!-- $$ -->

<!-- ##### Conos -->
<!-- Label transfer is treated as a general problem of information propagation between vertices of the common graph (detailed in @barkasJointAnalysisHeterogeneous2019). The label score is the label probability updating during the diffusion process. -->

### KNN purity score {-}
We construct the k-nearest-neighbor (KNN) graph of scATAC-seq cells on the LSI reduction (k = 30). For each prediction, we measure the fraction of NNs per cell with the same predicted label (retaining only labels with prediction score > 0.5). We do the same after randomly permuting the predicted labels, to estimate a null distribution. 
<!-- The purpose of this step is to avoid giving a high score to a prediction that assigns many cells to just a few clusters.  -->
We then compute the Kolmogorov-Smirnov deviation statistic between true and null distribution and define this as KNN purity.

\newpage 

| Method       | Reference | Model for embedding | Label/feature propagation | Reason for Excluding |
| ---------------|-------------------|:--------------------------|:-----------------------|:----------------------------|
| Seurat CCA   | @stuartComprehensiveIntegrationSingleCell2019 | Canonical Correlation analysis | mNN pairing | / |
| LIGER | @welchSingleCellMultiomicIntegration2019 | Joint Non-Negative Matrix factorization | KNN graph | / |
| Conos | @barkasJointAnalysisHeterogeneous2019  | Joint PCA | Inter/Intra-dataset edges | / |
| scGen | @lotfollahiScGenPredictsSinglecell2019 | Variational Autoencoder | Decoder | Requires cell type annotation in both datasets |
| totalVI | @gayosoJointModelRNA2019 | Variational inference | Generative model | Requires  multi-omic data from the same single-cells |
| BBKNN | @polanskiBBKNNFastBatch | PCA | Batch balanced graph construction | Bad alignment during testing |
| gimVI | @lopezJointModelUnpaired2019 | Variational inference | Generative model | No implementation for right log-likelihood distribution |
Table: Details of published data integration methods considered for comparison.


### Analysis of T cell maturation {-}
We selected putative T cell clusters based on independent clustering of scATAC-seq and scRNA-seq. We performed co-embedding and label transfer using the Seurat CCA method [@stuartComprehensiveIntegrationSingleCell2019], with preprocessing as previously described. We imputed gene expression values for scATAC-seq cells and computed diffusion pseudotime [@haghverdiDiffusionPseudotimeRobustly2016] on all cells as implemented in Scanpy (v1.4.4). The cell of origin for the pseudotime algorithm was selected by expression of markers of early DN cells (IGLL1, CD34). [@wolfSCANPYLargescaleSinglecell2018]. For analysis of TF motif enrichment, we used the ChromVAR package [@schepChromVARInferringTranscriptionfactorassociated2017], with default settings.  

### Code availability {-}
All code for this analysis can be accessed at https://github.com/EmmaDann/multiOmic_benchmark.

![(\#fig:label-comp)(ref:label-comp-fig)](output/Fig1_2.pdf)

(ref:label-comp-fig) **Comparison of integration methods performance on PBMC dataset:** (A-B) Benchmark for integration with increasing size of scATAC-seq dataset (query cells), each point represents a label transfer run, color indicate the used method (2 runs per method); (A) comparison of run time; (B) similarity of predicted ID, compared to integration with full scATAC-seq dataset, measured by adjusted rand index; (C) UMAP visualization of scATAC-seq cells (as in B), colored by label transfer outcomes integrating on gene accessibility counts; right: quantification of KNN purity

![(\#fig:opt-thymus)(ref:opt-thymus-fig)](output/Fig2_2.pdf){width=80%}

(ref:opt-thymus-fig) **Optimization of label transfer on developing thymus dataset:** (A) UMAP visualization of scRNA-seq cells in the thymus colored by cell type (DC: dendritic cells, DN: double negative T cells, DP (P): Proliferative double positive T cells, DP (Q): quiescent double positive T cells, Ery: eruthrocytes, Fib: fibroblasts, ILC3: innate lymphoid cell types, Mac: macrophages, NK: natural killer cells, SP: single positive T cells, TEC: thymic epithelial cells); (B) UMAP visualization of scATAC-seq cells in the thymus (genome-wide accessibility), colored by clusters identified using the leiden algorithm; (C) left: UMAP visualization of scATAC-seq cells (as in B), colored by label transfer outcomes; (D) quantification of KNN purity

![(\#fig:tcells)(ref:tcells-fig)](output/Fig3.pdf){width=80%}

(ref:tcells-fig) **Integrative analysis of T cell maturation in developing thymus:** Joint visualization of scRNA-seq and scATAC-seq cells for the T cell clusters, colored by (A) technology, (B) inferred cell type labels and (C) diffusion pseudotime; (D) Distribution of cells along 100 equal-sized pseudotime rank bins, colored by inferred cell type; (E) Distribution of the fraction of accessible genomic bins per cell for each pseudotime rank bin; (F) Heatmap of TF motif deviation for the top 50 most variable TF motifs inferred by chromVAR. Cells are ordered by pseudotime. Values are smoothed with a running average function (step = 30); (G) Smoothed values for log-normalized gene expression counts (top, pink) and TF motif deviation (bottom, cyan) along pseudotime ranks. Values are smoothed with the LOESS function (span = 0.2).

# Bibliography {-}

