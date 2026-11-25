# Phishing URL Detection: Comparative Experimental Study

A comparative machine learning and deep learning study for detecting
phishing URLs under extremely low false-positive-rate (FPR) constraints.

**Institution:** The NorthCap University, Gurugram, Haryana, India\
**Author:** Arunima Upreti\
**Study:** November 2026

------------------------------------------------------------------------

## 1. Project Overview

Phishing URLs direct users to fraudulent websites designed to steal
credentials, financial information, and other sensitive data. This
project develops and evaluates URL-based classification models to
distinguish phishing URLs from legitimate URLs.

The main objective is not only to achieve high classification
performance, but also to maximise phishing detection while keeping false
positives extremely low. Particular attention is given to the **True
Positive Rate (TPR) at 0.1% FPR**.

The study contains two experiments using different datasets. Both follow
a broad experimental workflow involving classical machine learning,
character-level deep learning, averaging ensembles, and
validation-fitted logistic stacking.

## 2. Objectives

-   Audit datasets, labels, duplicate URLs, and repeated hosts.
-   Use hostname-grouped train, validation, and test splits to reduce
    host leakage.
-   Train classical machine learning models using character n-grams,
    word n-grams, and engineered lexical features.
-   Evaluate character-level CNN and Transformer architectures.
-   Combine model predictions using score-averaging ensembles and
    logistic stacking.
-   Measure AUC and TPR at 10%, 5%, 1%, and 0.1% FPR.
-   Compare raw URLs with scheme- and `www`-stripped URL
    representations.
-   Analyse false positives and missed phishing URLs.

## 3. Datasets

### Dataset 1: Balanced URL Dataset

-   Original URLs: 340,000
-   Phishing URLs: 170,000
-   Legitimate URLs: 170,000
-   Exact duplicate URLs removed: 277
-   Retained URLs: 339,723
-   Hosts appearing in both classes: 131

### Dataset 2: 101K URL Dataset

Source file: `url_features_extracted1.csv`

-   Original rows reported: 101,219
-   Rows after deduplication reported: 101,218
-   Phishing URLs (label 0): 63,678
-   Legitimate URLs (label 1): 37,540
-   Duplicate URLs reported removed: 346
-   Hosts appearing in both classes: 1

**Data audit caveat:** The second dataset's saved audit output contains
an inconsistency: 101,219 original rows minus 346 reported duplicates
does not equal the reported 101,218 retained rows. The displayed class
counts sum to 101,218, while the split sizes sum to 100,872. These
discrepancies are retained as limitations rather than silently
corrected.

### Dataset Comparison

  Characteristic                  Dataset 1       Dataset 2
  ----------------------------- ----------- ---------------
  Original rows                     340,000         101,219
  Retained rows (reported)          339,723         101,218
  Phishing URLs                     170,000          63,678
  Legitimate URLs                   170,000          37,540
  Class distribution              50% / 50%   62.9% / 37.1%
  Hosts shared across classes           131               1

The datasets have different class distributions, URL characteristics,
and formatting patterns. Their absolute model scores should therefore
not be treated as a controlled, like-for-like comparison.

## 4. Experimental Methodology

### 4.1 Hostname-Grouped Data Splitting

URLs are grouped by parsed hostname before creating training,
validation, and test partitions. This reduces the likelihood of URLs
from the same host appearing across multiple partitions.

  Split                    Dataset 1   Dataset 2
  ---------------------- ----------- -----------
  Training                   237,232      71,562
  Validation                  34,189       9,603
  Test                        68,302      19,707
  Legitimate test URLs        34,310       7,449

At a 0.1% FPR operating point, the test sets permit approximately 34
false alarms in Dataset 1 and only 7 in Dataset 2. Consequently, the
strictest FPR estimate is sensitive to a small number of examples.

### 4.2 URL Preprocessing

Two URL representations are evaluated:

-   **Raw URLs:** Original URL strings.
-   **Stripped URLs:** URL schemes (such as `http://` and `https://`)
    and leading `www` or numbered-`www` prefixes are removed.

This comparison helps investigate whether models depend heavily on
formatting cues.

### 4.3 Classical Machine Learning Features

-   **Character n-gram Logistic Regression:** Learns character-level
    patterns from URL strings.
