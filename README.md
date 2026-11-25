# 🔐 Phishing URL Detection – Comparative Experimental Study

A comparative study of Machine Learning, Deep Learning, Ensemble Learning, and Logistic Stacking techniques for detecting phishing URLs under extremely low False Positive Rate (FPR) constraints.

**Author:** Arunima Upreti  
**Institution:** The NorthCap University, Gurugram, India  
**Project:** Phishing URL Detection Under Extremely Low False-Positive Rates  
**Study:** November 2026

---

## 📌 1. Project Overview

Phishing attacks use fraudulent URLs to trick users into revealing sensitive information such as passwords, financial details, and personal credentials.

This project focuses on developing and evaluating machine learning and deep learning models to classify URLs as:

- **Phishing (Malicious)**
- **Legitimate (Safe)**

The primary objective is to achieve high phishing detection while maintaining an extremely low False Positive Rate (FPR).

The experiments compare multiple model architectures across two different datasets, focusing particularly on the True Positive Rate (TPR) at 0.1% FPR.

## 🎯 2. Objectives

- Perform dataset auditing, duplicate removal, and label verification.
- Apply hostname-grouped train, validation, and test splitting.
- Implement classical machine learning algorithms.
- Develop character-level CNN and Transformer architectures.
- Experiment with different loss functions, including BCE and BCE+pAUC.
- Build score-averaging ensembles and logistic regression stacking.
- Evaluate models using AUC, TPR, and FPR.
- Compare raw URLs with scheme/www-stripped URLs.
- Analyse false positives and missed phishing URLs.

## 📂 3. Datasets

Two datasets were used to evaluate model performance under different data distributions.

### Dataset 1: Balanced URL Dataset

| Description | Count |
|---|---:|
| Original URLs | 340,000 |
| Phishing URLs | 170,000 |
| Legitimate URLs | 170,000 |
| Duplicate URLs removed | 277 |
| Retained URLs | 339,723 |
| Hosts appearing in both classes | 131 |

### Dataset 2: 101K URL Dataset

**Source:** `url_features_extracted1.csv`

| Description | Count |
|---|---:|
| Original rows reported | 101,219 |
| Retained rows reported | 101,218 |
| Phishing URLs (Label 0) | 63,678 |
| Legitimate URLs (Label 1) | 37,540 |
| Duplicate URLs reported removed | 346 |
| Hosts appearing in both classes | 1 |

**Data audit note:** The second dataset contains an inconsistency in its saved audit output. The reported original row count, duplicate count, and retained row count do not reconcile arithmetically. This discrepancy is retained as a limitation.

### Dataset Comparison

| Characteristic | Dataset 1 | Dataset 2 |
|---|---:|---:|
| Original rows | 340,000 | 101,219 |
| Retained rows | 339,723 | 101,218 |
| Phishing URLs | 170,000 | 63,678 |
| Legitimate URLs | 170,000 | 37,540 |
| Class distribution | 50% / 50% | 62.9% / 37.1% |
| Shared hosts across classes | 131 | 1 |

## ⚙️ 4. Experimental Methodology

### 4.1 Hostname-Grouped Data Splitting

URLs were grouped by hostname before splitting the data into training, validation, and testing sets.

This approach reduces the possibility of URLs from the same host appearing in multiple partitions.

| Split | Dataset 1 | Dataset 2 |
|---|---:|---:|
| Training | 237,232 | 71,562 |
| Validation | 34,189 | 9,603 |
| Testing | 68,302 | 19,707 |
| Legitimate test URLs | 34,310 | 7,449 |

### 4.2 URL Preprocessing

Two URL representations were evaluated:

- **Raw URLs:** Original URL strings.
- **Stripped URLs:** URL schemes (`http://`, `https://`) and leading `www` prefixes removed.

This comparison helps identify whether models depend heavily on URL formatting patterns.

### 4.3 Classical Machine Learning

The following models were implemented:

**1. Character n-gram Logistic Regression**
- Learns character-level patterns from URL strings.
- Captures suspicious character combinations.

