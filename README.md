# 🏠 Beekin Rent Model Project
**End-to-end ML workflow: Experiments → Retraining → Monitoring → Rollback**

This project is not just about building a rent prediction model. The goal was to design an ML workflow that behaves like a real production system: we experiment responsibly, refresh the model when new data arrives, measure whether the new version is safe, and only then promote (or roll back) the model.

---

## 🎯 1) What problem this project solves

Rent prediction models can look strong during training but fail later when:
- market conditions shift 📉📈
- new data starts coming in continuously 🗂️
- model refresh happens blindly (without checks) ⚠️

So the real question is:

> ✅ **Can we build a rent model that updates safely over time and stays reliable?**

This project answers that by implementing:
- 🔬 experimentation (baseline vs census features)
- ♻️ retraining pipeline using new 2022 data
- 🧪 monitoring across versions (old vs new model)
- 🔁 rollback decision rule (tolerance-based governance)

---

## 🗃️ 2) Data used

We work with four datasets:

- **train.csv**  
  Historical property transactions used for training the baseline model.

- **test.csv**  
  Holdout dataset used only for evaluation (**never used during training**).

- **data_2022.csv**  
  New incremental market data used to retrain / refresh the model.

- **census.csv**  
  External demographic dataset used only for experimentation (not recommended for deployment).

---


## 🔁 3) How the system works (end-to-end)

The architecture follows a production-style ML lifecycle:

✅ **Data Sources → Ingestion → Features → Training → Registry → Monitoring → Decision/Rollback**

---

### 🔬 A) Experiments — Baseline vs Census
📌 Notebook: `notebooks/experiments.ipynb`

**What happens here**
1. ✅ Baseline model is trained using stable property/location features  
2. 🧩 Census data is merged by blockgroup and used to train an enhanced model  
3. 📊 Both models are evaluated using MdAPE  

**Why we did this**
- census features may introduce bias / explainability issues  
- we only include them if they add **real signal**

**Outcome**
- census did not improve MdAPE meaningfully  
- baseline remained the safer model to deploy  

**Outputs**
- `artifacts/model_comparison_results.json`
- `models/beekin_model_baseline.pkl`

---

### ♻️ B) Retraining / Refresh with new data
📌 Notebook: `notebooks/retrain_2022.ipynb`

**What happens here**
1. historical data is loaded (`train.csv`)  
2. new market data is loaded (`data_2022.csv`)  
3. both datasets are combined into updated training data  
4. same feature engineering + encoding is applied  
5. refreshed model is trained and saved  

**Why we did this**
- real systems require periodic refresh when new data arrives  
- training must remain consistent and reproducible

**Outputs**
- `models/beekin_model_2022.pkl`
- `models/property_type_encoder.pkl`

---

### 🧪 C) Monitoring — Old model vs New model
📌 Notebook: `notebooks/monitor_model.ipynb`

**What happens here**
1. old and new models are loaded  
2. both evaluated on the same holdout dataset  
3. MdAPE computed for both versions  
4. performance delta calculated  
5. tolerance rule applied:
   - ✅ within tolerance → promote new model  
   - ⚠️ beyond tolerance → rollback recommended  

**Why this matters**
- retraining does not guarantee improvement  
- silent degradation is common in production  
- monitoring avoids risky deployment

**Outputs**
- `artifacts/model_monitor_report.json`

---

## 📏 4) Why MdAPE was chosen as metric

MdAPE (Median Absolute Percentage Error) is used because:
- rent values may contain outliers  
- median reduces distortion from extreme points  
- % error is easy to interpret for business stakeholders  

✅ **Lower MdAPE = better model**

---

## 🧠 5) Key decisions & learnings

### ✅ Census features were not deployed
Even if census sounds useful, it increases:
- fairness / bias risk ⚖️  
- explainability complexity 🧩  
- external dependency and maintenance cost 🛠️  

Since performance didn’t improve meaningfully, we recommend:
> ✅ deploy the baseline feature set for stability + safety

---

## 📚 6) Documentation
- 📄 `docs/Beekin_Rent_Model_Memo.pdf`  
  Covers: feature risks, refresh strategy, silent failure detection, monotonicity trade-offs.

- 🧩 Architecture PDFs/diagrams  
  Show full lifecycle and deployment governance.

---

## ▶️ 7) How to run (quick)
1. Open `notebooks/experiments.ipynb` → Run All  
2. Open `notebooks/retrain_2022.ipynb` → Run All  
3. Open `notebooks/monitor_model.ipynb` → Run All  

---

## ✅ Final note
This project demonstrates production-aligned ML thinking:
- reproducible experiments  
- clean refresh pipeline  
- monitoring + rollback governance  
- documentation of risk trade-offs  