-   **Lexical XGBoost:** Uses engineered URL properties, including URL,
    host, path, and query lengths; digit and punctuation counts;
    hostname structure; IP-address and URL-shortener indicators;
    entropy; suspicious keyword counts; and file-extension/path
    features.
-   **Word n-gram Logistic Regression:** Captures token-level patterns
    in URL text.

### 4.4 Deep Learning Models

**Character-level CNN** - Uses byte/character embeddings. - Applies
parallel convolution branches with kernel widths 3, 5, and 7. - Uses
pooling and dense layers for classification.

**Character-level Transformer** - Trained from scratch, not
pretrained. - Uses learned positional embeddings. - Contains two encoder
layers and four attention heads.

**Loss functions** - Binary Cross-Entropy (BCE). - BCE with an
additional partial-AUC-inspired term (BCE+pAUC).

The first experiment trains CNNs on a 100,000-URL subset and the
Transformer on a 60,000-URL subset for three epochs. The second
experiment reports executed raw Transformer and stripped CNN runs
alongside cached model scores.

## 5. Models and Ensemble Strategy

  Model family          Models
  --------------------- ------------------------------------------------------
  Classical ML          Character n-gram LR, lexical XGBoost, word n-gram LR
  Deep learning         Character CNN, character Transformer
  Averaging ensembles   ENS-A, ENS-B, ENS-C, ENS-STRIPPED
  Stacking              Logistic regression meta-learner

### Ensemble configurations

-   **ENS-A:** Three raw-URL classical models and four raw-URL CNN runs.
-   **ENS-B:** ENS-A with the character Transformer.
-   **ENS-C:** Gives the CNN group twice the weight of the classical
    group.
-   **ENS-STRIPPED:** Combines stripped character LR, stripped lexical
    XGBoost, and stripped CNN scores.
-   **STACK:** Logistic regression trained on validation scores from the
    three classical models and four CNN runs.

For averaging ensembles, model scores are standardised using
validation-set means and standard deviations before combination. The
logistic stacker is fitted on validation predictions rather than test
labels.

## 6. Evaluation Metrics

  -----------------------------------------------------------------------
  Metric                              Description
  ----------------------------------- -----------------------------------
  AUC                                 Area Under the ROC Curve; measures
                                      ranking quality across thresholds.

  TPR                                 True Positive Rate, also called
                                      recall; fraction of phishing URLs
                                      correctly detected.

  FPR                                 False Positive Rate; fraction of
                                      legitimate URLs incorrectly flagged
                                      as phishing.

  TPR@FPR                             Phishing detection rate at a
                                      specified false-positive rate.
  -----------------------------------------------------------------------

The 0.1% FPR operating point is particularly important for practical URL
detection, where incorrectly flagging legitimate links can disrupt
users.

## 7. Results: Dataset 1

### Main test results

| Model | AUC | TPR @ 1% FPR | TPR @ 0.1% FPR |
|---|---:|---:|---:|
| Character n-gram LR (raw) | 0.9967 | 94.18% | 75.94% |
| Lexical XGBoost (raw) | 0.9919 | 84.94% | 70.89% |
| Word n-gram LR | 0.9927 | 84.66% | 66.02% |
| CNN BCE only, seed 0 | 0.9969 | 93.92% | 83.67% |
| CNN BCE+pAUC, seed 0 | 0.9964 | 93.42% | 80.30% |
| Character Transformer | 0.9885 | 81.26% | 64.54% |
| ENS-A | 0.9982 | 96.51% | 89.51% |
| ENS-B | 0.9981 | 96.28% | 88.02% |
| ENS-C | 0.9981 | 96.25% | 88.48% |
| ENS-STRIPPED | 0.9978 | 95.97% | 87.15% |
| **STACK** | **0.9984** | **97.24%** | **90.16%** |

The validation-trained logistic stacker is the strongest reported model
for Dataset 1, achieving 90.16% TPR at 0.1% FPR.

### Stripped URL results

| Model | AUC | TPR @ 1% FPR | TPR @ 0.1% FPR |
|---|---:|---:|---:|
| Character n-gram LR (stripped) | 0.9947 | 91.90% | 68.52% |
| Lexical XGBoost (stripped) | 0.9830 | 79.76% | 63.90% |
| CNN BCE+pAUC (stripped) | 0.9943 | 91.72% | 79.36% |
| ENS-STRIPPED | 0.9978 | 95.97% | 87.15% |

