# 🛡️ PSGuard: Comparative Static Feature Analysis for Multi-Class PowerShell Malware Detection

[![Python](https://img.shields.io/badge/Python-3.10%20%7C%203.11%20%7C%203.12-blue?logo=python&logoColor=white)](https://www.python.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Dataset](https://img.shields.io/badge/Dataset-MPSD%20(das--lab)-orange.svg)](https://github.com/das-lab/mpsd)
[![Models](https://img.shields.io/badge/Models-LightGBM%20%7C%20RF%20%7C%20CodeBERT-purple.svg)]()
[![Explainability](https://img.shields.io/badge/XAI-SHAP%20TreeExplainer-red.svg)]()

> **Official repository for:**  
> *PSGuard: Comparative Static Feature Analysis for Multi-Class PowerShell Malware Detection*  
> **Authors:** Adiba Muttaqin, Belal Hossain  
> **Affiliation:** Department of Computer Science and Engineering, East West University, Dhaka, Bangladesh

---

## 📌 Table of Contents
- [Overview](#-overview)
- [Key Contributions](#-key-contributions)
- [Architecture & Workflow](#-architecture--workflow)
- [Dataset Summary](#-dataset-summary)
- [Feature Engineering & Ablation Study](#-feature-engineering--ablation-study)
- [Experimental Results](#-experimental-results)
- [Explainable AI (XAI) via SHAP](#-explainable-ai-xai-via-shap)
- [Repository Structure](#-repository-structure)
- [Getting Started](#-getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
  - [Running the Notebook](#running-the-notebook)
- [Citation](#-citation)
- [License](#-license)

---

## 📖 Overview

PowerShell is widely exploited in **Living-off-the-Land (LotL)** cyberattacks, enabling fileless malware execution directly in memory while bypassing disk-based defenses. Detecting such threats is increasingly complex due to **mixed malicious scripts**, where attackers conceal malicious commands within legitimate administrative scripts.

**PSGuard** provides an end-to-end, reproducible framework for **three-class PowerShell malware detection** (*Benign*, *Pure Malicious*, and *Mixed Malicious*). It performs a systematic ablation of static feature representations across classical machine learning models and benchmarks them against a fine-tuned **CodeBERT** transformer baseline.

```
+-----------------------------------------------------------------------------------+
|                                   PSGuard Pipeline                                |
|                                                                                   |
|  [Raw Scripts] -> [Preprocessing] -> [3-Branch Feature Extraction]                |
|                                       ├── Lexical (6D)                            |
|                                       ├── TF-IDF (5000D)                          |
|                                       └── Word2Vec (100D)                         |
|                                                                                   |
|  -> [7 Ablation Configurations (C1-C7)] -> [Classifiers: LR, RF, LightGBM]        |
|  -> [Transformer Baseline: CodeBERT]   -> [5-Fold Cross-Validation]               |
|  -> [Statistical Significance: McNemar] -> [SHAP Explainability (TreeExplainer)]   |
+-----------------------------------------------------------------------------------+
```

---

## 🚀 Key Contributions

- **Multi-Class Evaluation on Mixed Scripts:** Evaluates detection performance across three distinct classes: **Benign**, **Pure Malicious**, and **Mixed Malicious**, extending prior binary classification studies.
- **Factorial Feature Ablation (21 Experimental Runs):** Rigorously isolates the contributions of **Handcrafted Lexical**, **TF-IDF $n$-grams**, and **Word2Vec embeddings** across 7 subset combinations using Logistic Regression, Random Forest, and LightGBM.
- **Classical ML vs. Pretrained Transformer:** Compares lightweight feature-engineered classifiers against a fine-tuned **CodeBERT** baseline, demonstrating the practical and computational trade-offs for SOC environments.
- **Statistical Significance & Stability:** Employs **McNemar's test** ($p < 0.001$) and **5-fold stratified cross-validation** refitting feature extractors inside each fold to ensure zero data leakage.
- **Model Explainability (XAI):** Employs **TreeSHAP** to interpret global and class-specific feature attributions, explaining what patterns differentiate mixed malicious scripts.

---

## 🏗️ Architecture & Workflow

![PSGuard Methodology](_2023_2_60_10__2023_2_60_028__CSE487_Final_Project/Figures/Cyber_Methodology.png)

1. **Preprocessing:** Lowercasing, empty string sanitization, URL (`<url>`) and IP address (`<ip>`) token normalization, and non-ASCII character filtering without altering core PowerShell semantics.
2. **Feature Extraction:**
   - **Branch A (Lexical):** 6 statistical & structural features (Shannon entropy, character count, line count, token count, special character ratio, security keyword frequency).
   - **Branch B (TF-IDF):** Sublinear unigrams and bigrams restricted to the top 5,000 discriminative features.
   - **Branch C (Word2Vec):** 100-dimensional Skip-Gram embeddings aggregated via mean pooling.
3. **Classification:** 7 feature subsets ($C1$ to $C7$) evaluated across Logistic Regression, Random Forest, and LightGBM, alongside fine-tuned CodeBERT sequence classification.
4. **Interpretation:** Global and per-class feature attribution via SHAP TreeExplainer.

---

## 📊 Dataset Summary

Experiments are conducted on the benchmark **MPSD (Malicious PowerShell Script Detection)** dataset:

| Class | Samples | Description |
| :--- | :---: | :--- |
| `powershell_benign_dataset` | 4,316 | Legitimate PowerShell administrative scripts |
| `malicious_pure` | 4,202 | Purely malicious offensive PowerShell scripts |
| `mixed_malicious` | 4,202 | Malicious payloads embedded within benign scripts |
| **Total** | **12,720** | Balanced 3-class distribution (Stratified 80:20 Split) |

---

## 🔬 Feature Engineering & Ablation Study

![Feature Combinations](_2023_2_60_10__2023_2_60_028__CSE487_Final_Project/Figures/Feature_Combinations.png)

Seven feature configurations ($C1$–$C7$) were formulated through horizontal matrix concatenation:

| Config | Lexical ($6\text{D}$) | TF-IDF ($5000\text{D}$) | Word2Vec ($100\text{D}$) | Total Dimensions |
| :---: | :---: | :---: | :---: | :---: |
| **C1** | ✅ | ❌ | ❌ | 6 |
| **C2** | ❌ | ✅ | ❌ | 5,000 |
| **C3** | ❌ | ❌ | ✅ | 100 |
| **C4** | ✅ | ✅ | ❌ | 5,006 |
| **C5** | ✅ | ❌ | ✅ | 106 |
| **C6** | ❌ | ✅ | ✅ | 5,100 |
| **C7** | ✅ | ✅ | ✅ | 5,106 |

---

## 📈 Experimental Results

### 1. Primary Test-Set Performance (21 Classical Runs)

| Config | Model | Accuracy | Macro Precision | Macro Recall | Macro F1 | OvR ROC-AUC |
| :---: | :--- | :---: | :---: | :---: | :---: | :---: |
| **C1** | Logistic Regression | 0.7288 | 0.7224 | 0.7287 | 0.7240 | 0.8594 |
| | Random Forest | 0.9127 | 0.9156 | 0.9133 | 0.9132 | 0.9778 |
| | LightGBM | 0.9116 | 0.9143 | 0.9121 | 0.9122 | 0.9797 |
| **C2** | Logistic Regression | 0.9406 | 0.9419 | 0.9403 | 0.9399 | 0.9913 |
| | Random Forest | 0.9619 | 0.9619 | 0.9617 | 0.9617 | 0.9962 |
| | LightGBM | 0.9760 | 0.9761 | 0.9759 | 0.9759 | 0.9985 |
| **C3** | Logistic Regression | 0.8400 | 0.8437 | 0.8399 | 0.8395 | 0.9510 |
| | Random Forest | 0.7311 | 0.7348 | 0.7314 | 0.7306 | 0.8757 |
| | LightGBM | 0.7697 | 0.7722 | 0.7700 | 0.7701 | 0.9173 |
| **C4** | Logistic Regression | 0.9430 | 0.9433 | 0.9428 | 0.9426 | 0.9898 |
| | Random Forest | 0.9650 | 0.9651 | 0.9649 | 0.9650 | 0.9967 |
| | **LightGBM (Best)** | **0.9780** | **0.9781** | **0.9779** | **0.9780** | **0.9982** |
| **C5** | Logistic Regression | 0.8620 | 0.8648 | 0.8619 | 0.8619 | 0.9577 |
| | Random Forest | 0.9061 | 0.9070 | 0.9064 | 0.9065 | 0.9734 |
| | LightGBM | 0.9202 | 0.9214 | 0.9206 | 0.9206 | 0.9848 |
| **C6** | Logistic Regression | 0.9552 | 0.9561 | 0.9550 | 0.9552 | 0.9941 |
| | Random Forest | 0.9705 | 0.9707 | 0.9704 | 0.9705 | 0.9973 |
| | LightGBM | 0.9760 | 0.9762 | 0.9759 | 0.9760 | 0.9985 |
| **C7** | Logistic Regression | 0.9564 | 0.9571 | 0.9562 | 0.9564 | 0.9938 |
| | Random Forest | 0.9701 | 0.9703 | 0.9700 | 0.9701 | 0.9968 |
| | LightGBM | 0.9760 | 0.9762 | 0.9759 | 0.9760 | 0.9982 |

---

### 2. Best Classical ML vs. CodeBERT Baseline

| Model / Pipeline | Accuracy | Precision | Recall | Macro F1 | OvR AUC | Training Time |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **LightGBM (Lexical + TF-IDF, C4)** | **0.9780** | **0.9781** | **0.9779** | **0.9780** | **0.9982** | **45.06 s (CPU)** |
| **CodeBERT (`microsoft/codebert-base`)** | 0.7433 | 0.8133 | 0.7471 | 0.7187 | 0.9022 | 3,499.50 s (GPU) |

> 📌 **McNemar's Significance Test:** $\chi^2 = 547.33, p < 0.001 \implies$ Reject $H_0$. The superiority of the feature-engineered LightGBM model over CodeBERT is statistically significant.

---

### 3. Confusion Matrix (LightGBM on C4)

![Confusion Matrix](_2023_2_60_10__2023_2_60_028__CSE487_Final_Project/Figures/confusion_matrix.png)

---

## 🔍 Explainable AI (XAI) via SHAP

### Global Feature Importance (Top 20 Features)
![SHAP Top 20](_2023_2_60_10__2023_2_60_028__CSE487_Final_Project/Figures/shap_top20.png)

### Class-Specific SHAP Beeswarm Distributions
| Benign Class | Pure Malicious Class | Mixed Malicious Class |
| :---: | :---: | :---: |
| ![Benign](_2023_2_60_10__2023_2_60_028__CSE487_Final_Project/Figures/shap_benign.png) | ![Pure Malicious](_2023_2_60_10__2023_2_60_028__CSE487_Final_Project/Figures/shap_pure_malicious.png) | ![Mixed Malicious](_2023_2_60_10__2023_2_60_028__CSE487_Final_Project/Figures/shap_mixed_malicious.png) |

---

## 📁 Repository Structure

```text
├── PSGuard.ipynb                   # Complete interactive Jupyter Notebook pipeline
├── Paper.pdf                       # Full research paper PDF
├── _2023_2_60_10__.../             # LaTeX Source & Publication Assets
│   ├── main.tex                    # LaTeX source code
│   ├── references.bib              # Complete BibTeX citations
│   └── Figures/                    # High-resolution methodology & result plots
│       ├── Cyber_Methodology.png
│       ├── Feature_Combinations.png
│       ├── FEATURE_COMPARISON.png
│       ├── confusion_matrix.png
│       ├── shap_top20.png
│       ├── shap_benign.png
│       ├── shap_pure_malicious.png
│       └── shap_mixed_malicious.png
└── README.md                       # Repository documentation
```

---

## ⚡ Getting Started

### Prerequisites
- Python 3.10+
- (Optional for CodeBERT) NVIDIA GPU with CUDA support

### Installation

Clone the repository and install required packages:

```bash
git clone https://github.com/your-username/PSGuard.git
cd PSGuard

pip install scikit-learn lightgbm gensim shap torch transformers matplotlib seaborn pandas numpy
```

### Running the Notebook

Launch Jupyter and open `PSGuard.ipynb`:

```bash
jupyter notebook PSGuard.ipynb
```

The notebook will sequentially execute:
1. **MPSD Dataset loading & exploratory analysis**
2. **Preprocessing & 80:20 stratified splitting**
3. **Lexical, TF-IDF, and Word2Vec feature extraction**
4. **21 Classical ML runs across C1–C7**
5. **CodeBERT fine-tuning and evaluation**
6. **5-Fold Cross-Validation & McNemar's Test**
7. **TreeSHAP Explainability & Visualizations**

---

## 📝 Citation

If you find this repository or research helpful in your work, please cite our paper:

```bibtex
@article{muttaqin2026psguard,
  title     = {PSGuard: Comparative Static Feature Analysis for Multi-Class PowerShell Malware Detection},
  author    = {Muttaqin, Adiba and Hossain, Belal},
  journal   = {Department of Computer Science and Engineering, East West University},
  year      = {2026}
}
```

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
