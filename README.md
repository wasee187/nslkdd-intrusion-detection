# 🛡️ NSL-KDD Network Intrusion Detection

> Binary classification of network connections as **normal** or **malicious**, using the NSL-KDD dataset.


---

## 📊 Dataset

| | |
|---|---|
| **Source** | NSL-KDD (improved version of KDD Cup 1999) |
| **Official page** | [unb.ca/cic/datasets/nsl.html](https://www.unb.ca/cic/datasets/nsl.html) *(currently down — mirrors used instead)* |
| **Train set** | 125,973 rows × 43 columns |
| **Test set** | 22,544 rows × 43 columns |
| **Columns** | 41 features + `label` + `difficulty` |
| **Unit** | Each row = **one network connection** |

---

## ⚙️ Setup

```bash
python -m venv venv
venv\Scripts\activate
pip install pandas numpy scikit-learn matplotlib seaborn jupyter notebook
```

---
## 📌 Project Status

- ✅ **Binary classification** (normal vs. attack) — complete
- 🔄 **Multi-class classification** (5 categories: normal, DoS, Probe, R2L, U2R) — in progress

## 🏆 Results (Binary Classification)

Three models trained and compared: Logistic Regression, Decision Tree, and Random Forest (default + tuned).

| Model | Attack Precision | Attack Recall | Attack F1 | Accuracy |
|---|---|---|---|---|
| Logistic Regression | 0.92 | 0.62 | 0.74 | 75% |
| **Decision Tree** | 0.96 | **0.65** | **0.77** | **79%** |
| Random Forest (default) | 0.97 | 0.61 | 0.75 | 77% |
| Random Forest (tuned) | 0.97 | 0.63 | 0.76 | 78% |


- 🔄 **Multi-class classification** (5 categories: normal, DoS, Probe, R2L, U2R) — preprocessing in progress

**Key findings:**
- Decision Tree outperformed Random Forest, even after tuning — a reminder that simpler models can beat more complex ones on certain data
- Feature importance analysis (Random Forest + Decision Tree structure) showed the model relies on sensible signals: traffic volume (`src_bytes`), service-behavior consistency, and login status
- Hyperparameter tuning revealed a significant overfitting gap: 99.9% cross-validation accuracy dropped to 78% on the real test set — a concrete example of why proper train/test evaluation matters

- ✅ **Multi-class classification** — complete. All three models trained and compared; Decision Tree best overall (76% accuracy); R2L and U2R remain hard to detect across all models due to severe class imbalance

## 📖 Full Methodology & Learning Notes

For a detailed, section-by-section breakdown of every step — including debugging notes, statistics explanations, and what was learned along the way — see [LEARNING_LOG.md](LEARNING_LOG.md).