The reduction in performance after stripping URL schemes and `www`
indicates that these formatting cues contribute to model predictions.

## 8. Results: Dataset 2

### Main test results

| Model | AUC | TPR @ 1% FPR | TPR @ 0.1% FPR |
|---|---:|---:|---:|
| Character n-gram LR (raw) | 1.0000 | 99.99% | 99.94% |
| Lexical XGBoost (raw) | 1.0000 | 100.00% | 99.91% |
| Word n-gram LR | 1.0000 | 100.00% | 99.90% |
| CNN BCE+pAUC, seed 0 | 1.0000 | 99.99% | 99.43% |
| CNN BCE only, seed 0 | 1.0000 | 100.00% | 99.75% |
| CNN BCE only, seed 1 | 1.0000 | 100.00% | 99.60% |
| CNN BCE only, seed 2 | 1.0000 | 100.00% | 99.60% |
| Character Transformer | 1.0000 | 99.99% | 99.50% |
| ENS-A | 1.0000 | 100.00% | 99.94% |
| ENS-B | 1.0000 | 100.00% | 99.93% |
| ENS-C | 1.0000 | 100.00% | 99.93% |
| ENS-STRIPPED | 1.0000 | 100.00% | 99.84% |
| **STACK** | **1.0000** | **100.00%** | **99.96%** |

AUC values are rounded to four decimal places. A displayed value of
1.0000 should not be interpreted as proof of mathematically perfect
classification.

### Stripped URL results

| Model | AUC | TPR @ 1% FPR | TPR @ 0.1% FPR |
|---|---:|---:|---:|
| Character n-gram LR (stripped) | 0.9997 | 99.92% | 91.70% |
| Lexical XGBoost (stripped) | 1.0000 | 99.99% | 99.61% |
| CNN BCE+pAUC (stripped) | 0.9999 | 99.98% | 99.04% |
| ENS-STRIPPED | 1.0000 | 100.00% | 99.84% |

### Validation-calibrated threshold check

The notebook also evaluates thresholds selected on validation scores and
then frozen before testing. This provides a more deployment-oriented
view than selecting thresholds directly from test negatives.

| Model | Target FPR | Realised test FPR | False alarms | Test TPR |
|---|---:|---:|---:|---:|
| Character n-gram LR (raw) | 0.1% | 0.148% | 11 | 99.94% |
| CNN BCE only, seed 0 | 0.1% | 0.107% | 8 | 99.75% |
| ENS-A | 0.1% | 0.148% | 11 | 99.95% |
| STACK (cross-fitted validation) | 0.1% | 0.121% | 9 | 99.96% |
| ENS-STRIPPED | 0.1% | 0.067% | 5 | 99.80% |

The frozen-threshold stacker records a realised test FPR of 0.121%,
above the 0.1% target. ENS-STRIPPED records 0.067%, below the target,
with 99.80% TPR.

## 9. Direct Comparison

  Measure                              Dataset 1          Dataset 2
  ---------------------------------- ----------- ------------------
  Test URLs                               68,302             19,707
  Legitimate test URLs                    34,310              7,449
  Best reported model                      STACK              STACK
  Best reported AUC                       0.9984   1.0000 (rounded)
  Best reported TPR @ 1% FPR              97.24%            100.00%
  Best reported TPR @ 0.1% FPR            90.16%             99.96%
  Stripped ensemble TPR @ 0.1% FPR        87.15%             99.84%

The second dataset's reported stacker TPR at 0.1% FPR is 9.80 percentage
points higher than the first dataset's result. This is a descriptive
difference, not proof of improved model generalisation.

The second dataset has much stronger formatting correlations: HTTPS
appears in 100% of legitimate URLs but only 5.5% of phishing URLs. A
leading `www` appears in 99.2% of legitimate URLs and 1.0% of phishing
URLs. These patterns may act as dataset-specific shortcuts.

## 10. Error Analysis

### Dataset 1

At the stacker's reported 0.1% FPR operating point: - 3,345 of 33,992
phishing test URLs are missed (approximately 9.8%). - Missed examples
include URLs hosted on `www.atheneconstruction.com`, `tiny.cc`,
`afna.fr`, `r2me.com`, `missiontry.org`, and `www.tinyurl.com`. - Some
legitimate account, login, and security-related URLs receive high
phishing scores because their lexical patterns resemble suspicious URLs.

