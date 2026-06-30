# AcetylBench

A deduplicated benchmark dataset for lysine acetylation site prediction, with cross-database evaluation code.

This repository accompanies the paper:

> **AcetylBench: A Deduplicated Benchmark Dataset Reveals Systematic Performance Inflation and Cross-Database Generalization Failure in Lysine Acetylation Site Prediction**  
> Mohd Uzair Ashraf, Shaista Khan, Rizwan Hasan Khan  
> Interdisciplinary Biotechnology Unit, Aligarh Muslim University, Aligarh 202002, India  
> *BMC Bioinformatics* (under review)

---

## Dataset

The AcetylBench training corpus and pre-computed ESM-1v embeddings are available at Zenodo:

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.19475598.svg)](https://doi.org/10.5281/zenodo.19475598)

| File | Sequences | Description |
|---|---|---|
| `AcetylBench_train_positive.csv` | 27,182 | Acetylated lysine windows (31-mer, dbPTM-derived, CD-HIT filtered at 40%) |
| `AcetylBench_train_negative.csv` | 7,063 | Non-acetylated lysine windows (same proteins, same filtering) |

Pre-computed ESM-1v embeddings (mean pool and lysine-position representations) for the AcetylBench, DeepAcet training, and independent test splits are also deposited at the same Zenodo record, enabling full reproduction of all classification experiments without GPU access.

The independent test set (3,221 positive + 3,221 negative) is from the DeepAcet study (CPLM database): https://github.com/Lab-Xu/DeepAcet

---

## Repository structure

```
AcetylBench/
├── notebooks/
│   ├── 01_dataset_curation_proteinbert_clustering.ipynb
│   ├── 02_cdhit_deduplication.ipynb
│   ├── 03_classification_experiments_esm1v.ipynb
│   └── 04_forensic_analysis_original_pipeline.ipynb
├── models/
│   ├── pos_pca_model_positive.pkl
│   ├── neg_pca_model_negative.pkl
│   └── acetylation_model_1.pth
├── distinct_positive_sequences.csv
├── distinct_negative_sequences.csv
├── distinct_positive_clean.csv
├── distinct_negative_clean.csv
└── README.md
```

---

## Notebooks

| Notebook | Description |
|---|---|
| `01_dataset_curation_proteinbert_clustering.ipynb` | ProteinBERT global embeddings → IncrementalPCA → MiniBatchKMeans (k=2) → pure cluster selection |
| `02_cdhit_deduplication.ipynb` | CD-HIT 40% identity filtering of positive and negative sequence sets |
| `03_classification_experiments_esm1v.ipynb` | ESM-1v embedding experiments (pre-computed embeddings loaded directly from Zenodo), combined feature baselines (CKSAAP + BLOSUM62 + one-hot), PCA dimensionality sensitivity analysis, CPLM→dbPTM asymmetry analysis |
| `04_forensic_analysis_original_pipeline.ipynb` | Reproduction of original dual-PCA pipeline on DeepAcet independent test; documents three sources of metric inflation |

Notebook 03 runs on CPU or GPU since embeddings are pre-computed and downloaded directly from Zenodo. Notebooks 01, 02, and 04 require ProteinBERT and were originally run on Google Colab Pro with an NVIDIA A100 GPU; an A100 is only required for regenerating embeddings from raw sequences.

---

## Key results

| Method | Training DB | CV AUC | Ind. Test AUC | Ind. MCC |
|---|---|---|---|---|
| LR (CKSAAP+BLOSUM+OH) | dbPTM | — | 0.582 | 0.139 |
| ESM-1v Mean | dbPTM | 0.867 | 0.590 | 0.139 |
| ESM-1v K-pos | dbPTM | 0.861 | 0.607 | 0.181 |
| CKSAAP+MLP | CPLM | 0.629 | 0.640 | 0.216 |
| ESM-1v Mean | CPLM | 0.668 | 0.670 | 0.248 |
| **ESM-1v K-pos** | **CPLM** | **0.704** | **0.705** | **0.305** |
| ESM-1v K-pos | CPLM→dbPTM | — | 0.729 | 0.271 |
| DeepAcet (reference) | CPLM | 0.854 | 0.841 | 0.698 |

Independent test set: DeepAcet test set, 3,221 positive + 3,221 negative sequences (CPLM-sourced).

---

## Dataset construction summary

Raw dbPTM lysine acetylation windows: 416,553 total sequences (208,276 positive, 208,277 negative)

1. ProteinBERT global embeddings → StandardScaler → IncrementalPCA (50 components) → MiniBatchKMeans (k=2) → pure cluster selection: **72,843 positive + 72,843 negative**
2. CD-HIT 40% identity threshold (version 4.8.1, `-n 2`), positive and negative sets clustered independently: **27,182 positive + 7,063 negative**

Redundancy removed: 63.1% of positives, 90.3% of negatives.

---

## Requirements

```
python >= 3.10
torch >= 2.0
fair-esm
proteinbert
scikit-learn >= 1.0
pandas
numpy
h5py
matplotlib
seaborn
cd-hit >= 4.8.1
```

`fair-esm` and an A100 GPU are only required if regenerating ESM-1v embeddings from raw sequences. Notebook 03 downloads pre-computed embeddings directly and does not require either.

Install ESM (optional, only for regenerating embeddings):
```bash
pip install fair-esm
```

Install ProteinBERT (required for Notebooks 01 and 04):
```bash
git clone https://github.com/nadavbra/protein_bert.git
cd protein_bert && python setup.py install
```

---

## Citation

If you use AcetylBench in your research, please cite:

```bibtex
@article{ashraf2025acetylbench,
  title     = {AcetylBench: A Deduplicated Benchmark Dataset Reveals Systematic 
               Performance Inflation and Cross-Database Generalization Failure 
               in Lysine Acetylation Site Prediction},
  author    = {Ashraf, Muhammad Uzair and Khan, Shaista and Khan, Rizwan Hasan},
  journal   = {BMC Bioinformatics},
  year      = {2026},
  note      = {Under review}
}
```

---

## License

The code in this repository is released under the MIT License.  
The dataset (Zenodo DOI: 10.5281/zenodo.19475598) is released under CC BY 4.0.

---

## Contact

Rizwan Hasan Khan — rhkhan.cb@amu.ac.in  
Interdisciplinary Biotechnology Unit, Aligarh Muslim University, Aligarh 202002, India
