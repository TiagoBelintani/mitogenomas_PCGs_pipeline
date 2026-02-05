# Mitogenomes & PCGs Phylogenomic Pipeline

This repository documents a **reproducible and general workflow** for downloading mitochondrial genomes from NCBI, extracting mitochondrial protein-coding genes (PCGs), and preparing alignments for phylogenetic analyses.

The pipeline is **dataset-agnostic** and can be applied to any group of organisms with available mitogenomes.

---

## Overview

**Main steps:**

1. Retrieve mitogenomes from NCBI (Entrez)
2. Standardize sequence headers
3. Extract mitochondrial PCGs
4. Organize sequences by gene
5. Multiple sequence alignment (MAFFT)
6. Alignment trimming (trimAl)
7. Concatenation and partitioning (AMAS)
8. Ready-to-use matrices for phylogenetic inference

---

## Requirements

### Software
- `entrez-direct` (EDirect)
- Python ≥ 3.7
- Biopython
- MAFFT
- trimAl
- AMAS
- seqkit (recommended)
- Conda (recommended)

### Example Conda environment
```bash
conda create -n mito_pcgs python=3.9 biopython mafft trimal seqkit
conda activate mito_pcgs
