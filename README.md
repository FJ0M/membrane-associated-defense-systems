# Membrane-Associated Anti-Phage Defense Systems

Code and data for the bioinformatic analysis in:

> Hu, H.\*, Songailiene, I.\*, Rutbeek, N.R., Šiborová, M., Martin, F.J.O., Taylor, N.M.I.
> **Bacterial anti-phage defense at the membrane boundary.**
> *(forthcoming)*

This repository reproduces **Figure 1** of the paper, which quantifies the prevalence and distribution of membrane-associated defense systems (MADS) across bacterial genomes by combining the [DefenseFinder](https://defensefinder.mdmlab.fr/) database with transmembrane topology predictions from [DeepTMHMM](https://dtu.biolib.com/DeepTMHMM).

---

## Overview

Of the 172,333 annotated anti-phage defense systems in the DefenseFinder database, approximately 7,916 (~5%) are predicted to contain at least one membrane-associated component. The analysis pipeline:

1. **Collects** defense system annotations from the DefenseFinder RefSeq database and, independently, from the PADLOC server.
2. **Retrieves** protein sequences for all component proteins via NCBI Entrez.
3. **Clusters** sequences at 85% identity / 70% coverage using [MMseqs2](https://github.com/soedinglab/MMseqs2) to reduce redundancy.
4. **Predicts** transmembrane regions on cluster representatives using [DeepTMHMM](https://dtu.biolib.com/DeepTMHMM).
5. **Propagates** membrane annotations back to all members, aggregates per defense system, and generates summary plots.

---

## Repository Structure

| File / Directory | Description |
|---|---|
| `DefenseFinder-Cluster85-DeepTMHMM.ipynb` | **Main pipeline.** Loads DefenseFinder data, fetches protein sequences from NCBI, clusters with MMseqs2 (85% identity), runs DeepTMHMM on cluster representatives, and merges transmembrane predictions back to produce `defensefinder_tmhmm_results.csv`. |
| `DefenseFinder_clustering_and_sampling.ipynb` | **Sampling-based analysis.** Samples up to 100 systems per defense subtype, exports FASTA files for DeepTMHMM, parses results, and produces `df_DefenseFinder_DeepTMHMM_sampled_results.xlsx`. |
| `Padloc_download.ipynb` | **PADLOC data acquisition.** Scrapes the PADLOC web server to download defense system annotations for all RefSeq genomes (~223,000 assemblies) and consolidates them into a single CSV. |
| `plotting.ipynb` | **Figure generation.** Reads `defensefinder_tmhmm_results.csv` and produces the plots used in Figure 1 of the paper. |
| `defensefinder_tmhmm_results.csv` | Merged output: one row per defense system with columns for membrane protein count and membrane-association flag. |
| `df_DefenseFinder_DeepTMHMM_sampled_results.xlsx` | Results from the sampling-based analysis (subtype-stratified). |
| `membrane_defensefinder.xlsx` | Full merged DefenseFinder + DeepTMHMM dataset exported from `plotting.ipynb`. |
| `proteins_for_deeptmhmm.fasta` | FASTA file of all sampled protein sequences used as DeepTMHMM input. |
| `proteins_for_deeptmhmm_1.fasta` | First half of the sampled sequences (split for batch submission). |
| `proteins_for_deeptmhmm_2.fasta` | Second half of the sampled sequences. |

### Large files on Zenodo

The following files are too large for GitHub and are deposited on Zenodo:

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.21010793.svg)](https://doi.org/10.5281/zenodo.21010793)

| File | Description |
|---|---|
| `input_sequences.fasta` | All protein sequences retrieved from NCBI for DefenseFinder-annotated defense systems. |
| `df-sequences.csv` | Expanded DefenseFinder table with one row per protein, including fetched sequences (~380,000 rows). |
| `results.zip` | DeepTMHMM output files (`TMRs.gff3`, `predicted_topologies.3line`) for all batches. |

Download these files and place them alongside the notebooks before running.

---

## Workflow

```
DefenseFinder RefSeq DB (df-RefSeq.csv)
        │
        ▼
  Expand multi-protein systems (explode comma-separated accessions)
        │
        ▼
  Fetch protein sequences from NCBI Entrez ──► df-sequences.csv / input_sequences.fasta
        │
        ▼
  Cluster at 85% identity with MMseqs2
        │
        ▼
  Predict TM helices on cluster reps with DeepTMHMM ──► results/
        │
        ▼
  Propagate TM annotations to all cluster members
        │
        ▼
  Aggregate per defense system (any TM protein → membrane-associated)
        │
        ▼
  defensefinder_tmhmm_results.csv ──► plotting.ipynb ──► Figure 1
```

---

## Generated Figures

| Plot | Description |
|---|---|
| Percentage of Membrane-Associated Anti-Phage Defense Systems | Bar chart of membrane-association prevalence by phylum, with the global average (~4.6%) shown as a dashed line. |
| Percentage of Membrane-Association by Defense System Type (≥5%) | Bar chart showing which defense system families are most frequently membrane-associated. |
| Percentage of Membrane-Association in CRISPR-Cas Subtypes (≥0.1%) | Focused bar chart for CRISPR-Cas subtypes only. |
| Distribution of Defense Systems per Genome | Stacked histogram of membrane vs. non-membrane defense systems per genome. |

---

## Requirements

- Python 3.9+
- Jupyter Notebook / JupyterLab

### Python packages

```
pandas
numpy
seaborn
matplotlib
biopython
beautifulsoup4
tqdm
joblib
openpyxl
```

### External tools

- [MMseqs2](https://github.com/soedinglab/MMseqs2) — for sequence clustering
- [DeepTMHMM](https://dtu.biolib.com/DeepTMHMM) — for transmembrane helix prediction (academic license)

---

## Usage

### 1. Clone and set up

```bash
git clone https://github.com/FJ0M/membrane-associated-defense-systems.git
cd membrane-associated-defense-systems
pip install pandas numpy seaborn matplotlib biopython beautifulsoup4 tqdm joblib openpyxl
```

### 2. Download large files from Zenodo

Download `input_sequences.fasta`, `df-sequences.csv`, and `results.zip` from [https://doi.org/10.5281/zenodo.21010793](https://doi.org/10.5281/zenodo.21010793) and place them in the repository root. Unzip `results.zip`.

### 3. Obtain an NCBI API key

The sequence retrieval notebooks use the NCBI Entrez API. To avoid rate limiting, obtain a free API key from [https://www.ncbi.nlm.nih.gov/account/](https://www.ncbi.nlm.nih.gov/account/) and set it in the notebook:

```python
Entrez.email = "your.email@example.com"
Entrez.api_key = "YOUR_API_KEY"
```

### 4. Run the notebooks

Run the notebooks in this order:

1. **`Padloc_download.ipynb`** — Downloads PADLOC annotations (optional; independent analysis).
2. **`DefenseFinder-Cluster85-DeepTMHMM.ipynb`** — Main pipeline: sequence retrieval → clustering → DeepTMHMM → merged results.
3. **`DefenseFinder_clustering_and_sampling.ipynb`** — Alternative sampling-based analysis.
4. **`plotting.ipynb`** — Generates all figures from the merged results.

> **Note:** Steps involving NCBI sequence retrieval and DeepTMHMM prediction are computationally intensive and may take several hours. Pre-computed results are available on Zenodo.

---

## Citation

If you use this code or data, please cite:

```
Hu, H.*, Songailiene, I.*, Rutbeek, N.R., Šiborová, M., Martin, F.J.O., Taylor, N.M.I.
Bacterial anti-phage defense at the membrane boundary.
(forthcoming)
```

---

## License

This project is licensed under the [MIT License](LICENSE).