### Dataset 2

At the stacker's reported 0.1% FPR threshold: - 5 of 12,258 phishing
test URLs are missed. - Examples include URLs hosted on
`www.hercvlesconcrete.com`, `www.apijs.djibysoft.com`, and
`www.eb2a.live`. - Some legitimate URLs near the high-score end belong
to websites such as Kellogg's, NTA, Bootstrap, Netcraft, and Spotify.

These findings are specific to the evaluated datasets and should not be
taken as evidence of live deployment performance.

## 11. Limitations and Threats to Validity

-   **Dataset shortcut risk:** HTTPS and `www` are strongly correlated
    with labels, especially in Dataset 2.
-   **Small low-FPR denominator:** Dataset 2 has only 7 legitimate test
    examples corresponding to the 0.1% false-alarm allowance.
-   **Threshold selection:** Primary TPR@FPR tables select thresholds
    from test negatives; frozen validation-calibrated results can
    differ.
-   **Host-disjoint is not time-disjoint:** Host grouping does not
    establish performance on future phishing campaigns or newly
    registered domains.
-   **Source and temporal bias:** The datasets may represent different
    sources and collection periods.
-   **Rounded AUC:** Values printed as 1.0000 are rounded.
-   **Cached scores:** Reproduction depends on the corresponding
    datasets, split files, and cached model scores where used.
-   **Deployment gap:** The experiments are offline and do not establish
    live browser integration, production latency, adversarial
    robustness, or user-impact performance.
-   **Label mapping:** Dataset 2 uses label 0 for phishing and label 1
    for legitimate, based on the notebook's audit and sample URLs.

## 12. Conclusion

The experiments show that classical machine learning, character-level
deep learning, and score-combination methods can achieve strong phishing
URL ranking performance under low-FPR evaluation.

For Dataset 1, the logistic stacker achieves an AUC of 0.9984 and 90.16%
TPR at 0.1% FPR. For Dataset 2, it reports an AUC of 1.0000 (rounded)
and 99.96% TPR at the test-selected operating point. However, its
validation-calibrated threshold produces a realised test FPR of 0.121%.

The higher scores on Dataset 2 should not be interpreted as a direct
improvement in generalisation. Strong HTTPS and `www` label
correlations, a smaller test set, and different class distributions make
it a substantially different evaluation setting.

## 13. Future Work

-   Evaluate on a larger, independently collected legitimate URL
    dataset.
-   Use time-based splits and fresh URLs to test temporal
    generalisation.
-   Select thresholds using validation data only and report realised
    test FPR.
-   Increase bootstrap resamples and report uncertainty intervals.
-   Perform feature ablations for scheme, `www`, URL length, host
    identity, and engineered features.
-   Test on external datasets with different collection sources and
    class proportions.
-   Evaluate pretrained URL/language models and adversarial cases such
    as shorteners, redirects, homoglyphs, encoded paths, and compromised
    legitimate hosts.
-   Assess calibration, latency, and false alarms per million URLs
    before deployment.

## 14. Reproducibility Notes

  -------------------------------------------------------------------------------
  Setting                 First experiment        Second experiment
  ----------------------- ----------------------- -------------------------------
  Dataset                 Balanced raw-URL        `url_features_extracted1.csv`
                          dataset                 

  Split                   Host-grouped,           Host-grouped, approximately
                          approximately 70/10/20  70/10/20

  Classical models        Character LR, lexical   Character LR, lexical XGBoost,
                          XGBoost, word LR        word LR

  Deep models             Character CNN,          Character CNN, from-scratch
                          from-scratch            Transformer
                          Transformer             

  Ensembles               ENS-A, ENS-B, ENS-C,    ENS-A, ENS-B, ENS-C,
                          ENS-STRIPPED, STACK     ENS-STRIPPED, STACK

  Cache mode              Cached scores used      `RETRAIN=False`; cached scores
                          where available         used where available

  Additional analysis     Low-FPR ROC, error      Host-cluster bootstrap, frozen
                          analysis                validation thresholds, error
                                                  analysis
  -------------------------------------------------------------------------------

**Scope note:** This README reflects the saved outputs and explanatory
content in the supplied project document. It does not claim that the
experiments were rerun during README preparation. The second notebook's
DistilBERT section is described as proposed/optional; no executed
DistilBERT result is reported in the inspected saved outputs.
