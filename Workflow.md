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
- entrez-direct (EDirect)
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
```

---

## 1. Downloading Mitogenomes from NCBI (Entrez)

### Using accession numbers
```bash
efetch -db nucleotide -id NC_012345,NC_067890 -format fasta > mitogenomes_raw.fasta
```

### Using a taxonomic query + size
```bash
esearch -db nucleotide -query \
"Triatominae[Organism] AND mitochondrion[Filter] AND 10000:25000[SLEN]" \
| efetch -format gb > mitogenomes.gb
```

---

## 2. Header Standardization

Recommended format:
```
Genus_species_accession
```

Example:
```bash
awk '
  /^>/ {
    acc=$1; gsub(/^>/,"",acc);
    genus=$2;
    species=$3;
    gsub(/\.1$/,"",acc);
    print ">"genus"_"species"_"acc;
    next
  }
  { print }
' mitogenomes_raw.fasta > mitogenomes_renamed.fasta
```

---

## 3. Gene Annotation and PCG Extraction

- If annotations exist: extract directly from GenBank files
- If annotations are missing: reannotate using a mitochondrial annotation tool (e.g. MITOS2)

Typical mitochondrial PCGs:
```
ATP6 ATP8 COI COII COIII CYTB ND1 ND2 ND3 ND4 ND4L ND5 ND6
```

---

## 4. Organizing PCGs by Gene

Directory structure:
```
by_gene/
├── ATP6.fasta
├── ATP8.fasta
├── COI.fasta
├── COII.fasta
├── COIII.fasta
├── CYTB.fasta
├── ND1.fasta
├── ND2.fasta
├── ND3.fasta
├── ND4.fasta
├── ND4L.fasta
├── ND5.fasta
├── ND6.fasta
```

---

## 5. Multiple Sequence Alignment (MAFFT)

```bash
mkdir aligned_by_gene

for f in by_gene/*.fasta; do
  gene=$(basename "$f")
  mafft --auto "$f" > aligned_by_gene/$gene
done
```

---

## 6. Alignment Trimming (trimAl)

### Strict trimming
```bash
mkdir trimmed_by_gene

for f in aligned_by_gene/*.fasta; do
  gene=$(basename "$f")
  trimal -in "$f" -out trimmed_by_gene/$gene -nogaps
done
```

---

## 7. Concatenation and Partitioning (AMAS)

```bash
cd trimmed_by_gene

python AMAS.py concat \
  -i *.fasta \
  -f fasta \
  -d dna \
  -u fasta \
  -y raxml \
  -t ../concatenated/mito_concat.fasta \
  -p ../concatenated/mito_partitions.txt
```

---

## 8. Quality Control

```bash
for f in trimmed_by_gene/*.fasta; do
  echo -n "$(basename $f .fasta): "
  grep -c "^>" "$f"
done
```

```bash
seqkit stats concatenated/mito_concat.fasta
```

---

## 9. Downstream Phylogenetic Analyses

Example (IQ-TREE):
```bash
iqtree2 -s mito_concat.fasta -p mito_partitions.txt -m MFP -B 1000 -nt AUTO
```

---

## Notes on Reproducibility

- Keep raw data unchanged
- Document software versions and parameters
- Run each step explicitly

---

## License

Academic and research use. Reuse with attribution.
