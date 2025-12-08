# **Predictive Maintenance Suite: Failure Classification & RUL Regression**
This repository contains two distinct predictive maintenance projects aimed at optimizing industrial equipment reliability. The projects address two common business problems: predicting immediate failure risks (Classification) and estimating the exact time until failure (Regression).

### **Source:** 
[https://www.kaggle.com/datasets/canozensoy/industrial-iot-dataset-synthetic](https://www.kaggle.com/datasets/canozensoy/industrial-iot-dataset-synthetic)

## **Project 1: Failure Classification (7-Day Window)**
### **Objective**
The goal of this module is to classify whether a piece of industrial equipment will experience a failure within the next **7 days**. This creates an immediate alert system for maintenance teams to react to imminent breakdowns.

### **Feature Engineering Strategy**
The raw dataset contained null values for features specific to certain machine types. To test the models' robustness, I compared two approaches:

**Baseline (Untreated)**: The original dataset with missing values left as-is, relying on the models' default handling mechanisms.

**Enhanced (Binary Flags)**: The dataset enriched with new binary features indicating if a specific sensor reading is applicable to that machine (e.g., `has_laser`, `has_hydraulics`).

### **Key Findings**
- Both XGBoost and LightGBM achieve a balanced accuracy ≈ 0.82 (LightGBM slightly higher), indicating strong overall ability to discriminate between failure / non-failure.
- Similar performance on the baseline (original missing values) and enhanced (binary applicability flags) datasets suggests both tree models handle the dataset's missingness well without heavy preprocessing.
- Models were trained with simple configs (e.g., max_depth=6). Reported performance is promising but likely improvable with tuning and careful validation.

### **Recommendations**
- Prioritize recall (sensitivity) and precision for the failure class, plus PR-AUC and confusion-matrix cost analysis because missing a true failure is far more costly than a false positive.
- Instead of relying only on binary applicability flags, run experiments that impute missing sensor values.
- Run grid/random search or Bayesian optimization (e.g., Optuna) over key hyperparameters (learning rate, num_leaves/estimators, max_depth).


## **Project 2: Remaining Useful Life (RUL) Regression**
### **Objective**
The goal of this module is to predict the exact ***Remaining Useful Life (RUL)*** of equipment in days. This allows for long-term resource planning and inventory management.

### **Handling Imbalanced Regression Data (SMOGN)**
Industrial datasets are often imbalanced; there are many examples of healthy machines with high RUL, but very few examples of machines near failure (low RUL). Standard regression models often fail to predict these rare "near-failure" states accurately.
* **Technique:** I utilized ***SMOGN (Synthetic Minority Over-Sampling Technique for Regression)***. This algorithm generates synthetic data points for the minority region (low RUL values) to help the model generalize better on critical cases.
* **Comparison:** Models trained **with SMOGN** vs. **without SMOGN**.

### **Business-Centric Evaluation**
A global ***R<sup>2</sup>*** score can be misleading in predictive maintenance. Predicting that a machine has 500 days left when it actually has 550 is acceptable. However, predicting 150 days when it actually has 50 is catastrophic.

Planning major maintenance (ordering parts, scheduling downtime) requires approximately **3 months (90-100 days)** lead time. Therefore, accuracy is most critical when the RUL is less than 100 days.

Evaluated the models by segmenting the *R<sup>2</sup>* score:
1. **RUL > 100 Days:** General health monitoring.
2. **RUL < 100 Days:** Critical planning phase.

### **Key Findings**
- Both XGBoost and LightGBM achieve very high overall R2 (≈ 0.97) when trained on the larger dataset but only a small improvement over the SMOGN-augmented small training set.
- SMOGN increases examples in the low-RUL region, improving model exposure to critical cases but did not close the performance gap for RUL < 100 days.
- Current features are snapshot-style; time-series features (lags, slopes, rolling stats) or explicit degradation indicators are not present but could improve low-RUL predictions.

### **Recommendations**
- Include MAE @ < 100 as a top KPI.
- Train with a loss that penalizes errors more strongly when y_true < 100 (weighted MAE/MSE)
- Systematically compare (a) SMOGN and (b) targeted upsampling of real low-RUL examples (if possible).
- Tune hyperparameters.
- Train per-Machine_Type models when enough data exists.