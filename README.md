# Motor Insurance Pricing Model and Portfolio Analytics

## Overview
An end-to-end actuarial pricing project built on 678,013 motor insurance policies.
The project covers data validation, SQL-based portfolio analysis, claim frequency
modelling, model validation, and portfolio performance monitoring.

---

## Dataset
**French Motor Third-Party Liability (freMTPL2freq)** — OpenML ID 41214

| Feature | Description |
|---------|-------------|
| ClaimNb | Number of claims in the policy period |
| Exposure | Fraction of policy year observed (0 to 1) |
| BonusMalus | Accumulated claims history score |
| DrivAge | Driver age in years |
| VehAge | Vehicle age in years |
| VehPower | Vehicle engine power |
| VehGas | Fuel type — Diesel or Regular |
| Density | Population density of policyholder's municipality |
| Area / Region | Geographic location indicators |

---

## Tools
Python, SQLite, pandas, numpy, statsmodels, XGBoost, scikit-learn, matplotlib, seaborn

---

## Key Equations

**Claim Frequency**

$$\text{Claim Frequency} = \frac{\text{Total Claims}}{\text{Total Exposure}}$$

**Poisson GLM — Log Link with Exposure Offset**

$$\log\left(\frac{\mu_i}{e_i}\right) = \beta_0 + \beta_1 x_{i1} + \beta_2 x_{i2} + \cdots + \beta_p x_{ip}$$

where $\mu_i$ is the expected claim count, $e_i$ is the exposure, and
$x_{i1}, \ldots, x_{ip}$ are the rating factors. Rearranging:

$$\log(\mu_i) = \log(e_i) + \beta_0 + \sum_{j=1}^{p} \beta_j x_{ij}$$

The term $\log(e_i)$ is the **offset** — it enters the model with a fixed
coefficient of 1, ensuring predicted claims scale proportionally with exposure.

**Multiplicative Rating Structure**

$$\text{Pure Risk Premium}_i = e^{\beta_0} \cdot e^{\beta_1 x_{i1}} \cdot e^{\beta_2 x_{i2}} \cdots e^{\beta_p x_{ip}}$$

Each rating factor contributes a multiplicative relativity to the base rate —
the standard structure of a general insurance pricing model.

**Loss Ratio**

$$\text{Loss Ratio} = \frac{\text{Expected Claims} \times \text{Average Severity}}{\text{Earned Premium}}$$

**Risk Relativity**

$$\text{Relativity}_k = \frac{\text{Claim Frequency}_k}{\text{Claim Frequency}_{\text{base}}}$$

**Actual vs Expected**

$$\text{A/E Ratio} = \frac{\sum \text{Predicted Claims}}{\sum \text{Actual Claims}}$$

A ratio of 1.0 indicates perfect calibration. Values above 1.0 indicate
over-prediction; below 1.0 indicate under-prediction.

---

## What This Project Covers

**Data preparation and SQL analytics**
Cleaned and loaded into a SQLite database. SQL queries produce portfolio
summaries by area, bonus-malus band, and vehicle power — replicating
a real insurance data warehouse workflow.

**Exploratory analysis**
Claim frequency analysed across all rating factors. Bonus-malus and driver
age emerged as the strongest predictors. Log-transformed density outperformed
broad area categories, rendering Area and Region statistically insignificant
in the GLM.

**Pricing models**

| Model | Predicted/Actual | MAE | RMSE |
|-------|-----------------|-----|------|
| Poisson GLM | 1.000 | 0.0990 | 0.2381 |
| XGBoost Poisson | 1.057 | 0.1004 | 0.2367 |

The GLM uses a log exposure offset — the actuarial standard for frequency
modelling — achieving perfect portfolio calibration. XGBoost uses the same
offset as a base margin and captures non-linear relationships the GLM misses.

**Actual vs Expected validation**
Models validated on a held-out test set of 135,358 policies across four
rating factor bands. The GLM under-predicts young drivers (A/E 0.878) and
new vehicles (A/E 0.815) — XGBoost tracks both segments more accurately,
identifying where GLM refinements are warranted.

**Portfolio analytics and risk segmentation**
Loss ratios and risk relativities computed by segment. High bonus-malus
drivers and young drivers carry the largest relativities — a flat-rate
portfolio ignoring these factors would systematically attract adverse selection.

---

## Key Findings
- Bonus-malus score is the single strongest claim frequency predictor
- New vehicles (0-1 years) are materially under-priced by the GLM — a banded vehicle age term is warranted
- XGBoost outperforms the GLM in three segments: young drivers, new vehicles, and high bonus-malus policyholders
- Both models agree that vehicle power is a broadly linear risk factor, well captured by the GLM
