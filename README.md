# ConfRAG-ASR
## Retrieval-Augmented Post-Correction of Clinical ASR with Confidence-Guided Anti-Hallucination Filtering

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1YerxafzO61egS7R45xFTabQ-_3xncJKg?usp=sharing)

---

## Overview

ConfRAG-ASR is a **training-free** post-correction pipeline for clinical Automatic Speech Recognition (ASR). It corrects medical terminology errors produced by Whisper while guaranteeing zero hallucination rate — a critical requirement for safe clinical deployment.

---

## Pipeline (5 Stages)

```
Audio → [1] Frozen Whisper ASR
             ↓ avg_logprob < θ
        [2] Medical-Aware Confidence Gate
             ↓ flagged medical words only
        [3] FAISS Domain Retrieval (41,685 terms)
             ↓ top-5 candidates
        [4] Constrained Similarity Corrector
             ↓ semantic × phonetic score
        [5] Anti-Hallucination Filter (5 layers)
             ↓
        Corrected Transcript
```

---

## Knowledge Base

| Source | Content | Terms |
|--------|---------|-------|
| MIMIC-III Clinical Database Demo v1.4 | Diseases, procedures, drugs | ~20K |
| RxNorm API (NLM) | Drug names | ~22K |
| **Total** | | **~41,685** |

---

## Evaluation: Controlled Terminology Corruption Benchmark (CTCB)

| Property | Value |
|----------|-------|
| Total terms | 500 |
| Diseases | 167 |
| Drugs | 167 |
| Procedures | 166 |
| Corruption levels | 3 (Mild / Moderate / Severe) |
| Samples | 497 |

---

## Key Results

| Level | TER (Baseline) | TER (ConfRAG) | Reduction | F1 | HR |
|-------|---------------|---------------|-----------|----|----|
| L1 Mild | 0.931 | 0.219 | **76.5%** | 0.501 | **0.000** |
| L2 Moderate | 0.979 | 0.631 | **35.5%** | 0.241 | **0.000** |
| L3 Severe | 1.000 | 0.927 | **7.3%** | 0.079 | **0.000** |


> **TER** = Terminology Error Rate (phrase-level)  
> **HR** = Hallucination Rate — zero across all levels ✅

---

## Ablation Study

| System | Description | WER (L1) | TER (L1) |
|--------|-------------|----------|----------|
| S1 | Corrupted baseline | 0.096 | 0.931 |
| S2 | RAG only (no gate, no AHF) | 0.096 | 0.554 |
| S3 | Corrector only (no AHF) | **1.006** | 0.176 |
| S4 | **Full ConfRAG-ASR ★** | **0.090** | **0.219** |

> S3 achieves lower TER but WER > 1.0 — clinically unusable.  
> S4 is the only configuration safe for clinical deployment.

---

## Benchmark Validation

CTCB error patterns were validated against 102 real Whisper errors:

| Error Type | Whisper % | CTCB % | Diff |
|---|---|---|---|
| Phonetic Substitution | 64.7% | 66.0% | **1.3%** |
| Word Splitting | 29.4% | 19.0% | 10.4% |
| Syllable Deletion | 5.9% | 0.0% | 5.9% |
| **TVD** | — | — | **0.163** |

> TVD = 0.163 → Moderate structural similarity ✅

---

## How to Run

1. Open in Colab via the badge above
2. Run **Cell 1** (Install Dependencies)
3. Run **Cell 2** (CONFIG)
4. Run **Cell 3** (Download MIMIC-III)
5. Run **Cell 4** (Extract Terms)
6. Run **Cell 5** (Build FAISS Index)
7. Run **Cell 6–14** (CTCB Benchmark + Ablation)
8. Run **BV-1 → BV-7** (Benchmark Validation)

> If `numpy.dtype size changed` error appears at Cell 1,  
> run this first then restart:
> ```python
> import subprocess, sys, os
> subprocess.run([sys.executable,'-m','pip','install','-q',
>                 '--upgrade','numpy==1.26.4'], check=True)
> os.kill(os.getpid(), 9)
> ```

---

## Requirements

```
openai-whisper
sentence-transformers
faiss-cpu
rapidfuzz
jiwer
datasets
pandas
requests
matplotlib
```

---

## Project Structure

```
ConfRAG_ASR_CTCB.ipynb   ← Main notebook (CTCB evaluation)
README.md                 ← This file
```

---

## Authors

| Name | Affiliation |
|------|-------------|
| Ahmad ElSamak | Islamic University of Gaza |
| Lamiya AlSaedi | Islamic University of Gaza |
| Ramzi Abed | Islamic University of Gaza |
| Wisam Saqalla | Islamic University of Gaza |

**Supervisor:** Prof. Asma Sbaih — University of Seville

---

---

## Data Sources

- **MIMIC-III Demo**: [physionet.org/content/mimiciii-demo/1.4](https://physionet.org/content/mimiciii-demo/1.4/) — free, no registration required
- **RxNorm API**: [rxnav.nlm.nih.gov](https://rxnav.nlm.nih.gov/) — free, no registration required

---