**2. Lexical XGBoost**
- Uses engineered URL features.
- Includes URL length, hostname length, path length, query length, digit counts, punctuation counts, entropy, suspicious keywords, IP-address indicators, and URL-shortener indicators.

**3. Word n-gram Logistic Regression**
- Captures token-level patterns in URLs.
- Identifies suspicious words and URL structures.

### 4.4 Deep Learning Models

**Character-level CNN**
- Uses character embeddings.
- Applies parallel convolution branches with kernel sizes 3, 5, and 7.
- Uses pooling and dense layers for classification.

**Character-level Transformer**
- Trained from scratch.
- Uses learned positional embeddings.
- Contains two encoder layers and four attention heads.
- Learns contextual relationships between URL characters.

### 4.5 Loss Functions

Two loss configurations were investigated:

- Binary Cross-Entropy (BCE)
- Binary Cross-Entropy with an additional partial-AUC-inspired term (BCE+pAUC)

The first experiment trained CNNs on a 100,000-URL subset and the Transformer on a 60,000-URL subset for three epochs.

## 🧠 5. Ensemble Learning and Stacking

Multiple models were combined to improve detection performance.

| Ensemble | Description |
|---|---|
| ENS-A | Three raw-URL classical models and four CNN runs |
| ENS-B | ENS-A with the character Transformer |
| ENS-C | Gives the CNN group twice the weight of the classical group |
| ENS-STRIPPED | Combines stripped character LR, lexical XGBoost, and CNN |
| STACK | Logistic regression trained on validation predictions |

For averaging ensembles, model scores were standardised using validation-set means and standard deviations.

The logistic stacker learns the contribution of individual model predictions using validation data rather than test labels.

## 📊 6. Evaluation Metrics

| Metric | Description |
|---|---|
| AUC | Area Under the ROC Curve; measures overall ranking performance |
| TPR | True Positive Rate; percentage of phishing URLs correctly detected |
| FPR | False Positive Rate; percentage of legitimate URLs incorrectly flagged |
| TPR @ 1% FPR | Phishing detection rate at a 1% false-positive rate |
| TPR @ 0.1% FPR | Phishing detection rate at a 0.1% false-positive rate |

The primary focus is **TPR at 0.1% FPR**, as practical phishing detection systems must minimise false alarms while detecting malicious URLs.

---

# 📈 7. Results – Dataset 1

### 7.1 Main Test Results (Raw URLs)

| Model | AUC | TPR @ 1% FPR | TPR @ 0.1% FPR |
|---|---:|---:|---:|
| Character n-gram LR | 0.9967 | 94.18% | 75.94% |
| Lexical XGBoost | 0.9919 | 84.94% | 70.89% |
| Word n-gram LR | 0.9927 | 84.66% | 66.02% |
| CNN BCE Only (Seed 0) | 0.9969 | 93.92% | 83.67% |
| CNN BCE+pAUC (Seed 0) | 0.9964 | 93.42% | 80.30% |
| Character Transformer | 0.9885 | 81.26% | 64.54% |
| ENS-A | 0.9982 | 96.51% | 89.51% |
| ENS-B | 0.9981 | 96.28% | 88.02% |
| ENS-C | 0.9981 | 96.25% | 88.48% |
| ENS-STRIPPED | 0.9978 | 95.97% | 87.15% |
| **STACK** | **0.9984** | **97.24%** | **90.16%** |

**Best-performing model:** Logistic Regression Stacker (STACK)

- AUC: 0.9984
- TPR @ 1% FPR: 97.24%
- TPR @ 0.1% FPR: 90.16%

### 7.2 Stripped URL Results

| Model | AUC | TPR @ 1% FPR | TPR @ 0.1% FPR |
|---|---:|---:|---:|
| Character n-gram LR | 0.9947 | 91.90% | 68.52% |
| Lexical XGBoost | 0.9830 | 79.76% | 63.90% |
| CNN BCE+pAUC | 0.9943 | 91.72% | 79.36% |
| ENS-STRIPPED | 0.9978 | 95.97% | 87.15% |

