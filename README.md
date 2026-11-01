# Differential expression in triple-negative breast cancer: TCGA-BRCA, replicated in METABRIC

I compared triple-negative breast cancer (TNBC) with all other breast cancers in two independent cohorts to see which genes and pathways differ, and whether those differences hold up in a second dataset. Mostly, this confirmed known biology: estrogen-response genes were lower in TNBC and cell-cycle genes were higher, and both replicated. Some results did not replicate or I could not interpret, and I report those below.

## The question

Which genes and pathways differ between TNBC and other breast cancers in TCGA-BRCA, and do those differences replicate in METABRIC?

## Data

- **TCGA-BRCA** (discovery; Cancer Genome Atlas Network, 2012): clinical data from the cBioPortal PanCancer Atlas (Cerami et al., 2012; Gao et al., 2013; 1,084 patients), receptor status (ER, PR, HER2) from the original TCGA breast study on cBioPortal, and gene-level RNA-seq counts (GDC STAR counts) from UCSC Xena (Goldman et al., 2020).
- **METABRIC** (replication; Curtis et al., 2012): clinical and Illumina microarray expression data from cBioPortal (2,509 patients).

## How I defined TNBC

I wrote the rule before looking at any expression data. TNBC means ER, PR and HER2 all negative (Foulkes et al., 2010). Non-TNBC means at least one of the three is positive. Patients with missing or unresolved receptor status were excluded rather than guessed. In TCGA, equivocal HER2 by IHC was settled with FISH where available. Male patients were excluded.

| Cohort | TNBC | Non-TNBC | Excluded |
|---|---|---|---|
| TCGA (clinical file, 1,084 patients) | 142 | 840 | 102 (12 male, 1 unmatched, 89 missing or unresolved status) |
| TCGA (with primary-tumour expression, used in the analysis) | 141 | 839 | - |
| METABRIC (2,509 patients) | 320 | 1,979 | 210 |
| METABRIC (with expression, used in the analysis) | 320 | 1,660 | - |

As checks on the groups: in TCGA, 104 of the 126 TNBC patients with a PAM50 label (Parker et al., 2009) were Basal-like (83%). In METABRIC, 246 of the 266 TNBC patients with a three-gene classifier label were ER-/HER2-. About a third of METABRIC's TNBC group (119 of 320) were claudin-low (Prat et al., 2010), a category that TCGA's PAM50 labels do not have, so the two TNBC groups are probably not identical in composition.

## Methods

**TCGA.** Xena stores the counts as log2(count + 1). I reversed this and confirmed the result was whole numbers. I kept one primary tumour sample per patient (the first by barcode order when a patient had more than one). I kept genes with at least 10 counts in at least 141 samples (the size of the smaller group), which left 24,684 of 60,660 genes. I ran differential expression with PyDESeq2 version 0.5.4 (Muzellec et al., 2023), a Python implementation of DESeq2 (Love et al., 2014). The design formula was `~group`, which means each gene's counts are modelled by TNBC status. The contrast was TNBC versus non-TNBC, with Benjamini-Hochberg correction (Benjamini & Hochberg, 1995). Fold changes are not shrunk. Ensembl IDs were mapped to gene symbols with the GENCODE v36 annotation file distributed with Xena (Frankish et al., 2021), and I checked the mapping on ESR1 and GATA3.

**METABRIC.** The expression data are log2 microarray intensities. I averaged 219 duplicated gene symbols, then compared the groups gene by gene with Welch's t-test and Benjamini-Hochberg correction. The effect size is the difference in mean log2 intensity. This is not the full limma method, which borrows information across genes. With about 2,000 samples I expect the difference to be small, but I did not test that.

**Replication.** Because the platforms differ, I compared effect sizes and direction, not p-values. I matched genes by symbol and used the genes measured in both cohorts.

**Pathways.** I tested the Hallmark gene sets (Liberzon et al., 2015) for over-representation among genes with adjusted p < 0.05 and |log2 fold change| > 1. I used GSEApy (Fang et al., 2023) with its offline test, which takes the gene-set library from Enrichr (Kuleshov et al., 2016) and accepts a custom background: the genes I actually tested (24,659 in TCGA, 16,570 in the replication step), not the whole genome.

## Results

### TCGA

19,221 of 24,684 genes had an adjusted p-value below 0.05. With 980 patients, almost any difference reaches significance, so I also required |log2 fold change| > 1. That left 4,659 genes (2,413 higher and 2,246 lower in TNBC), or 4,655 once duplicate gene symbols were collapsed (2,412 higher and 2,243 lower).

As a check on the pipeline, ESR1, GATA3 and FOXA1 were lower in TNBC and KRT5 and KRT17 were higher, which is what known biology predicts. METABRIC shows the same pattern, with smaller values because microarrays compress the range.

| Gene | TCGA log2 fold change | METABRIC difference (log2) |
|---|---|---|
| ESR1 | -5.25 | -4.05 |
| GATA3 | -3.20 | -2.93 |
| FOXA1 | -2.92 | -3.84 |
| KRT5 | +2.50 | +1.36 |
| KRT17 | +2.36 | +1.56 |

