# 🛡️ Network Intrusion Detection System (NIDS) — NSL-KDD

A machine-learning based Network Intrusion Detection System trained on the **NSL-KDD**
benchmark dataset. It compares three classifiers — **Logistic Regression**, **Decision
Tree**, and **Random Forest** — and applies **decision-threshold tuning** so the final
system favors *catching real attacks* over squeezing out a slightly higher raw accuracy.

---

## 📑 Table of Contents

- [Overview](#-overview)
- [Dataset](#-dataset)
- [Project Structure](#-project-structure)
- [Setup](#-setup)
- [How It Works](#-how-it-works)
- [Results](#-results)
- [Why Test Accuracy Is ~77%](#-why-test-accuracy-is-77)
- [Tech Stack](#-tech-stack)
- [License](#-license)

---

## 🔍 Overview

Intrusion detection systems (IDS) classify network traffic as either **normal** or
**malicious**. This project simplifies the classic multi-class NSL-KDD attack labels
(`neptune`, `smurf`, `portsweep`, ...) into a **binary** classification task:

> `0 = Normal traffic` &nbsp;|&nbsp; `1 = Attack`

Three models are trained and compared, the best one (by ROC-AUC) is selected, and its
decision threshold is tuned to prioritize **recall on attacks** — because in security, a
missed attack is far more costly than an extra false alarm.

## 📊 Dataset

This project uses the **[NSL-KDD](https://www.unb.ca/cic/datasets/nsl.html)** dataset
(also available on [Kaggle](https://www.kaggle.com/datasets/hassan06/nslkdd)), an improved
version of the original KDD Cup 1999 dataset. Each row is a network connection record
described by 41 features (protocol, service, byte counts, error rates, etc.) plus a label.

The dataset is **not included** in this repo (per NSL-KDD's distribution terms / file
size). Download `KDDTrain.csv` and `KDDTest.csv` and place them in a local `data/` folder
before running the notebook — see [Setup](#-setup) below.

## 📁 Project Structure

```
nids-nsl-kdd/
├── NIDS_NSL-KDD.ipynb          # main notebook (annotated, section-by-section)
├── docs/
│   └── NIDS_Code_Explanation.pdf   # full line-by-line explanation of the notebook
├── data/                        # (not tracked) place KDDTrain.csv / KDDTest.csv here
├── requirements.txt
├── .gitignore
└── README.md
```

## ⚙️ Setup

```bash
# 1. Clone the repo
git clone https://github.com/<your-username>/nids-nsl-kdd.git
cd nids-nsl-kdd

# 2. Create a virtual environment (optional but recommended)
python -m venv venv
source venv/bin/activate      # Windows: venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Add the dataset
mkdir -p data
# download KDDTrain.csv and KDDTest.csv into data/

# 5. Launch the notebook
jupyter notebook NIDS_NSL-KDD.ipynb
```

## 🧠 How It Works

| Step | What Happens |
|------|--------------|
| **1. Load Data** | Read NSL-KDD's headerless CSVs using a manually declared 42-column schema. |
| **2. Binary Labeling** | Collapse all specific attack types into a single `Attack` class vs `Normal`. |
| **3. Preprocessing** | Label-encode low-cardinality categoricals (`protocol_type`, `flag`); one-hot encode the 70+-value `service` column; scale features with `StandardScaler`. |
| **4. Train Models** | Fit Logistic Regression, Decision Tree, and Random Forest; evaluate with accuracy, ROC-AUC, and 5-fold cross-validation. |
| **5. Threshold Tuning** | Lower the classification cutoff from 0.5 → 0.35 on the best model to boost attack recall. |
| **6. Visualize** | Confusion matrices, accuracy/AUC comparison, ROC curves, and Random Forest feature importances. |
| **7. Live Demo** | Sample individual test rows (including a "mixed-confidence" sample) and walk through real predictions with confidence scores. |

A full line-by-line explanation of every cell is available in
[`docs/NIDS_Code_Explanation.pdf`](docs/NIDS_Code_Explanation.pdf).

## 🏆 Results

| Model | Accuracy | ROC-AUC | CV Accuracy (train) |
|---|---|---|---|
| Logistic Regression | 0.7284 | 0.8303 | 0.9708 ± 0.0006 |
| Decision Tree | 0.7845 | 0.7282 | 0.9981 ± 0.0002 |
| **Random Forest (best)** | 0.7680 | **0.9555** | 0.9990 ± 0.0001 |

**Random Forest** is selected as the best model (highest ROC-AUC). After threshold tuning
(0.5 → 0.35), attack recall improves from **0.613 → 0.696**, at an accuracy of **0.8140**
on the tuned predictions.

<details>
<summary>Classification report (tuned threshold)</summary>

```
              precision    recall  f1-score   support

      Normal       0.71      0.97      0.82      9711
      Attack       0.97      0.70      0.81     12833

    accuracy                           0.81     22544
   macro avg       0.84      0.83      0.81     22544
weighted avg       0.86      0.81      0.81     22544
```
</details>

## ❓ Why Test Accuracy Is ~77%

Cross-validation accuracy during training reaches ~99%, while accuracy on the held-out
`KDDTest.csv` set is only ~77–81%. **This is expected, not a bug:** NSL-KDD's test set
deliberately includes attack types that never appear in the training set, to simulate
real-world "zero-day" attacks. No model — however good — can correctly classify a pattern
it has never seen. Published NSL-KDD binary-classification benchmarks (including deep
learning approaches) typically report test accuracy in the **75–82%** range; a result near
99% would actually be a red flag for data leakage.

## 🛠️ Tech Stack

- Python 3.11
- [pandas](https://pandas.pydata.org/) / [NumPy](https://numpy.org/) — data handling
- [scikit-learn](https://scikit-learn.org/) — models, preprocessing, metrics
- [Matplotlib](https://matplotlib.org/) / [Seaborn](https://seaborn.pydata.org/) — visualization

## 📄 License

This project is licensed under the [MIT License](LICENSE).