The reduction in performance indicates that URL schemes and `www` prefixes contribute predictive information.

---

# 📈 8. Results – Dataset 2

### 8.1 Main Test Results (Raw URLs)

| Model | AUC | TPR @ 1% FPR | TPR @ 0.1% FPR |
|---|---:|---:|---:|
| Character n-gram LR | 1.0000 | 99.99% | 99.94% |
| Lexical XGBoost | 1.0000 | 100.00% | 99.91% |
| Word n-gram LR | 1.0000 | 100.00% | 99.90% |
| CNN BCE+pAUC (Seed 0) | 1.0000 | 99.99% | 99.43% |
| CNN BCE Only (Seed 0) | 1.0000 | 100.00% | 99.75% |
| CNN BCE Only (Seed 1) | 1.0000 | 100.00% | 99.60% |
| CNN BCE Only (Seed 2) | 1.0000 | 100.00% | 99.60% |
| Character Transformer | 1.0000 | 99.99% | 99.50% |
| ENS-A | 1.0000 | 100.00% | 99.94% |
| ENS-B | 1.0000 | 100.00% | 99.93% |
| ENS-C | 1.0000 | 100.00% | 99.93% |
| ENS-STRIPPED | 1.0000 | 100.00% | 99.84% |
| **STACK** | **1.0000** | **100.00%** | **99.96%** |

**Best reported model:** Logistic Regression Stacker (STACK)

- AUC: 1.0000 (rounded)
- TPR @ 1% FPR: 100.00%
- TPR @ 0.1% FPR: 99.96%

Note: AUC values are rounded to four decimal places. A displayed value of 1.0000 does not necessarily indicate mathematically perfect classification.

### 8.2 Stripped URL Results

| Model | AUC | TPR @ 1% FPR | TPR @ 0.1% FPR |
|---|---:|---:|---:|
| Character n-gram LR | 0.9997 | 99.92% | 91.70% |
| Lexical XGBoost | 1.0000 | 99.99% | 99.61% |
| CNN BCE+pAUC | 0.9999 | 99.98% | 99.04% |
| ENS-STRIPPED | 1.0000 | 100.00% | 99.84% |

### 8.3 Validation-Calibrated Threshold Evaluation

Thresholds were selected using validation predictions, frozen, and then evaluated on the test set.

| Model | Target FPR | Realised Test FPR | False Alarms | Test TPR |
|---|---:|---:|---:|---:|
| Character n-gram LR | 0.1% | 0.148% | 11 | 99.94% |
| CNN BCE Only (Seed 0) | 0.1% | 0.107% | 8 | 99.75% |
| ENS-A | 0.1% | 0.148% | 11 | 99.95% |
| STACK | 0.1% | 0.121% | 9 | 99.96% |
| ENS-STRIPPED | 0.1% | 0.067% | 5 | 99.80% |

The stripped ensemble achieves a realised test FPR of 0.067%, below the 0.1% target.

---

# 🔍 9. Direct Dataset Comparison

| Metric | Dataset 1 | Dataset 2 |
|---|---:|---:|
| Test URLs | 68,302 | 19,707 |
| Legitimate Test URLs | 34,310 | 7,449 |
| Best Reported Model | STACK | STACK |
| Best AUC | 0.9984 | 1.0000 |
| Best TPR @ 1% FPR | 97.24% | 100.00% |
| Best TPR @ 0.1% FPR | 90.16% | 99.96% |
| Stripped Ensemble TPR @ 0.1% FPR | 87.15% | 99.84% |

### Key Observations

1. The same broad model families performed strongly across both datasets.
2. Dataset 2 produced substantially higher reported scores.
3. The second dataset contains stronger HTTPS and `www` correlations with class labels.
4. Ensemble improvements over individual models are relatively small in Dataset 2.
5. The second dataset has fewer legitimate test URLs, making low-FPR estimates more sensitive to individual examples.
6. Validation-calibrated thresholds provide a more realistic assessment of the operating FPR.