![Volcano plot of TNBC versus non-TNBC in TCGA](figures/volcano_tnbc_vs_non.png)

### Replication in METABRIC

In METABRIC, 12,708 of 20,384 genes had an adjusted p-value below 0.05, and 355 also had a difference above 1 (131 higher, 224 lower in TNBC). Fewer genes pass the size cutoff than in TCGA, which is expected because microarrays compress the range.

Of the 16,570 genes measured in both cohorts:

- The effect sizes correlated at Spearman 0.78, and 83% of genes moved in the same direction. That 83% is the baseline, since many genes shift together when ER biology differs.
- Of the 4,655 TCGA genes that passed my cutoffs, 2,976 could be compared in METABRIC (the rest are not in the METABRIC data, and many of them are non-coding genes or pseudogenes). Of those 2,976, 94% moved in the same direction and 2,473 (83%) were also significant in METABRIC: 1,366 of 1,622 higher in TNBC and 1,107 of 1,354 lower.

![TCGA fold change against METABRIC difference](figures/replication_scatter.png)

### Pathways

In TCGA (background: the 24,659 genes I tested), the Hallmark sets with adjusted p < 0.05 were:

| Direction in TNBC | Hallmark set | Adjusted p |
|---|---|---|
| Lower | Estrogen Response Early | 8e-30 |
| Lower | Estrogen Response Late | 7e-21 |
| Lower | Pancreas Beta Cells | 0.009 |
| Higher | KRAS Signaling Dn | 5e-08 |
| Higher | G2-M Checkpoint | 3e-05 |
| Higher | E2F Targets | 4e-04 |
| Higher | Spermatogenesis | 0.001 |
| Higher | Inflammatory Response | 0.027 |

I repeated the test on the genes that replicated in METABRIC (background: the 16,570 shared genes). The picture was the same:

- **Lower in TNBC (1,107 genes):** Estrogen Response Early (2e-34) and Late (2e-27), plus Pancreas Beta Cells and Xenobiotic Metabolism (both about 0.025).
- **Higher in TNBC (1,366 genes, 13 sets in total):** G2-M Checkpoint (2e-6), KRAS Signaling Dn (2e-6), E2F Targets (4e-5), and weaker sets including Inflammatory Response, Allograft Rejection, Hypoxia and Interferon Gamma Response (about 0.003 to 0.037).

Loss of estrogen response is close to expected, because TNBC is defined by loss of the estrogen receptor. The higher cell-cycle signal fits the faster growth of basal-like tumours and is the more informative result.

## What did not work or is unclear

- **173 genes went the opposite way in METABRIC**, but only 31 of those were significant there, and the differences were small (mostly under 0.4). Several are pseudogenes or members of gene families with near-identical copies (POTEKP, LGALS9B, LGALS9C, MT1A), which may be measured differently on the two platforms. I did not test this.
- **About a third of TCGA's significant genes could not be compared** because they are not in the METABRIC data.
- **The immune-related sets** (Inflammatory Response, Allograft Rejection, Interferon Gamma Response) were higher in TNBC in both cohorts. Because this is bulk tissue, I cannot tell whether the tumour cells themselves differ or whether TNBC tumours simply contain more immune cells.
- **KRAS Signaling Dn** was among the strongest higher-in-TNBC sets. It means genes that go down when KRAS is activated, so it does not show that KRAS signalling is active. I do not know how to interpret it, and I make no biological claim from it.
- **GO Biological Process enrichment would not load.** The library download failed on every attempt, so I report Hallmark sets only.

## Limitations

- TNBC is compared with all other breast cancers, so the results reflect ER, PR and HER2 biology as much as anything specific to TNBC.
- The comparison is unadjusted for age, stage or tumour cell composition.
- The groups are uneven (141 against 839 in TCGA), and with this many samples the test flags very small differences, so I relied on a fold-change cutoff as well as the adjusted p-value.
- Receptor status in TCGA came from the original TCGA study and was matched by patient ID. In METABRIC, HER2 status may come from copy number rather than IHC, and I did not confirm how it was derived. Among the TCGA TNBC tumours with a PAM50 label, 11 were HER2-enriched, 7 Normal-like and 4 Luminal, which reflects disagreement between IHC and expression-based subtypes.
- Both cohorts define TNBC by the same receptor rule, so strong agreement on ER-related genes is expected and is not an independent discovery.
- Fold changes in TCGA are not shrunk, and I did not verify which STAR count column Xena used.
- The METABRIC test is Welch's t-test, not limma.

## How I worked

I chose the research question and the scope of the project: a breast cancer analysis connected to my interest in TNBC and immune editing, deliberately limited to one project I could explain fully, with replication in a second cohort. I decided how TNBC would be defined, what counted as replication, and that negative and unclear results would be reported. I ran every step in Google Colab, checked each result against known biology, and interpreted the findings and limitations.

