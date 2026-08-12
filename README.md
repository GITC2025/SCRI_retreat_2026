# SCRI_retreat_2026
- extension of SCRI Retreat 2026 poster - static version (11 Aug 2026)
- contact person: delphine.ang@queensu.ca
- funded by TFRI PPG 2024-2029, Sinclair Graduate Scholars Award 2026-2027 (D.A.) and Sinclair Summer Studentship 2026 (Y.G.)
- [SCRI Retreat Abstract 12 July 2026.pdf](https://github.com/user-attachments/files/30965052/SCRI.Retreat.Abstract.12.July.2026.pdf)
- analysis conducted in a custom R 4.5.3 container on Narval Cluster HPC - Alliance Canada, 128GB RAM and 8 cores

<img width="11200" height="7200" alt="updateSCRIpostergreen_8aug-1" src="https://github.com/user-attachments/assets/18485d22-3a63-4958-b146-ff11badba365" />

# Paradoxical Maturation 
<img width="642" height="510" alt="image" src="https://github.com/user-attachments/assets/6d48d30b-22fa-4beb-a463-0776c7dea63e" />

- PM found at the invasive front of pT1 (superficially invasive) NMIBC cases
- increased cytoplasm : nucleus ratio (lower nuclear density)
- correlation with progression risk?
- literature focuses on histological features : Cheng et al., 2009, Toll et al., 2012, Raspollini et al., 2020, Iakymenko et al., 2022
- molecular basis uncharacterised

## Visual metrics 
- work done by Barberry Yu based on Delphine's template
- segmented in QUpath instanseg
- select the densest (nonPM) area to match the least dense (PM) area

<img width="569" height="425" alt="image" src="https://github.com/user-attachments/assets/2b2babb9-48c7-40a8-831c-c84a25dd5c1e" />

- Sum of PM Areas: 87460 px2
- Number of detections (nuclei): 270
- Nuclei density per total cell area = 30.8 nuclei per 10,000 px2
- Non-PM Area: 87415 px2
- Number of detections (nuclei): 449
- Nuclei density per total cell area = 51.4 nuclei per 10,000 px2
- PM areas 60% nuclear density of nonPM area
- nonPM has approximately ~1.66x more nuclei than PM 

# Hypothesis
- PM cells are senescent phenotypes
- senescence triggered by stress during microinvasion (hypoxia, pseudohypoxia, ECM changes etc.)
- mTORC1 activity contributes to increased cell size
- senescence-associated plasticity drives epigenetic reprogramming into proliferative, aggressive phenotype with postsenescent features

<img width="1462" height="1728" alt="2x3numbered - Copy" src="https://github.com/user-attachments/assets/207ff3f8-cbf9-494e-867f-733e730d25a0" />

# Mine scRNA data for senescent epithelial phenotypes
Luo, Y., Tao, T., Tao, R., Huang, G., & Wu, S. (2022). Single-Cell Transcriptome Comparison of Bladder Cancer Reveals Its Ecosystem. Frontiers in oncology, 12, 818147. https://doi.org/10.3389/fonc.2022.818147
* https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE190888
* https://www.ebi.ac.uk/ena/browser/view/SRP351272
* 4 samples total: 2x pri BCa, 1 recurrent BCa and one cystitis glandularis (Singleron platform)
* use the 3 BLCA samples

| Run | Sample | Type | Grading | Age | Gender | Other | 
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| SRR17259462 | LHBC0803 | Urothelial Carcinoma | High Grade | 58 | Male | None | 
| SRR17259463 | LHBC0911 | Urothelial Carcinoma | High Grade | 58 | Male | LHBC0803 Recurrence | 
| SRR17259464 | LHBC0917 | Urothelial Carcinoma | Low Grade | 28 | Male | None | 
| SRR17259465 | LH0826 | Cystitis Glandularis | None | 39 | Male | None | 

# preprocessing scRNA data
- Narval Cluster HPC on Alliance Canada
- fastq.gz downloaded from ebi, aligned using Celescope to latest human reference genome, and checked with FastQC
- zcat checked chemistry: Singleron GEXSCOPER protocol (150x150 paired end chemistry)

# Seurat QC
- data driven floors and ceilings need to be set for counts/features, and ceiling need to be set for mito content
- low count/features, and high mito content indicate low quality cells
- exceptionally high count/features indicate doublets
- ribosomal % is of biological significance though not a standard QC metric
- in general, exceptionally low ribosomal % indicates absence of transcriptional activity and hence low quality cells
- we run deContX for decontamination, scDblfinder to remove doublets, and set a 99th percentile threshold to remove the top 1% of counts
- due to the nature of the dataset, we set a mito threshold at 30% or 35% to avoid excessive cell loss, and may have to QC further downstream 

<img width="800" height="400" alt="image" src="https://github.com/user-attachments/assets/fa37b8d0-3704-4595-af5e-e3550e5963c7" /> \
<img width="800" height="800" alt="image" src="https://github.com/user-attachments/assets/d76b7028-470f-41bd-815a-bf94f21bf22c" /> \

# post QC
- only focus on the 3 BLCA samples 
```
attrition summary
   sample_id unfiltered_cells final_singlets cells_removed attrition_rate_pct
 SRR17259462             7020           4926          2094              29.83
 SRR17259463             8723           6141          2582              29.60
 SRR17259464             6941           4759          2182              31.44
```
<img width="800" height="400" alt="image" src="https://github.com/user-attachments/assets/d5c335f8-93b2-40f0-b83c-f71111d69b17" />

# Normalization
- SCTransform was used for normalization of counts and to find the top 3000 consensus highly variable genes for downstream dimension reduction
- genes expressed in less than 3 cells are dropped 
- differential gene expression would still be run on the full feature space (minus the genes xp in < 3 cells)

# examine PCAs per sample
- in hunting for rare cell types, we can do PCA/UMAP per sample first on the consensus 3000 HVGs before integration
- check whether cells separate by cell cycle variables or technical variables in the first 2 largest axes of variance
- there is a decent level of mixing, though not perfect
- we don't regress out anything at this stage as 'technical' covariates (e.g. transcriptional complexity - counts) have biological correlations
- integration downstream can correct for batch effects

<img width="1800" height="600" alt="image" src="https://github.com/user-attachments/assets/d7e136a5-59c7-4a23-92a6-60b721296c8c" /> \
<img width="5400" height="4500" alt="image" src="https://github.com/user-attachments/assets/07109194-3a2a-4714-8dfc-ace9cc8c63d0" />

- as expected, the top loadings separate cell lineages 
```
TOP GENE LOADINGS FOR PC1 AND PC2
PC_ 1 
Positive:  SPINK1, S100P, ADIRF, KRT13, LY6D, S100A6, DHRS2, S100A2, FABP5, KRT19
           HES1, CLDN4, GPX2, AGR2, GSTP1, KRT7, S100A11, HSPB1, KRT17, ELF3
           HPGD, TACSTD2, ID1, MT-CYB, MT-ND3, AQP3, PERP, FXYD3, MT-ND2, PSCA
Negative:  CD74, HLA-DRA, RGS1, HLA-DPA1, HLA-DRB1, HLA-DPB1, TYROBP, AIF1, HLA-DQA1, SRGN
           MS4A6A, FCER1G, LYZ, FGL2, HLA-DQB1, C1QB, LAPTM5, FTL, C1QC, RGS2
           IGSF6, CCL3, CYBB, GPR183, SGK1, HLA-DMB, PTPRC, C1QA, MNDA, CCL3L3
PC_ 2
Positive:  DCN, LUM, COL1A2, COL3A1, A2M, COL1A1, COL6A3, SPARC, PDGFRA, LGALS1
           C1R, BMP5, CALD1, MMP2, PLAT, C1S, COL6A1, MFAP4, POSTN, IGFBP7
           IFITM3, TRPA1, F3, VIM, SERPING1, FN1, NR2F1, COL5A1, APCDD1, IL6ST
Negative:  SPINK1, S100P, LY6D, CD74, DHRS2, KRT13, HES1, HLA-DRA, RGS1, S100A11
           FABP5, FOS, CXCL8, LYZ, KRT19, TYROBP, GPX2, AIF1, CLDN4, HLA-DPA1
           MS4A6A, HPGD, HSP90AA1, ATF3, HLA-DPB1, HLA-DQA1, ADIRF, HLA-DRB1, AGR2, CCL3
```

# subset to epithelial cells 
- after identifying a possibly senescent axis in the mixed cell populations, we subset to epithelial cells by this logic
- we do a parameter sweep from low to high settings to ensure subsetting is stable across a range of parameters

```
[Cell Lineage] 
      │
      ├──> (Max Score <= -0.05) ──────────────> Lineage Unresolved (Negative Score)
      │
      ├──> (Max - Runner Up < 0.02) 
      │         ├──> If Epi & Stromal ────────> Epithelial (EMT Hybrid)
      │         └──> Else ────────────────────> Lineage Unresolved (Mixed Signal)
      │
      └──> (Max - Runner Up >= 0.02) 
                ├──> If Immune Wins ──────────> Non-Tumor (Immune)
                ├──> If Stromal Wins ─────────> Non-Tumor (Stromal)
                └──> If Epi_Total Wins 
                          │
                          ├──> (|Lum - Bas| < 0.02) ──> Epithelial (Epi Hybrid)
                          ├──> (Lum > Bas) ───────────> Epithelial (Luminal)
                          └──> (Bas > Lum) ───────────> Epithelial (Basal)
```

```
SUBSET ATTRITION SUMMARY:  SRR17259462 

|Metric                                  |Value  |
|:---------------------------------------|:------|
|Total Initial Cells                     |4926   |
|Total Retained Epithelial Cells         |4175   |
|Total Removed Non-Epithelial/Unresolved |751    |
|Percentage Retained                     |84.75% |
Epithelial subset saved to: /lustre07/scratch/delphine/dataset2_R_output/SRR17259462_epithelial_subset.rds

SUBSET ATTRITION SUMMARY:  SRR17259463

|Metric                                  |Value  |
|:---------------------------------------|:------|
|Total Initial Cells                     |6141   |
|Total Retained Epithelial Cells         |2793   |
|Total Removed Non-Epithelial/Unresolved |3348   |
|Percentage Retained                     |45.48% |
Epithelial subset saved to: /lustre07/scratch/delphine/dataset2_R_output/SRR17259463_epithelial_subset.rds 

SUBSET ATTRITION SUMMARY:  SRR17259464

|Metric                                  |Value  |
|:---------------------------------------|:------|
|Total Initial Cells                     |4759   |
|Total Retained Epithelial Cells         |4124   |
|Total Removed Non-Epithelial/Unresolved |635    |
|Percentage Retained                     |86.66% |
Epithelial subset saved to: /lustre07/scratch/delphine/dataset2_R_output/SRR17259464_epithelial_subset.rds
```

# re-normalise and dimension reduction
- since the feature space has changed after subsetting, we re-normalise, select 3000 HVGs and reduce dimensions in the epithelial subset
- zooming in this way, within-lineage transcriptomic differences will show up clearly in the major axes of variance

# epithelial only PCA
- separation by cell cycle is of clinical interest to us so we don't regress out anything here
- we can see clear technical separation - this would need to be QC'd downstream

<img width="1800" height="600" alt="image" src="https://github.com/user-attachments/assets/14ea299e-7943-40b0-9d8d-929e411b39cf" /> \
<img width="2700" height="2250" alt="image" src="https://github.com/user-attachments/assets/554bcfd0-8aab-4238-ae2a-f4ea2670045e" />

# PCA dimheatmap
- to show if the top 250 cells separate cleanly from the bottom 250 cells per marker in the top PCs
- PC1, 2 and 3 capture binary differences, and then it gets fuzzier downstream

<img width="3600" height="4800" alt="image" src="https://github.com/user-attachments/assets/2820e477-1464-4939-916c-eba3f66ceeab" />

# per sample UMAP module score overlays
- we now overlay some module scores of relevant GSEA gene sets to see if senescent phenotypes are present
- we overlay various senescence panels, plasticity panels, invasive front panels etc. 

<img width="5400" height="1800" alt="image" src="https://github.com/user-attachments/assets/41f023b6-7083-4b53-9db7-f554cdd12b3f" /> \
<img width="7200" height="1800" alt="image" src="https://github.com/user-attachments/assets/41de0073-2220-42a9-9d3a-245e96b1b2cf" /> \
<img width="7200" height="1800" alt="image" src="https://github.com/user-attachments/assets/59e8ca68-bbf9-43ac-a3e0-a90bcab315e2" />

- we check coverage metrics

```
Panel Name: Corescence 39
Total Reference Genes in Set: 39
  Sample: SRR17259462 -> Found: 34 / 39 (87.18%)
    Missing elements: IL6, IGFBP1, FGF2, CCL2, WNT2
  Sample: SRR17259463 -> Found: 34 / 39 (87.18%)
    Missing elements: IL6, IGFBP1, FGF2, CCL2, WNT2
  Sample: SRR17259464 -> Found: 34 / 39 (87.18%)
    Missing elements: IL6, IGFBP1, FGF2, CCL2, WNT2

Panel Name: Reactome SASP
Total Reference Genes in Set: 111
  Sample: SRR17259462 -> Found: 102 / 111 (91.89%)
  Sample: SRR17259463 -> Found: 102 / 111 (91.89%)
  Sample: SRR17259464 -> Found: 102 / 111 (91.89%)

Panel Name: mTORC1 Hallmark
Total Reference Genes in Set: 200
  Sample: SRR17259462 -> Found: 198 / 200 (99%)
    Missing elements: CFP, STARD4
  Sample: SRR17259463 -> Found: 198 / 200 (99%)
    Missing elements: CFP, STARD4
  Sample: SRR17259464 -> Found: 198 / 200 (99%)
    Missing elements: CFP, STARD4

Panel Name: Wu Cell Migration
Total Reference Genes in Set: 183
  Sample: SRR17259462 -> Found: 167 / 183 (91.26%)
  Sample: SRR17259463 -> Found: 167 / 183 (91.26%)
  Sample: SRR17259464 -> Found: 167 / 183 (91.26%)

Panel Name: GOBP Cell Migration
Total Reference Genes in Set: 1685
  Sample: SRR17259462 -> Found: 1183 / 1685 (70.21%)
  Sample: SRR17259463 -> Found: 1183 / 1685 (70.21%)
  Sample: SRR17259464 -> Found: 1183 / 1685 (70.21%)

Panel Name: Positive Regulation of Cell Migration
Total Reference Genes in Set: 10
  Sample: SRR17259462 -> Found: 8 / 10 (80%)
    Missing elements: ANGPTL3, CRIPTO
  Sample: SRR17259463 -> Found: 8 / 10 (80%)
    Missing elements: ANGPTL3, CRIPTO
  Sample: SRR17259464 -> Found: 8 / 10 (80%)
    Missing elements: ANGPTL3, CRIPTO

Panel Name: Negative Regulation of Cell Migration
Total Reference Genes in Set: 15
  Sample: SRR17259462 -> Found: 9 / 15 (60%)
  Sample: SRR17259463 -> Found: 9 / 15 (60%)
  Sample: SRR17259464 -> Found: 9 / 15 (60%)

Panel Name: Hallmark EMT
Total Reference Genes in Set: 200
  Sample: SRR17259462 -> Found: 152 / 200 (76%)
  Sample: SRR17259463 -> Found: 152 / 200 (76%)
  Sample: SRR17259464 -> Found: 152 / 200 (76%)

Panel Name: Hallmark Hypoxia
Total Reference Genes in Set: 200
  Sample: SRR17259462 -> Found: 178 / 200 (89%)
  Sample: SRR17259463 -> Found: 178 / 200 (89%)
  Sample: SRR17259464 -> Found: 178 / 200 (89%)

Panel Name: Yoshihara 2026 Invasive Front
Total Reference Genes in Set: 15
  Sample: SRR17259462 -> Found: 7 / 15 (46.67%)
  Sample: SRR17259463 -> Found: 7 / 15 (46.67%)
  Sample: SRR17259464 -> Found: 7 / 15 (46.67%)

Panel Name: Reactome Epigenetic Regulation 
Total Reference Genes in Set: 320
  Sample: SRR17259462 -> Found: 303 / 320 (94.69%)
  Sample: SRR17259463 -> Found: 303 / 320 (94.69%)
  Sample: SRR17259464 -> Found: 303 / 320 (94.69%)

etc.
```

# investigate PC5 - senescence and dissoc stress markers
```
PC_ 5
Positive:  GDF15, HES1, CLDN4, KLF6, ATF3, CXCL8, KRT17, TMSB4X, IER3, NFKBIA
           S100A2, HSPA1A, TUBA1B, S100A6, KRT15, PMAIP1, NFKBIZ, AQP3, FOS, HSP90AA1
           AGR2, STMN1, ACTB, JUN, DNAJB1, HMGB2, ANKRD36C, LCN2, H3-3B, TMSB10
Negative:  DHRS2, LY6D, KRT13, MT-ND1, UPK1B, MT-ND2, MT-ATP6, MT-CO1, MT-ND5, SPINK1
           MT-CYB, MT-ND3, PTGR1, ADIRF, PSCA, RPS6, RPS18, UPK2, GMNN, HPGD
           TNNT3, SNCG, RPLP1, IDH1, RPS16, RPL31, RPL37, IGFBP3, RPL30, PLA2G2A
```
- PC5 positive is of interest here
- KRT15, 17, S100A2, A6 and AQP3 indicate basal subtype (common at tumour boundary and assoc with aggresiveness in NMIBC)
- set of markers correlated with NFKB pathway to SASP
- but could also be confounded by dissociation cell stress in protocols
- check for true proliferative arrest and upregulation of CDKN1A and CDKN2A
- dual module score analysis - dissociation artefact gene set vs sen gene sets

# disentangle true senescence from technical stress
- investigating our PC5 loading, we find a mixture of biological programs
- untangling true senescence and handling/dissoc stress is crucial, but consider the overlapping markers between the two
- true sen will contain high levels of true sen markers, relatively low in overlapping markers, minimal dissoc stress markers and low proliferation

```
# pc5_dissoc_stress
FOS, JUN, FOSB, KLF6, ATF3, HSPA1A, HSP90AA1, DNAJB1, DUSP1, UBC

# pc5_overlap
GDF15, CXCL8, LCN2, IER3, GADD45A, GADD45B, DDIT3, PPP1R15A, PMAIP1, PHLDA1, NFKBIA, NFKBIZ, BTG2

# pc5_senescence_core
HES1, SOX4, PTGS2

# pc5_proliferation_cycling
TOP2A, PTTG1, CENPF, STMN1, HMGB2, H3-3B, H2AZ1, PTMA

# pc5_epithelial_lineage
CLDN4, KRT17, KRT15, S100A2, S100A6, AQP3, AGR2, TFF3

# pc5_cytoskeleton
TMSB4X, TMSB10, ACTB, ACTG1, TUBA1B, TUBB
```

- we create a dissoc stress panel (40 genes) based on O'Flanagan et al 2019 top 40
- overlapping genes between O'Flanagan 40 and corescence 39 and placed in overlap
- move genes around in the categories based on current knowledge (to be refined)
- unique genes across all 3 panels to avoid double counting
- exact categories are a current research gap
- ideally we want sen-only genes to be associated with sen 90% of the time, same for stress-only genes, with overlap panels being 50-50 assoc with either

```
# 3 July 2026 version
SenCore Panel (n = 35):
 AXL, BRCA1, BUB1B, CCNA2, CDK1, CDKN2A, CDKN2B, EGFR, FAS, FGF2, FOXM1, HELLS, HES1, HMGB1, HMGB2, ICAM1, IGF1, IGFBP1, IGFBP2, 
 IGFBP3, IGFBP5, IGFBP7, IL1A, LMNB1, MDM2, MIF, PARP1, PTGS2, SERPINE1, SOX4, STAT1, TGFB1, TNFRSF10C, VEGFA, WNT2

Flanagan Unique Panel (n = 33):
 CCNL1, CEBPD, CLDN4, CXCL2, CXCL3, CYR61, DEPP1, DNAJB1, DUSP1, DUSP2, EGR1, FOS, FOSB, HSPA1A, HSPA1B, HSPA6, ID1, IER2, IER5, 
 IRF1,  JUN, JUNB, KLF4, KLF6, KRT6A, MAFF, MTRNR2L2, NR4A1, PHLDA2, PLAUR, SOCS3, TNFAIP3, ZFP36

Overlap Panel (n = 18):
 ATF3, BTG2, CCL2, CDKN1A, CXCL1, CXCL8, DDIT3, GADD45A, GADD45B, GDF15, IER3, IL6, LCN2, NFKBIA, NFKBIZ, PHLDA1, PMAIP1, PPP1R15A
```

<img width="2400" height="1800" alt="image" src="https://github.com/user-attachments/assets/91ddff5b-db7e-4fac-852a-fa1df0299204" /> \
<img width="4800" height="3600" alt="image" src="https://github.com/user-attachments/assets/94eedec6-3b64-4985-b2a9-54ff561d6dcd" /> \
<img width="4800" height="3600" alt="image" src="https://github.com/user-attachments/assets/f3290471-f2d9-44de-bb83-ca148c25fe9c" />

- consider the high proportion of sample 62 Q1 true senescent cells in cluster 6 yet paired with an exceptionally high proliferation score
- we assume that true senescence is characterised by proliferative arrest
- but it not necessarily be so at the invasive front
- consider Sen-Mark+ cancer cells which would exhibit senescence and proliferation like cluster 4 Q1
- O’Sullivan, E.A., Wallis, R., Mossa, F. et al. The paradox of senescent-marker positive cancer cells: challenges and opportunities. npj Aging 10, 41 (2024). https://doi.org/10.1038/s41514-024-00168-y
- are these phenotypes truly senescent and proliferative, or does this indicate phenotypic transition? - we investigate later
- alternatively, the invasive front might be a patchwork of sen+, non-proliferative cells along with highly proliferative cells
- sen+ non prolif cells may act in a paracrine manner on nearby sen- cells to promote proliferation and invasion

# deconvolution metrics per sample

```
# 03July_163210
SAMPLE SRR17259462 FULL MANIFOLD SIGNAL DECONVOLUTION SUMMARY (MODIFIED CORESCENCE PANEL)
Total Footprint Evaluated: 4143 cells across all clusters

 Cluster                     Quadrant Cell_Count Mean_SenCore Mean_Stress Mean_Overlap Mean_Prolif Low_Prolif_Count Cluster_Percentage Low_Prolif_Within_Quad_Pct
       1           Q1: True Senescent        226       0.1335      0.1917      -0.0466     -0.0051              141              13.61                      62.39
       1       Q2: Stressed Senescent        532       0.1398      0.4622       0.2309     -0.0097              324              32.03                      60.90
       1    Q3: Baseline/Unresponsive        403       0.0337      0.1564      -0.1501     -0.0120              281              24.26                      69.73
       1 Q4: Pure Dissociation Stress        500       0.0440      0.4515       0.1694     -0.0142              342              30.10                      68.40

 Cluster                     Quadrant Cell_Count Mean_SenCore Mean_Stress Mean_Overlap Mean_Prolif Low_Prolif_Count Cluster_Percentage Low_Prolif_Within_Quad_Pct
       2           Q1: True Senescent        178       0.1339      0.2018      -0.0872      0.0191               36              18.02                      20.22
       2       Q2: Stressed Senescent        495       0.1471      0.4849       0.1707      0.0183               91              50.10                      18.38
       2    Q3: Baseline/Unresponsive        160       0.0561      0.1801      -0.1249      0.0097               50              16.19                      31.25
       2 Q4: Pure Dissociation Stress        155       0.0628      0.4578       0.1167      0.0119               40              15.69                      25.81

 Cluster                     Quadrant Cell_Count Mean_SenCore Mean_Stress Mean_Overlap Mean_Prolif Low_Prolif_Count Cluster_Percentage Low_Prolif_Within_Quad_Pct
       3           Q1: True Senescent        145       0.1274      0.1662      -0.1638      0.0004               76              23.42                      52.41
       3       Q2: Stressed Senescent        148       0.1398      0.4460       0.1269      0.0070               57              23.91                      38.51
       3    Q3: Baseline/Unresponsive        205       0.0400      0.1153      -0.2317     -0.0004               94              33.12                      45.85
       3 Q4: Pure Dissociation Stress        121       0.0546      0.4227       0.0472      0.0012               55              19.55                      45.45

 Cluster                     Quadrant Cell_Count Mean_SenCore Mean_Stress Mean_Overlap Mean_Prolif Low_Prolif_Count Cluster_Percentage Low_Prolif_Within_Quad_Pct
       4           Q1: True Senescent         79       0.1209      0.0962      -0.0603     -0.0240               60              23.44                      75.95
       4       Q2: Stressed Senescent         14       0.1418      0.3662       0.2105     -0.0149                9               4.15                      64.29
       4    Q3: Baseline/Unresponsive        229       0.0298      0.0259      -0.1385     -0.0314              194              67.95                      84.72
       4 Q4: Pure Dissociation Stress         15       0.0516      0.3943       0.0281     -0.0187               13               4.45                      86.67

 Cluster                     Quadrant Cell_Count Mean_SenCore Mean_Stress Mean_Overlap Mean_Prolif Low_Prolif_Count Cluster_Percentage Low_Prolif_Within_Quad_Pct
       5           Q1: True Senescent         72       0.1315      0.0875      -0.2511      0.0120               35              22.71                      48.61
       5       Q2: Stressed Senescent         12       0.1233      0.3756      -0.0444      0.0016                6               3.79                      50.00
       5    Q3: Baseline/Unresponsive        218       0.0127      0.0032      -0.2718     -0.0107              149              68.77                      68.35
       5 Q4: Pure Dissociation Stress         15       0.0472      0.3687      -0.0202     -0.0298               13               4.73                      86.67

 Cluster                     Quadrant Cell_Count Mean_SenCore Mean_Stress Mean_Overlap Mean_Prolif Low_Prolif_Count Cluster_Percentage Low_Prolif_Within_Quad_Pct
       6           Q1: True Senescent        119       0.1886      0.0838      -0.2113      0.3352                1              53.85                       0.84
       6       Q2: Stressed Senescent         52       0.2024      0.4031       0.0382      0.3263                2              23.53                       3.85
       6    Q3: Baseline/Unresponsive         37       0.0558      0.0610      -0.2754      0.1751                1              16.74                       2.70
       6 Q4: Pure Dissociation Stress         13       0.0617      0.4272       0.0172      0.1428                1               5.88                       7.69

SAMPLE SRR17259463 FULL MANIFOLD SIGNAL DECONVOLUTION SUMMARY (MODIFIED CORESCENCE PANEL)
Total Footprint Evaluated: 2741 cells across all clusters

 Cluster                     Quadrant Cell_Count Mean_SenCore Mean_Stress Mean_Overlap Mean_Prolif Low_Prolif_Count Cluster_Percentage Low_Prolif_Within_Quad_Pct
       1           Q1: True Senescent        149       0.0995      0.1662       0.1434      0.0258               48              15.49                      32.21
       1       Q2: Stressed Senescent        303       0.1030      0.4129       0.3282      0.0280               68              31.50                      22.44
       1    Q3: Baseline/Unresponsive        270       0.0049      0.1165       0.0426      0.0205              117              28.07                      43.33
       1 Q4: Pure Dissociation Stress        240       0.0178      0.3990       0.2906      0.0244               81              24.95                      33.75

 Cluster                     Quadrant Cell_Count Mean_SenCore Mean_Stress Mean_Overlap Mean_Prolif Low_Prolif_Count Cluster_Percentage Low_Prolif_Within_Quad_Pct
       2           Q1: True Senescent        142       0.1046      0.1535       0.1529      0.0435               44              15.81                      30.99
       2       Q2: Stressed Senescent        265       0.1097      0.3996       0.3394      0.0283               94              29.51                      35.47
       2    Q3: Baseline/Unresponsive        258       0.0004      0.1144       0.0272      0.0220              118              28.73                      45.74
       2 Q4: Pure Dissociation Stress        233       0.0141      0.3913       0.3101      0.0206              101              25.95                      43.35

 Cluster                     Quadrant Cell_Count Mean_SenCore Mean_Stress Mean_Overlap Mean_Prolif Low_Prolif_Count Cluster_Percentage Low_Prolif_Within_Quad_Pct
       3           Q1: True Senescent        265       0.1050      0.1759       0.0947      0.0102              203              30.08                      76.60
       3       Q2: Stressed Senescent        247       0.1179      0.3488       0.2167      0.0099              185              28.04                      74.90
       3    Q3: Baseline/Unresponsive        286       0.0205      0.1390       0.0209      0.0009              247              32.46                      86.36
       3 Q4: Pure Dissociation Stress         83       0.0270      0.3516       0.1734      0.0049               64               9.42                      77.11

SAMPLE SRR17259464 FULL MANIFOLD SIGNAL DECONVOLUTION SUMMARY (MODIFIED CORESCENCE PANEL)
Total Footprint Evaluated: 3936 cells across all clusters

 Cluster                     Quadrant Cell_Count Mean_SenCore Mean_Stress Mean_Overlap Mean_Prolif Low_Prolif_Count Cluster_Percentage Low_Prolif_Within_Quad_Pct
       1           Q1: True Senescent        417       0.0821      0.1849       0.1743      0.0169              197              16.65                      47.24
       1       Q2: Stressed Senescent        583       0.0918      0.4550       0.4704      0.0181              256              23.27                      43.91
       1    Q3: Baseline/Unresponsive        863      -0.0310      0.1486       0.1066      0.0144              446              34.45                      51.68
       1 Q4: Pure Dissociation Stress        642      -0.0192      0.4212       0.3941      0.0178              286              25.63                      44.55

 Cluster                     Quadrant Cell_Count Mean_SenCore Mean_Stress Mean_Overlap Mean_Prolif Low_Prolif_Count Cluster_Percentage Low_Prolif_Within_Quad_Pct
       2           Q1: True Senescent        347       0.1063      0.1253       0.0683      0.0396              123              41.51                      35.45
       2       Q2: Stressed Senescent        181       0.1124      0.4145       0.3689      0.0227               70              21.65                      38.67
       2    Q3: Baseline/Unresponsive        207      -0.0176      0.0968      -0.0196      0.0215               99              24.76                      47.83
       2 Q4: Pure Dissociation Stress        101      -0.0033      0.3961       0.3263      0.0237               36              12.08                      35.64

 Cluster                     Quadrant Cell_Count Mean_SenCore Mean_Stress Mean_Overlap Mean_Prolif Low_Prolif_Count Cluster_Percentage Low_Prolif_Within_Quad_Pct
       3           Q1: True Senescent         70       0.1073      0.2520       0.3204     -0.0050               58              14.31                      82.86
       3       Q2: Stressed Senescent        311       0.1150      0.4335       0.4392     -0.0058              258              63.60                      82.96
       3    Q3: Baseline/Unresponsive         23      -0.0016      0.2353       0.2336     -0.0111               22               4.70                      95.65
       3 Q4: Pure Dissociation Stress         85       0.0033      0.4121       0.3968     -0.0082               73              17.38                      85.88

 Cluster                     Quadrant Cell_Count Mean_SenCore Mean_Stress Mean_Overlap Mean_Prolif Low_Prolif_Count Cluster_Percentage Low_Prolif_Within_Quad_Pct
       4           Q1: True Senescent         18       0.0907      0.1977       0.1338      0.0199                8              16.98                      44.44
       4       Q2: Stressed Senescent         41       0.0875      0.4075       0.3608      0.0246               13              38.68                      31.71
       4    Q3: Baseline/Unresponsive         23       0.0036      0.2087       0.1291      0.0136               13              21.70                      56.52
       4 Q4: Pure Dissociation Stress         24      -0.0003      0.4202       0.3301      0.0195               10              22.64                      41.67
```

## run mito correlations for deconvolution
- to see if we find anything interesting - but mito levels are well mixed across all quadrants
- indicating that mito levels are not a confounder here

<img width="2000" height="2611" alt="image" src="https://github.com/user-attachments/assets/a19a2527-b7cb-4c74-afc7-a7240c77fda2" />

# run epigenetic regulator set correlations
- plasticity may mediate the opposing states of senescence vs proliferation
- i.e. sen-prolif cells may be postsenescent cells transitioning into prolif phenotypes via plasticity
- we run epigenetic regulator panels against corescence vs prolif panel correlations
- very nice positive trend in 62 and 63 (matched samples), but no similar trend in 64

<img width="3600" height="2100" alt="image" src="https://github.com/user-attachments/assets/80fe6a6c-170a-4862-90d5-b0c06412c2e6" /> \
<img width="3600" height="2100" alt="image" src="https://github.com/user-attachments/assets/88f44d11-20f1-4cda-a6ff-d70c624eefab" /> \
<img width="3600" height="2100" alt="image" src="https://github.com/user-attachments/assets/26e196b8-f1c0-41e2-b8df-91c65e995ebc" />

# per quadrant correlations
- we run the same correlations per quadrant from the deconvolution to ensure that the trend is specific to senescence
- the strong trend in stressed-sen cells suggest true senescence mixed with other/technica stress 

<img width="4200" height="2700" alt="image" src="https://github.com/user-attachments/assets/be874eb7-dc19-4954-ac62-eb221ba1309f" />

# multivariate model
- build a multivariate model correlating the triad, just for Q1 sample 62, we would expand to whole cluster later in the integrated dataset 

<img width="2200" height="1000" alt="image" src="https://github.com/user-attachments/assets/0e82b540-7ee5-4813-9106-e503511857ed" />

# harmony integration
- integrate all samples at various resolution
- integration performance can be tested via scIntegrationmetrics, LISI, kBET etc
- examining various resolutions, we choose Leiden 0.5 

<img width="4200" height="1800" alt="image" src="https://github.com/user-attachments/assets/d1e815e0-202d-4925-90af-74771e86a9ee" />

# check outliers
- partitioning in an UMAP may indicate outliers, do QC overlay
- very clearly, the lemon shaped island (cluster 4) is a technical artifact
- probably due to our generous mito ceiling
- but our generous mito ceiling did allow some genuinely viable cells to show up, right in the center of the main cluster
- here you can see how telling a low ribo % is for a low quality cluster 

<img width="3600" height="3000" alt="image" src="https://github.com/user-attachments/assets/5621a348-9d1b-4cf9-8fe4-a99fc43d5c5d" />

```
per-cluster technical qc summary (resolution 0.5):
 Cluster Cells nFeature_Median nFeature_Range nCount_Median  nCount_Range pct_MT_Median  pct_MT_Range pct_Ribo_Median pct_Ribo_Range
       1  2871          2503.0    [288, 4750]        7446.0  [503, 23431]         18.80    [1, 34.81]           13.91  [4.68, 28.96]
       2  2233          2232.0    [253, 4793]        5609.0  [508, 23371]         16.21 [1.89, 34.94]           13.09  [4.95, 28.55]
       3  2070          2014.5    [252, 4666]        5120.5  [510, 21513]         15.26 [1.22, 32.74]           18.94  [7.27, 44.24]
       4  1421           774.0    [219, 4120]        1781.0  [506, 11181]         24.81    [1.24, 35]            2.41  [0.16, 19.76]
       5   638          2378.5    [490, 4803]        6518.5  [784, 21168]         19.08 [0.67, 29.99]           14.87  [5.83, 32.12]
       6   510          2554.5    [359, 4431]        7374.5  [605, 21375]         17.12  [2.74, 33.1]           13.06  [5.43, 31.25]
       7   443          2781.0    [587, 4494]        8198.0 [1001, 21241]         15.71 [1.13, 33.59]           13.47  [5.68, 29.58]
       8   359          2639.0    [298, 4503]        7921.0  [544, 21362]         17.34 [6.17, 34.23]           12.47   [6.42, 23.4]
       9   275          3043.0    [427, 4667]        9451.0  [517, 22278]         16.76 [1.39, 31.34]           14.67  [6.54, 26.76]

Saved QC summary metrics TSV to: /lustre07/scratch/delphine/dataset2_R_output/epi_qc_summary_res0.5_06Aug_165608.tsv
```

# clean UMAP
- we remove the outlier and recluster at various resolutions, again choosing 0.5 

<img width="4200" height="1800" alt="image" src="https://github.com/user-attachments/assets/72ae19d7-2384-47a8-aea9-4935f08fe5c8" />

```
total cells before: 10820 
cluster 4 cells removed: 1421
cells remaining after: 9399
```

## clean UMAP QC check
- island on top left is not a technical outlier
- no more low ribo % clusters

<img width="3600" height="3000" alt="image" src="https://github.com/user-attachments/assets/8ede61bd-34a5-4837-a426-c7fd06341f9e" />

```
per-cluster technical qc summary (resolution 0.5):
 Cluster Cells nFeature_Median nFeature_Range nCount_Median  nCount_Range pct_MT_Median  pct_MT_Range pct_Ribo_Median pct_Ribo_Range
       1  2442          2233.0    [253, 4770]        5896.0  [503, 22819]         18.18    [1, 34.94]           13.55  [4.68, 28.55]
       2  2076          2022.5    [252, 4666]        5172.0  [510, 20803]         15.33 [1.22, 34.19]           18.93  [7.27, 44.24]
       3  1406          2665.5    [298, 4750]        8226.0  [511, 23033]         18.59 [1.43, 34.25]           14.19  [5.65, 28.23]
       4  1316          2363.5    [301, 4793]        6159.0  [531, 23371]         15.94 [1.89, 29.86]           12.83  [4.95, 23.79]
       5   666          2376.0    [414, 4803]        6518.5  [784, 21375]         19.30 [0.67, 29.99]           15.34  [6.24, 32.12]
       6   450          2753.5    [587, 4497]        8200.0 [1001, 21513]         15.78 [1.13, 33.59]           13.61  [5.68, 29.58]
       7   405          2623.0    [359, 4431]        7573.0  [605, 20899]         16.87  [2.74, 33.1]           12.97   [6.44, 27.7]
       8   352          2646.5    [306, 4503]        7966.0  [544, 21362]         17.37 [6.17, 33.46]           12.38   [6.23, 23.4]
       9   286          3011.5    [427, 4667]        9465.0  [517, 23431]         16.74 [1.39, 31.34]           14.85  [6.54, 26.76]
```
# integrated UMAP triad correlation
- similar strong trends as per sample UMAP
- cluster 9 has around 300 cells mainly from sample 62, and some from 63 and 64
- plasticity increases when both sen and prolif increase 

<img width="3300" height="2700" alt="image" src="https://github.com/user-attachments/assets/1b16706b-39e6-47a3-88f3-8c3d78c4ed38" /> \
<img width="1500" height="700" alt="dataset2_Res0 5_Cluster9_direct_multivariate_comparison_27July_2118" src="https://github.com/user-attachments/assets/d2b42c80-0d4d-4b9f-9623-9f28474d11c8" />


# findallmarkers 
- findallmarkers wilcoxon and ROC at Leiden 0.5 for top markers per cluster
- brief summary top log2FC markers passing log2FC>1

<img width="755" height="751" alt="image" src="https://github.com/user-attachments/assets/550d0529-a0a2-45ab-9741-cd7a57ed87d4" />


# run FGSEA
- we run comprehensive FGSEA and targeted FGSEA on senescence panels
- an example graph of top enriched pathways for cluster 9 - the sen prolif plastic cluster
- p adj values are more meaningful here and included in tsv summary

<img width="971" height="807" alt="image" src="https://github.com/user-attachments/assets/7ea3811f-ffe9-454d-9549-de149d3eeebf" />

```
# top positive NES for cluster 9
NES	ES	log2err	pval	padj	size	pathway
2.254	0.888	  1.417	0.000	0.000	 200	                    HALLMARK_E2F_TARGETS
2.170	0.855	  1.230	0.000	0.000	 196	                 HALLMARK_G2M_CHECKPOINT
1.997	0.787	  0.965	0.000	0.000	 200	                 HALLMARK_MYC_TARGETS_V1
1.825	0.783	  0.627	0.000	0.000	  89	                HALLMARK_SPERMATOGENESIS
1.715	0.676	  0.611	0.000	0.000	 196	                HALLMARK_MITOTIC_SPINDLE
1.695	0.686	  0.557	0.000	0.000	 148	                     HALLMARK_DNA_REPAIR
1.556	0.613	  0.498	0.000	0.002	 198	      HALLMARK_OXIDATIVE_PHOSPHORYLATION
1.627	0.739	  0.432	0.002	0.007	  58	                 HALLMARK_MYC_TARGETS_V2
1.312	0.517	  0.238	0.046	0.081	 195	               HALLMARK_MTORC1_SIGNALING
1.288	0.509	  0.211	0.057	0.098	 185	                     HALLMARK_GLYCOLYSIS
1.264	0.514	  0.171	0.088	0.143	 142	          HALLMARK_FATTY_ACID_METABOLISM
1.193	0.506	  0.117	0.183	0.261	  94	        HALLMARK_PI3K_AKT_MTOR_SIGNALING
etc.
```

# monocle 3 trajectories
- we now convert our seurat object to cds to run cell trajectories
- monocle conducts its own cell clustering - run various resolutions
- we use default resolution - which self optimises according to dataset

<img width="3600" height="1650" alt="image" src="https://github.com/user-attachments/assets/ca10245a-017e-435d-a388-f64d797f0ff9" />

## learn graph 
- monocle can only run trajectories on each partition, so we focus on partition 1 - the main partition
- we can also override partitions so that monocle builds trajectories across entire UMAP
- black circles are branch points, grey circles are isolated points - could be progenitors or terminal phenotypes
- any point can be set as a root cell, depending on your analysis

<img width="3600" height="2100" alt="image" src="https://github.com/user-attachments/assets/9cbc7730-c6b1-47e3-93ad-2f553b82a0ba" />

## choose root by lineage 
- to decide which node to set as root for pseudotime, we run lineage panel overlay
- while in non-cancer texts, urothelial cells differentiate from basal and terminate in luminal phenotypes...
- in cancer context, malignant luminal cells can undergo dedifferentiation into basal phenotypes at the invasive front
- so we assume the strongest luminal cluster as the root, and choose the node closest to it
- more objective directions of time can be verified via RNA velocity or possibility inferring CNV, with higher CNV loads indicating later lineages 

<img width="4200" height="3600" alt="image" src="https://github.com/user-attachments/assets/5d99286d-77b3-450c-90a7-0bf8c8190dc1" />

## run pseudotime
- setting the node in the luminal island to be the root, we run pseudotime
- the node numbers are not in any particular order, except that the 1 in white circle is the root
- we can trace the path in any direction to follow the lineage
- the lineage terminates at cluster 9, our senescent-proliferative-plastic phenotype
- however, this cluster is not particularly basal in the lineage panel overlay, though has strong basal markers in initial findallmarkers
- likely a mixed population of luminal and basal, or a transitional phenotype
- there are 3 termination fates (grey circles) - ending somewhere between cluster 1 and 4 on the right edge, cluster 2 on the lower right, and cluster 9 on the lowest middle

<img width="3900" height="1892" alt="image" src="https://github.com/user-attachments/assets/f194a43b-9202-482a-b77a-2c414e317ff8" />
<img width="450" height="350" alt="image" src="https://github.com/user-attachments/assets/3e89d81c-aa49-40ed-b99f-391b966fb611" />
