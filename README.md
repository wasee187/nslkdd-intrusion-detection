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

## 📓 Progress Log

### ✅ Section 1: Data Loading & Exploration
- Loaded train/test files with proper column names (raw files have no headers)
- Confirmed shapes match expected: `train (125973, 43)`, `test (22544, 43)`
- Checked class balance on the `label` column:

  | Class | Count | % |
  |---|---|---|
  | 🟢 `normal` | 67,343 | ~53% |
  | 🔴 attacks (combined, 22 types) | ~58,630 | ~47% |

  - Binary split is **well-balanced** ✅
  - Individual attack types are **severely imbalanced** ⚠️ (`neptune` = 41,214 rows vs `spy` = 2 rows)
  - → This is why the project starts with **binary classification** before extending to multi-class

- 🧹 Noted: the `difficulty` column is metadata about historical classification difficulty — **not a real feature**, excluded from model training

### ✅ Section 2: Binary Label Creation
- Created a new `binary_label` column collapsing all 22 specific attack types into a single `attack` category, alongside `normal`
- Train set: 67,343 normal / 58,630 attack (~53/47 split)
- Test set: 9,711 normal / 12,833 attack (~43/57 split) — **majority flipped compared to train**
- 🔑 Key finding: NSL-KDD's test set deliberately includes attack patterns not seen in training, to test generalization rather than memorization — a harder, more realistic evaluation than train/test sets with matching distributions

### 🔜 Next Up
- [ ] Preprocessing: encode categorical features, scale numeric features
- [ ] Train baseline models: Logistic Regression, Decision Tree, Random Forest
- [ ] Evaluate with precision / recall / F1 / confusion matrix (not just accuracy — class imbalance in attack subtypes)

```
## 🧠 Key Learnings (for interview prep)

- Accuracy alone is misleading on imbalanced data — precision/recall/F1 tell the real story
- Binary-first, multi-class-second is a deliberate strategy, not a shortcut
- The `difficulty` column looked useful at first glance but had to be excluded — good example of "not every column belongs in your model"