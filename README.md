# Contextual Hallucination Detector (CHD)

A research pipeline for detecting and classifying contextual hallucinations in LLM outputs, with cross-domain generalization experiments across Apple, X (Twitter), and Meta privacy/policy documents.

## Overview

This repository implements the **Contextual Hallucination Detector (CHD)** and its extensions, culminating in the **Domain-Adaptive Evidence Calibration CHD (DAEC-CHD)**. The system classifies claims about policy documents into four categories:

| Label | Description |
|-------|-------------|
| `SUPPORTED` | Claim is directly supported by the context |
| `UNSUPPORTED` | Claim is not addressed in the context |
| `CONTRADICTED` | Context explicitly contradicts the claim |
| `HALLUCINATED` | Claim contains fabricated information |

## Repository Structure

```
chd-repo/
├── notebooks/
│   ├── 01_dataset_analysis.ipynb       # Dataset EDA across Apple, X, Meta
│   ├── 02_chd_pipeline.ipynb           # Core CHD pipeline + conference extensions
│   └── 03_ae_chd_daec.ipynb            # AE-CHD, DAEC-CHD, and final experiments
├── data/
│   ├── apple_privacy_hallucination_10000.csv   # Apple Privacy Policy dataset
│   ├── x_privacy_policy_10000.csv              # X (Twitter) Privacy Policy dataset
│   └── meta_human_rights_policy_10000.csv      # Meta Human Rights Policy dataset
├── outputs/                            # Generated artifacts (gitignored)
├── requirements.txt
├── .gitignore
└── README.md
```

## Notebooks

### `01_dataset_analysis.ipynb` — Dataset Analysis
Comprehensive EDA across all three domain datasets (Apple, X, Meta):
- Label distribution and class balance checks
- Missing value analysis
- Topic, difficulty, and generation-type distributions
- Text length statistics by label
- Duplicate and label-conflict detection
- Lexical overlap analysis
- Cross-domain statistics and combined dataset construction

### `02_chd_pipeline.ipynb` — CHD Pipeline
Full experimental pipeline for the core CHD model:
- Data loading, cleaning, and normalization
- Source-fact grouping to prevent data leakage
- Group-safe train/val/test splits
- Feature extraction: 12 handcrafted features (NLI scores, semantic similarity, lexical overlap, entity/number consistency, negation detection)
- XGBoost classifier with Platt calibration
- Confidence analysis, abstention curves, and error analysis
- Contextual Hallucination Score (CHS)
- Ablation study and progressive context degradation
- Conference-ready extensions (§47–§53): baseline comparisons, statistical significance, cross-domain zero-shot, SHAP interpretability, robustness tests, latency benchmarks

### `03_ae_chd_daec.ipynb` — AE-CHD & DAEC-CHD
Extended pipeline building on the core CHD:

| Section | Content |
|---------|---------|
| §54 | Controlled Contextual Degradation Stress Test (0–80% corruption, 3 perturbation modes) |
| §55 | **AE-CHD**: Adaptive Evidence-Aware CHD with sentence-level evidence localization and sufficiency estimation (ESS) |
| §56 | **DAEC-CHD**: Domain-Adaptive Evidence Calibration with source-only CDF calibration, soft routing, and uncertainty-aware weighting |
| §57 | External reference baselines (MoritzLaurer DeBERTa zero-shot, Vectara HHEM-2.1) |
| §58 | Paired significance tests (bootstrap CI, McNemar, Wilcoxon, Holm correction) |
| §59 | Paper-ready table export and Core-B readiness checklist |
| §60–62 | Improvement experiments: weight sensitivity, HALLUCINATED deep analysis, temperature sensitivity |
| Option B | Single-domain vs cross-domain stability for NLI fine-tuning |
| Part 4 | DAEC-DeBERTa (fully standalone) |

## Model Architecture

```
Context + Claim
    ├── SentenceTransformer (all-MiniLM-L6-v2)   → semantic similarity
    ├── NLI model (DeBERTa-v3-base)               → entailment / contradiction / neutral scores
    └── Handcrafted features                       → lexical overlap, entity/number consistency,
                                                      negation mismatch, context coverage

12 CHD features → XGBoost → Platt calibration → 4-class output
```

**AE-CHD** adds 16 evidence features via sentence-level localization.  
**DAEC-CHD** further adds source-only CDF calibration of ESS and soft routing.

## Cross-Domain Protocol

Three Leave-One-Domain-Out (LODO) experiments, each repeated over 5 random seeds:

| Experiment | Train | Test |
|-----------|-------|------|
| 1 | Apple + X | Meta |
| 2 | Apple + Meta | X |
| 3 | X + Meta | Apple |

Results reported as mean ± std with Student-t 95% confidence intervals.

## Setup

### Requirements

```bash
pip install -r requirements.txt
```

### Data

Place the three CSV files in the `data/` directory:
- `apple_privacy_hallucination_10000.csv`
- `x_privacy_policy_10000.csv`
- `meta_human_rights_policy_10000.csv`

Each CSV must contain these columns:

```
id, document, document_version, topic, question, context,
claim, label, evidence, source_lines, generation_type, difficulty, variant_id
```

### Running

Run the notebooks **in order**:

```bash
jupyter notebook
# 1. notebooks/01_dataset_analysis.ipynb
# 2. notebooks/02_chd_pipeline.ipynb
# 3. notebooks/03_ae_chd_daec.ipynb
```

> **Note:** `03_ae_chd_daec.ipynb` is modular — cells §54+ can be run independently if the required objects from the core pipeline are in memory. Cell §60 (Improvement Experiments) and Part 4 (DAEC-DeBERTa) are fully standalone.

## Key Dependencies

| Package | Purpose |
|---------|---------|
| `sentence-transformers` | Semantic similarity features |
| `transformers` | DeBERTa NLI model |
| `xgboost` | Gradient boosting classifier |
| `torch` | GPU-accelerated inference |
| `scikit-learn` | Calibration, metrics, baselines |
| `scipy` | Statistical tests |

## Reproducibility

- Global seed: `SEED = 42`
- Multi-seed experiments use seeds: `[42, 123, 456, 789, 1024]`
- Group-safe splits prevent source-fact leakage across train/val/test
- All intermediate artifacts (splits, features, models) are saved to `outputs/`

## Citation

If you use this code, please cite accordingly.