The 9.80 percentage-point difference in the reported stacker TPR at 0.1% FPR is descriptive and should not be interpreted as proof of improved generalisation.

---

# 🧪 10. Error Analysis

### Dataset 1

At the stacker's reported 0.1% FPR operating point:

- 3,345 of 33,992 phishing URLs were missed.
- Missed examples included URLs from `tiny.cc`, `afna.fr`, `r2me.com`, and other hosts.
- Legitimate account, login, and security-related URLs sometimes received high phishing scores because their lexical patterns resembled suspicious URLs.

### Dataset 2

At the stacker's reported 0.1% FPR threshold:

- 5 of 12,258 phishing URLs were missed.
- Missed examples included URLs from `www.hercvlesconcrete.com`, `www.apijs.djibysoft.com`, and `www.eb2a.live`.
- Some legitimate URLs near the high-score end belonged to websites such as Kellogg's, NTA, Bootstrap, Netcraft, and Spotify.

---

# ⚠️ 11. Limitations

- **Dataset shortcut risk:** HTTPS and `www` are strongly correlated with labels, particularly in Dataset 2.
- **Small low-FPR denominator:** Dataset 2 has only 7 legitimate test examples corresponding to the 0.1% false-alarm allowance.
- **Threshold selection:** Primary test metrics use thresholds selected from test negatives. Frozen validation-calibrated thresholds can produce different results.
- **Host-disjoint limitation:** Hostname grouping does not guarantee generalisation to future phishing campaigns.
- **Source bias:** Different datasets may reflect different sources and collection periods.
- **Rounded AUC:** Values shown as 1.0000 are rounded.
- **Cached predictions:** Exact reproduction depends on the corresponding datasets, split files, and cached model scores.
- **Deployment gap:** The experiments are offline and do not establish production latency, live browser integration, or adversarial robustness.

---

# 🚀 12. Future Work

- Evaluate models on larger and independently collected URL datasets.
- Introduce time-based train/test splitting.
- Use validation-only threshold selection.
- Increase bootstrap resampling and report uncertainty intervals.
- Perform feature ablation studies.
- Evaluate external datasets from different collection sources.
- Investigate pretrained URL/language models.
- Test adversarial URLs involving redirects, shorteners, homoglyphs, and encoded paths.
- Evaluate model calibration, latency, and false alarms per million URLs.
- Explore real-time phishing URL detection systems.

---

# 🛠️ 13. Technologies and Libraries

The experiments involve the following techniques and tools:

| Category | Technologies |
|---|---|
| Programming Language | Python |
| Classical ML | Scikit-learn, XGBoost |
| Deep Learning | TensorFlow / Keras |
| Data Processing | Pandas, NumPy |
| Visualisation | Matplotlib, Seaborn |
| Models | Logistic Regression, CNN, Transformer |
| Evaluation | ROC-AUC, TPR, FPR |
| Ensemble Methods | Score Averaging, Logistic Stacking |

---

# 📌 14. Conclusion

This comparative experimental study demonstrates that classical machine learning, character-level deep learning, and ensemble methods can achieve strong phishing URL detection performance under strict false-positive constraints.

For Dataset 1, the logistic stacking model achieves an AUC of 0.9984 and a TPR of 90.16% at 0.1% FPR.

For Dataset 2, the same stacking approach reports an AUC of 1.0000 (rounded) and a TPR of 99.96% at the test-selected 0.1% FPR operating point.

However, the second dataset's validation-calibrated threshold produces a realised test FPR of 0.121%. Its strong formatting correlations and smaller test set also require careful interpretation.

The study highlights the importance of evaluating phishing detection systems not only through overall accuracy but also through low-FPR performance, validation-based threshold selection, and robust generalisation.

---

**Author:** Arunima Upreti  
**The NorthCap University | Computer Science and Engineering (Data Science)**  
**November 2026**
