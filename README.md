# VGP Genome Assembly — *Saccharomyces cerevisiae* S288C

## Overview
This repository contains the results of a **Vertebrate Genome Project (VGP)** genome assembly pipeline completed using [Galaxy Europe](https://usegalaxy.eu). The assembly was performed on synthetic HiFi reads of *Saccharomyces cerevisiae* S288C, a widely used eukaryotic model organism.

The pipeline integrates three sequencing technologies:
- **PacBio HiFi reads** — for contig assembly
- **Bionano optical maps** — for hybrid scaffolding
- **Illumina Hi-C reads** — for chromosome-scale scaffolding

---

## Table of Contents
1. [Introduction](#introduction)
2. [Methods](#methods)
3. [Results](#results)
   - [Genome Profile Analysis](#1-genome-profile-analysis)
   - [HiFi Assembly with Hifiasm](#2-hifi-assembly-with-hifiasm)
   - [Assembly QC — BUSCO](#3-assembly-qc--busco)
   - [Assembly QC — Merqury](#4-assembly-qc--merqury)
   - [Bionano Hybrid Scaffolding](#5-bionano-hybrid-scaffolding)
   - [Hi-C Scaffolding with YaHS](#6-hi-c-scaffolding-with-yahs)
   - [Final Assembly Statistics](#7-final-assembly-statistics)
4. [Conclusion](#conclusion)
5. [File Structure](#file-structure)
6. [References](#references)

---

## Introduction
The **Vertebrate Genomes Project (VGP)**, launched by the Genome 10K (G10K) consortium, aims to generate high-quality, near-error-free, gap-free, chromosome-level, haplotype-phased genome assemblies for every vertebrate species.

In this tutorial, *S. cerevisiae* S288C was used as a model organism due to its well-characterized genome (~12 Mb haploid size, 16 chromosomes), allowing precise evaluation of assembly quality. A set of synthetic HiFi reads corresponding to a theoretical diploid genome was used.

---

## Methods

### Tools Used
| Tool | Version | Purpose |
|------|---------|---------|
| Cutadapt | 4.4+galaxy0 | HiFi adapter trimming |
| Meryl | 1.3+galaxy6 | K-mer counting |
| GenomeScope2 | 2.0+galaxy2 | Genome profiling |
| Hifiasm | 0.19.8+galaxy0 | HiFi genome assembly |
| gfastats | 1.3.6+galaxy0 | Assembly statistics |
| BUSCO | 5.5.0+galaxy0 | Assembly completeness |
| Merqury | 1.3+galaxy3 | K-mer based QC |
| Bionano Hybrid Scaffold | 3.7.0+galaxy3 | Optical map scaffolding |
| BWA-MEM2 | 2.2.1+galaxy1 | Hi-C read mapping |
| Filter and merge | 1.0+galaxy1 | Hi-C chimeric read filtering |
| PretextMap | 0.1.9+galaxy0 | Hi-C contact map generation |
| YaHS | 1.2a.2+galaxy1 | Hi-C scaffolding |

### Pipeline Overview
The assembly pipeline consisted of four main stages:
1. **Genome profile analysis** — K-mer profiling with Meryl + GenomeScope2
2. **HiFi assembly** — Hi-C phased assembly with Hifiasm
3. **Bionano scaffolding** — Hybrid scaffolding with optical maps
4. **Hi-C scaffolding** — Chromosome-scale scaffolding with YaHS

### Input Data
| Dataset | Format | Source |
|---------|--------|--------|
| HiFi reads (3 files) | FASTA | Zenodo: 6098306 |
| Hi-C forward reads | FASTQ.GZ | Zenodo: 5550653 |
| Hi-C reverse reads | FASTQ.GZ | Zenodo: 5550653 |
| Bionano optical map | CMAP | Zenodo: 5887339 |

---

## Results

### 1. Genome Profile Analysis

K-mer profiling was performed using **Meryl** (k=31) followed by **GenomeScope2** to estimate genome properties from the k-mer frequency histogram.

#### GenomeScope2 Summary
| Property | Min | Max |
|----------|-----|-----|
| Genome Haploid Length | 11,739,513 bp | 11,747,352 bp |
| Genome Repeat Length | 723,114 bp | 723,597 bp |
| Genome Unique Length | 11,016,399 bp | 11,023,756 bp |
| Heterozygosity | 0.5759% | 0.5835% |
| Model Fit | 92.52% | 96.52% |
| Read Error Rate | 0.00094% | 0.00094% |

**Estimated genome size used for assembly: 11,747,160 bp**

#### GenomeScope Plots
![GenomeScope Linear Plot](images/genomescope_linear.png)
![GenomeScope Log Plot](images/genomescope_log.png)

The k-mer profile shows a bimodal distribution consistent with a diploid genome. The heterozygous peak is at ~25x coverage and the homozygous peak at ~50x coverage.

---

### 2. HiFi Assembly with Hifiasm

Hifiasm was run in **Hi-C phased mode**, producing two phased haplotype assemblies.

#### gfastats Assembly Statistics
| Statistic | Hap1 | Hap2 |
|-----------|------|------|
| Number of contigs | 17 | 16 |
| Total length | ~12.16 Mb | ~11.30 Mb |
| Scaffold N50 | 923 KB | 922 KB |
| Contig N50 | 923 KB | 922 KB |
| Percent gaps | 0.000% | 0.000% |

---

### 3. Assembly QC — BUSCO

BUSCO was run using the **Saccharomycetes** lineage database (odb10, 2137 genes).

#### BUSCO Results Summary
| Metric | Hap1 | Hap2 |
|--------|------|------|
| Complete BUSCOs (C) | 2039 (95.4%) | 1898 (88.8%) |
| Complete & Single-copy (S) | 1999 (93.5%) | 1867 (87.4%) |
| Complete & Duplicated (D) | 40 (1.9%) | 31 (1.5%) |
| Fragmented (F) | 68 (3.2%) | 58 (2.7%) |
| Missing (M) | 30 (1.4%) | 181 (8.5%) |
| Total BUSCOs searched | 2137 | 2137 |

#### BUSCO Assessment Images
![BUSCO Hap1](images/busco_hap1.png)
![BUSCO Hap2](images/busco_hap2.png)

Hap1 shows excellent completeness at **95.4%**, indicating a high-quality assembly. Hap2 shows slightly lower completeness at **88.8%**, which is expected for the second haplotype.

---

### 4. Assembly QC — Merqury

Merqury provides reference-free assessment of assembly quality using k-mer analysis.

#### Merqury Completeness Statistics
| Assembly | K-mers Found | Total K-mers | Completeness |
|----------|-------------|--------------|--------------|
| Hap1 (assembly_01) | 11,611,483 | 13,010,260 | 89.25% |
| Hap2 (assembly_02) | 10,792,811 | 13,010,260 | 82.96% |
| Both combined | 13,010,244 | 13,010,260 | **99.99%** |

#### Merqury QV Statistics
| Assembly | QV Score |
|----------|----------|
| Hap1 | 79.74 |
| Both | 82.60 |

#### Merqury Spectra Plots
![Merqury Spectra CN](images/merqury_spectra_cn.png)
![Merqury Spectra ASM](images/merqury_spectra_asm.png)

The spectra-cn plot shows a clean bimodal distribution with diploid coverage at ~50x. Combined completeness of **99.99%** confirms near-complete genome recovery across both haplotypes.

---

### 5. Bionano Hybrid Scaffolding

Bionano optical maps were integrated with the Hap1 assembly using the **Bionano Hybrid Scaffold** tool in VGP mode.

#### Bionano Scaffolding Statistics
| Metric | Value |
|--------|-------|
| Conflict cuts to Bionano maps | 0 |
| Conflict cuts to NGS sequences | 0 |
| Hybrid scaffold count | 16 |
| Hybrid scaffold N50 | **923 KB** |
| Hybrid scaffold total length | 12.075 Mb |
| Max scaffold length | 1.532 Mb |

Zero conflicts between Bionano and HiFi assemblies indicates excellent agreement between the two technologies.

---

### 6. Hi-C Scaffolding with YaHS

Hi-C reads were mapped to the Bionano-scaffolded assembly using **BWA-MEM2**, then used for chromosome-scale scaffolding with **YaHS**.

#### Hi-C Contact Maps

**Before YaHS scaffolding:**
![Contact Map Before YaHS](images/contact_map_before_yahs.png)

**After YaHS scaffolding:**
![Contact Map After YaHS](images/contact_map_after_yahs.png)

The contact maps show 17 distinct scaffolds corresponding to the haploid chromosomes of *S. cerevisiae*. After YaHS scaffolding, contact signals are concentrated along the diagonal, indicating correct contig ordering and orientation.

---

### 7. Final Assembly Statistics

| Metric | Value |
|--------|-------|
| Final scaffold count | 17 |
| Expected chromosomes | 16 + mitochondrial DNA |
| Final assembly N50 | ~923 KB |
| Total assembly length | ~12.16 Mb |
| Reference genome size | ~12.16 Mb |
| BUSCO completeness | 95.4% |
| Merqury combined completeness | 99.99% |

---

## Conclusion

The VGP assembly pipeline successfully generated a high-quality, chromosome-level genome assembly of *S. cerevisiae* S288C. Key achievements:

- ✅ **Near-complete assembly** — 99.99% k-mer completeness (Merqury)
- ✅ **High BUSCO score** — 95.4% complete BUSCOs in Hap1
- ✅ **Zero scaffolding conflicts** — perfect agreement between HiFi and Bionano
- ✅ **Chromosome-level scaffolds** — 17 scaffolds matching expected genome structure
- ✅ **High contiguity** — N50 of 923 KB

The final assembly is comparable to the *S. cerevisiae* S288C reference genome, demonstrating the effectiveness of the VGP pipeline for generating reference-quality genome assemblies.

---

## File Structure

```
VGP-Genome-Assembly/
│
├── README.md
│
├── images/
│   ├── busco_hap1.png
│   ├── busco_hap2.png
│   ├── merqury_spectra_cn.png
│   ├── merqury_spectra_asm.png
│   ├── genomescope_linear.png
│   ├── genomescope_log.png
│   ├── contact_map_before_yahs.png
│   └── contact_map_after_yahs.png
│
├── assembly_stats/
│   ├── busco_hap1_summary.txt
│   ├── busco_hap2_summary.txt
│   ├── gfastats_hap1_hap2.txt
│   ├── merqury_completeness.txt
│   └── merqury_qv_stats.txt
│
└── assembly_files/
    ├── hap1_contigs.fasta
    ├── hap2_contigs.fasta
    ├── hap1_bionano_scaffolds.fasta
    └── final_assembly_yahs.fasta
```

---

## References

- Rhie, A. et al. (2021). Towards complete and error-free genome assemblies of all vertebrate species. *Nature*, 592, 737–746.
- Cheng, H. et al. (2021). Haplotype-resolved de novo assembly using phased assembly graphs with hifiasm. *Nature Methods*, 18, 170–175.
- Ranallo-Benavidez, T.R. et al. (2020). GenomeScope 2.0 and Smudgeplot for reference-free profiling of polyploid genomes. *Nature Communications*, 11.
- Rhie, A. et al. (2020). Merqury: reference-free quality, completeness, and phasing assessment for genome assemblies. *Genome Biology*, 21.
- Simão, F.A. et al. (2015). BUSCO: assessing genome assembly and annotation completeness with single-copy orthologs. *Bioinformatics*, 31, 3210–3212.
- Formenti, G. et al. (2022). Gfastats: conversion, evaluation and manipulation of genome sequences using assembly graphs. *Bioinformatics*, 38, 4214–4216.
- Zhou, C. et al. (2022). YaHS: yet another Hi-C scaffolding tool. *Bioinformatics*, 39.
- Lariviere, D. et al. (2022). VGP assembly pipeline (Galaxy Training Materials). https://training.galaxyproject.org
