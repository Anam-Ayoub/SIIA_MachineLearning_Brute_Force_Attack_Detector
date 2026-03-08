<div align="center">

# 🛡️ Network Intrusion Detection System

### Machine Learning-Based Encrypted Traffic Classification

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![scikit-learn](https://img.shields.io/badge/scikit--learn-1.x-F7931E?style=for-the-badge&logo=scikit-learn&logoColor=white)](https://scikit-learn.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?style=for-the-badge&logo=jupyter&logoColor=white)](https://jupyter.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)](LICENSE)

> A high-performance Random Forest classifier that detects **brute-force attacks** in encrypted network traffic flows with **99.97% accuracy**.

---

</div>

## 📋 Table of Contents

- [Overview](#-overview)
- [Project Structure](#-project-structure)
- [Dataset](#-dataset)
- [ML Pipeline](#-ml-pipeline)
- [Results](#-results)
- [Installation & Usage](#-installation--usage)
- [Key Features](#-key-features)
- [Technologies](#-technologies)
- [License](#-license)

---

## 🔍 Overview

This project implements an end-to-end machine learning pipeline for **network intrusion detection**, specifically targeting **brute-force attacks** within encrypted (TLS) network traffic. By analyzing statistical properties of network flows — such as packet sizes, timing patterns, and TLS fingerprints — the model can distinguish between normal and malicious traffic **without decrypting the payload**.

The system achieves a **99.97% accuracy** rate, correctly classifying 13,490 out of 13,494 test samples with only **4 false negatives** and **zero false positives**.

---

## 📁 Project Structure

```
ML_Project/
├── model_notebook.ipynb      # Complete ML pipeline (preprocessing → training → evaluation → export)
├── samples.csv               # Dataset — 67,469 network flow records with 60+ features
├── detection_system.pkl      # Exported trained model (includes model, scaler, selector)
└── README.md                 # This file
```

---

## 📊 Dataset

The dataset (`samples.csv`) contains **67,469 network flow records** captured from backbone network traffic, with **60 features** per sample.

### Feature Categories

| Category | Examples | Description |
|---|---|---|
| **Network Identifiers** | `SRC_IP`, `DST_IP`, `SRC_PORT`, `DST_PORT` | Hashed source/destination IPs and ports |
| **Protocol Info** | `PROTOCOL`, `TLS_SNI`, `TLS_JA3` | Transport protocol and TLS fingerprinting |
| **Timing** | `TIME_FIRST`, `TIME_LAST`, `DURATION` | Flow start/end timestamps and duration |
| **Volume** | `BYTES`, `BYTES_REV`, `PACKETS`, `PACKETS_REV` | Byte and packet counts (forward/reverse) |
| **Statistical** | `request_sizes_std`, `request_sizes_median`, `response_sizes_q80` | Statistical distributions of packet sizes |
| **Derived Metrics** | `download_ratio`, `fft_peak_ratio`, `autocorr_2` | Computed flow behavior indicators |
| **Labels** | `SCENARIO`, `CLASS` | Capture scenario and binary classification target |

### Class Distribution

| Class | Label | Proportion |
|---|---|---|
| `0` | Normal Traffic | **82.7%** |
| `1` | Brute-Force Attack | **17.3%** |

---

## 🔬 ML Pipeline

The pipeline is implemented in `model_notebook.ipynb` and consists of **4 phases**:

### Phase 1 — Data Preprocessing

- **Load** the CSV dataset (67,469 × 61)
- **Drop** identifier/timestamp columns (`SRC_IP`, `DST_IP`, `TIME_FIRST`, `TIME_LAST`, `SCENARIO`, `TLS_SNI`)
- **Handle missing values** by filling NaN with `0` (e.g., absent TLS data means no encryption)
- **Label encode** categorical columns (`SRC_PORT`, `TLS_JA3`) using `LabelEncoder`
- **Split** into train/test sets (80/20) with stratification to preserve class balance

### Phase 2 — Optimization & Training

1. **Normalization** — `StandardScaler` fitted on training data only (prevents data leakage)
2. **Feature Selection** — `SelectFromModel` with a preliminary Random Forest reduces features from **54 → 19**
3. **Hyperparameter Tuning** — `GridSearchCV` (3-fold CV) over:
   - `n_estimators`: [50, 100]
   - `max_depth`: [10, 20, None]
   - `min_samples_split`: [2, 5]
4. **Best Parameters Found**: `{'max_depth': None, 'min_samples_split': 5, 'n_estimators': 100}`

### Phase 3 — Visualization

- **Confusion Matrix** heatmap (Seaborn)
- **Feature Importance** bar chart (Top 10 features)

### Phase 4 — Model Export

The trained system is serialized via `joblib` into `detection_system.pkl`, which bundles:
- The trained `RandomForestClassifier`
- The fitted `StandardScaler`
- The fitted `SelectFromModel` selector
- Feature name mappings
- Test data for reproducibility

---

## 📈 Results

### Performance Metrics

| Metric | Normal (0) | Brute Force (1) | Overall |
|---|---|---|---|
| **Precision** | 1.00 | 1.00 | — |
| **Recall** | 1.00 | 1.00 | — |
| **F1-Score** | 1.00 | 1.00 | — |
| **Accuracy** | — | — | **99.97%** |

### Confusion Matrix

|  | Predicted Normal | Predicted Attack |
|---|---|---|
| **Actual Normal** | 11,165 | 0 |
| **Actual Attack** | 4 | 2,325 |

> Only **4 out of 13,494** test samples were misclassified — all were false negatives (attacks missed).

### Top 10 Most Important Features

| Rank | Feature | Importance |
|---|---|---|
| 1 | `TLS_JA3` | 0.1617 |
| 2 | `request_sizes_max` | 0.1280 |
| 3 | `request_sizes_std` | 0.0725 |
| 4 | `request_pkts_mean` | 0.0582 |
| 5 | `fft_peak_ratio` | 0.0557 |
| 6 | `download_ratio_3` | 0.0521 |
| 7 | `roundtrips_per_sec` | 0.0488 |
| 8 | `unique_response_sizes_ratio` | 0.0475 |
| 9 | `unique_merged_sizes_ratio` | 0.0466 |
| 10 | `response_sizes_min` | 0.0456 |

---

## 🚀 Installation & Usage

### Prerequisites

- Python 3.10+
- pip

### Setup

```bash
# Clone the repository
git clone <repository-url>
cd ML_Project

# Install dependencies
pip install pandas numpy scikit-learn matplotlib seaborn joblib
```

### Train the Model

Open and run all cells in the Jupyter notebook:

```bash
jupyter notebook model_notebook.ipynb
```

### Load the Pre-Trained Model

```python
import joblib

# Load the complete detection system
system = joblib.load('detection_system.pkl')

model    = system['model']       # Trained RandomForestClassifier
scaler   = system['scaler']      # Fitted StandardScaler
selector = system['selector']    # Fitted feature selector
features = system['features_names']

# Predict on new data
# new_data should be a DataFrame with the same feature columns
scaled  = scaler.transform(new_data)
reduced = selector.transform(scaled)
predictions = model.predict(reduced)
# 0 = Normal, 1 = Brute-Force Attack
```

---

## ✨ Key Features

- 🎯 **99.97% Accuracy** — Near-perfect detection with minimal false negatives
- 🔒 **Encrypted Traffic Analysis** — Works on TLS-encrypted flows without payload decryption
- ⚡ **Efficient Feature Selection** — Reduces feature space from 54 to 19 for faster inference
- 🔧 **Automated Hyperparameter Tuning** — GridSearchCV finds optimal model configuration
- 📦 **Export-Ready** — Complete model pipeline serialized for deployment

---

## 🛠️ Technologies

| Tool | Purpose |
|---|---|
| **pandas** | Data loading and manipulation |
| **NumPy** | Numerical operations |
| **scikit-learn** | ML pipeline (preprocessing, training, evaluation) |
| **Matplotlib** | Base plotting library |
| **Seaborn** | Statistical data visualization |
| **joblib** | Model serialization |

---

## 📄 License

This project is available under the [MIT License](LICENSE).

---

<div align="center">

**Built with ❤️ for network security**

</div>
]]>