I used AI tools for help. Claude (Anthropic) helped me design the analysis steps, write and debug the Python code, and draft this README. Grok helped fix some code errors. I did not accept AI output without checking it. For example, I rejected a Grok version that replaced the receptor-based TNBC definition with PAM50 Basal-like as a proxy. When the web version of the enrichment test failed with a custom background, I used the offline version instead of dropping the background.

## Reproducing this

1. Open the notebook in Google Colab.
2. Run the cells in order. They download the public data, build the groups, run the TCGA analysis, then the METABRIC replication and the pathway tests. After the first cell (the package install), restart the session, then continue from the second cell.
3. By default the notebook reuses saved DESeq2 results to save time. Set `RERUN_DESEQ = True` in the setup cell to fit the model again (15 to 45 minutes).
4. The large input files (the TCGA counts matrix and the METABRIC expression file) are not stored here. They are public, and the notebook downloads them.

Python packages: PyDESeq2 0.5.4, pandas 2.2.3 (newer pandas versions broke PyDESeq2 for me), SciPy, statsmodels, GSEApy, matplotlib.

## Repository contents

- `TNBC_differential_expression_clean.ipynb`: the analysis notebook
- `figures/`: volcano plot and replication scatter plot

## References

Benjamini, Y., & Hochberg, Y. (1995). Controlling the false discovery rate: A practical and powerful approach to multiple testing. *Journal of the Royal Statistical Society: Series B (Methodological), 57*(1), 289-300.

Cancer Genome Atlas Network. (2012). Comprehensive molecular portraits of human breast tumours. *Nature, 490*(7418), 61-70.

Cerami, E., Gao, J., Dogrusoz, U., Gross, B. E., Sumer, S. O., Aksoy, B. A., ... & Schultz, N. (2012). The cBio cancer genomics portal: An open platform for exploring multidimensional cancer genomics data. *Cancer Discovery, 2*(5), 401-404.

Curtis, C., Shah, S. P., Chin, S.-F., Turashvili, G., Rueda, O. M., Dunning, M. J., ... & Aparicio, S. (2012). The genomic and transcriptomic architecture of 2,000 breast tumours reveals novel subgroups. *Nature, 486*(7403), 346-352.

Fang, Z., Liu, X., & Peltz, G. (2023). GSEApy: A comprehensive package for performing gene set enrichment analysis in Python. *Bioinformatics, 39*(1), btac757.

Foulkes, W. D., Smith, I. E., & Reis-Filho, J. S. (2010). Triple-negative breast cancer. *New England Journal of Medicine, 363*(20), 1938-1948.

Frankish, A., Diekhans, M., Jungreis, I., Lagarde, J., Loveland, J. E., Mudge, J. M., ... & Flicek, P. (2021). GENCODE 2021. *Nucleic Acids Research, 49*(D1), D916-D923.

Gao, J., Aksoy, B. A., Dogrusoz, U., Dresdner, G., Gross, B., Sumer, S. O., ... & Schultz, N. (2013). Integrative analysis of complex cancer genomics and clinical profiles using the cBioPortal. *Science Signaling, 6*(269), pl1.

Goldman, M. J., Craft, B., Hastie, M., Repečka, K., McDade, F., Kamath, A., ... & Haussler, D. (2020). Visualizing and interpreting cancer genomics data via the Xena platform. *Nature Biotechnology, 38*(6), 675-678.

Kuleshov, M. V., Jones, M. R., Rouillard, A. D., Fernandez, N. F., Duan, Q., Wang, Z., ... & Ma'ayan, A. (2016). Enrichr: A comprehensive gene set enrichment analysis web server 2016 update. *Nucleic Acids Research, 44*(W1), W90-W97.

Liberzon, A., Birger, C., Thorvaldsdóttir, H., Ghandi, M., Mesirov, J. P., & Tamayo, P. (2015). The molecular signatures database hallmark gene set collection. *Cell Systems, 1*(6), 417-425.

Love, M. I., Huber, W., & Anders, S. (2014). Moderated estimation of fold change and dispersion for RNA-seq data with DESeq2. *Genome Biology, 15*(12), 550.

Muzellec, B., Teleńczuk, M., Cabeli, V., & Andreux, M. (2023). PyDESeq2: A python package for bulk RNA-seq differential expression analysis. *Bioinformatics, 39*(9), btad547.

Parker, J. S., Mullins, M., Cheang, M. C. U., Leung, S., Voduc, D., Vickery, T., ... & Bernard, P. S. (2009). Supervised risk predictor of breast cancer based on intrinsic subtypes. *Journal of Clinical Oncology, 27*(8), 1160-1167.

Prat, A., Parker, J. S., Karginova, O., Fan, C., Livasy, C., Herschkowitz, J. I., ... & Perou, C. M. (2010). Phenotypic and molecular characterization of the claudin-low intrinsic subtype of breast cancer. *Breast Cancer Research, 12*(5), R68.

## Contact

Dickens Agie, dochieng014@gmail.com
