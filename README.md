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

### ✅ Section 3: Preprocessing
- Verified categorical columns (`protocol_type`, `service`, `flag`) using `.unique()`: 3 / 70 / 11 distinct values respectively
- One-hot encoded categorical columns using `OneHotEncoder(handle_unknown="ignore")`, fit on train only, applied to both train and test — 84 new columns total
- Verified numeric columns via `.dtypes` before scaling, rather than assuming
- Scaled 38 numeric columns using `StandardScaler`, fit on train only, applied to both train and test
- Merged scaled numeric + encoded categorical + binary label into one final table using `pd.concat(axis=1)`
- Final shapes: `train_final (125973, 123)`, `test_final (22544, 123)`
- 🔑 Key principle applied throughout: **fit only on train, transform both train and test** — prevents data leakage from test into how features are encoded/scaled

- 💾 Saved final processed data to `data/processed_train.csv` and `data/processed_test.csv` for use in the modeling notebook (kept preprocessing and modeling as separate notebooks for clarity)

### ✅ Section 4a: Logistic Regression (baseline)
- Trained `LogisticRegression(max_iter=1000)` on 122 features, 125,973 training rows
- Test set results:

  | Metric | Attack | Normal |
  |---|---|---|
  | Precision | 0.92 | 0.65 |
  | Recall | 0.62 | 0.93 |
  | F1-score | 0.74 | 0.76 |

  Overall accuracy: 75%

- 🔑 Key finding: high precision but low recall on the attack class means the model is **cautious about false alarms but misses real threats** — confusion matrix shows 4,831 real attacks (out of 12,833) were misclassified as normal, a 37.6% miss rate
- In a security context, this kind of false negative (missed attack) is more dangerous than a false positive (false alarm) — this is the gap I want Decision Tree and Random Forest to close
- Possible optimizations identified (not yet applied): adjusting decision threshold, `class_weight="balanced"`, tuning regularization (`C`), or trying non-linear models

![Confusion Matrix - Logistic Regression](images/confusion_matrix_logreg.png)

### 🔜 Next Up
- [ ] Train baseline models: Logistic Regression, Decision Tree, Random Forest
- [ ] Evaluate with precision / recall / F1 / confusion matrix (not just accuracy — class imbalance in attack subtypes)

```
## 🧠 Key Learnings (for interview prep)

- Accuracy alone is misleading on imbalanced data — precision/recall/F1 tell the real story
- Binary-first, multi-class-second is a deliberate strategy, not a shortcut
- The `difficulty` column looked useful at first glance but had to be excluded — good example of "not every column belongs in your model"

## 🐛 Bugs & Fixes

### `.gitignore` not excluding `data/` folder
- **Problem:** Added `data/` to `.gitignore`, but `git status` still showed the data files as untracked/staged
- **Diagnosis:** Used `git check-ignore -v data/` to test whether git recognized the ignore rule — it returned nothing, confirming git genuinely wasn't applying it (not a git status display issue)
- **Fix:** Deleted and recreated `.gitignore` from scratch, typing (not pasting) the contents directly in VS Code, confirmed UTF-8 encoding — likely an invisible character or encoding issue in the original file
- **Lesson:** When a config file "should" work but doesn't, verify with a diagnostic command (`git check-ignore -v`) rather than guessing — and don't rule out the file itself being subtly corrupted


## 📐 Statistics Notes

### Standard Deviation
- **Formula:** `scaled_value = (original_value - mean) / standard_deviation`
- Standard deviation is calculated as: mean → variance (average of squared distances from the mean) → square root of variance
- Standard deviation itself is **always ≥ 0** (built from squared distances, can't be negative)
- Higher std = more spread out data; lower std = data clustered near the mean
- Verified by hand on the `duration` column: mean = 287.14, std = 2604.52 — std being much larger than the mean itself is a signal that this feature has extreme outliers (most connections are short, a few run very long)
- **Common confusion I had:** standard deviation itself is never negative, but *scaled values* (the output of applying the formula to individual rows) absolutely can be negative — a negative scaled value just means that row's original value was below the column's average. These are two different numbers: one describes the whole column's spread, the other describes one row's position relative to that spread.