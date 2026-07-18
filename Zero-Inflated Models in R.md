# Zero-Inflated Models in R (GLMM) — Owl Nestlings Begging Behaviour

## Description

This project demonstrates how to detect, model, and diagnose **zero-inflated count data** using **Generalized Linear Mixed Models (GLMMs)** in R, with the `glmmTMB` package. The analysis uses the classic **Owls dataset** (Roulin & Bersier, 2007), which records how intensely barn owl nestlings beg for food (`SiblingNegotiation`) depending on which parent is present, food deprivation status, arrival time of the parent, and brood size, with `Nest` treated as a random effect to account for repeated measurements at the same nest.

Count data such as the number of begging calls often contains **more zeros than a standard Poisson or Negative Binomial model would predict**. This notebook walks through the full workflow of identifying that problem and correctly modeling it:

- Visual and numerical diagnosis of excess zeros
- Fitting **Zero-Inflated Poisson (ZIP)** and **Zero-Inflated Negative Binomial (ZINB)** models
- Comparing model families (`nbinom1` vs `nbinom2`)
- Assessing model fit using **DHARMa** simulated residuals (QQ plots, residual vs. predictor plots)
- Identifying influential points using Cook's distance
- Comparing nested models with **AIC** and **Likelihood Ratio Tests (LRT)**
- Simplifying the model by removing non-significant interaction terms
- Interpreting both the **conditional (count) model** and the **zero-inflation model** outputs on the odds/rate-ratio scale
- Critically re-checking residuals of the final chosen model

The notebook is structured as a guided tutorial, moving from *why zero-inflation matters* → *exploratory diagnostics* → *model fitting* → *model diagnostics* → *model comparison/simplification* → *final interpretation*.

##  Learning Objectives

By working through this notebook, you will be able to:

1. **Understand the concept of zero inflation** — distinguish between *structural zeros* (the outcome cannot occur) and *sampling zeros* (the outcome could occur but happened to be zero by chance).
2. **Recognize when a dataset needs a zero-inflated model** by visually and numerically comparing observed zero counts to what a Poisson/NegBin distribution would predict.
3. **Specify and fit Zero-Inflated Poisson (ZIP) and Zero-Inflated Negative Binomial (ZINB) models** in `glmmTMB`, including:
   - The conditional (count) formula with a log link
   - The `ziformula` (zero-inflation) formula with a logit link
   - Random intercepts for grouped/repeated-measures data
4. **Differentiate between `nbinom1` and `nbinom2`** variance structures and know when each is appropriate.
5. **Perform model diagnostics using the DHARMa package**, including simulated residual QQ plots and residual-vs-predictor plots, to assess goodness of fit.
6. **Detect influential observations** using Cook's distance via the `car` package.
7. **Compare and select models** using AIC and Likelihood Ratio Tests (LRT), and simplify a model by removing non-significant interaction terms.
8. **Interpret GLMM output on two different scales simultaneously**:
   - Conditional model coefficients (rate ratios via exponentiation) — effect on the expected count.
   - Zero-inflation model coefficients (odds ratios via exponentiation) — effect on the probability of a structural zero.
9. **Apply a critical eye to model results** — recognizing that a lower AIC or a converged model does not automatically mean an unambiguously "correct" model; residual diagnostics should always be revisited.

##  Dataset

The **Owls** dataset (built into `glmmTMB`) contains 599 observations with the following key variables:

| Variable | Description |
|---|---|
| `SiblingNegotiation` | Response variable — number of begging calls (count) |
| `SexParent` | Sex of the provisioning parent: Female / Male |
| `FoodTreatment` | Food treatment: Deprived / Satiated |
| `ArrivalTime` | Arrival time of the parent (minutes, during the night) |
| `Nest` | Random effect — individual nest identifier |
| `BroodSize` | Number of chicks in the brood |
| `NegPerChick` | Number of negotiations per chick |

##  Requirements

This notebook is written in **R** and requires the following packages:

```r
install.packages("glmmTMB")
install.packages("DHARMa")
install.packages("car")
install.packages("dplyr")
install.packages("ggplot2")
```

## How to Run

1. Clone this repository.
2. Open `R13_Zero_Inflated_Models_in_R.ipynb` in Jupyter Notebook (with an R kernel installed, e.g. via `IRkernel`) or convert/run the R code chunks directly in RStudio.
3. Run the cells sequentially — later models (e.g. `ZINB_Model1_COV`, `ZIP_Final`) depend on objects created earlier in the notebook.

##  Key Results / Conclusion

- The Owls dataset shows a clear excess of zero begging calls, particularly for the **Satiated** food treatment, confirming that a standard Poisson GLMM is inadequate and that zero-inflation modeling is warranted.
- Comparing candidate models by **AIC**, the **ZINB model with `BroodSize` included as a covariate** (`ZINB_Model1_COV`, using the `nbinom1` family) provided the best overall fit, outperforming the plain Poisson, ZIP-intercept-only, and simpler ZIP models.
- **DHARMa residual diagnostics** confirmed that `ZINB_Model1_COV` had no evidence of heteroscedasticity or non-linearity, with only two mild outliers — a substantially better diagnostic profile than the simpler models.
- **Likelihood ratio tests** supported simplifying the model by removing the non-significant `ArrivalTime × SexParent` and `FoodTreatment × SexParent` interaction terms, resulting in a more parsimonious final model.
- Interpreting the **final model (`ZIP_Final`)** on the exponentiated scale:
  - **Conditional model (expected call count):** Satiated nestlings are expected to make about **0.64×** as many calls as Deprived nestlings, holding other variables constant.
  - **Zero-inflation model (probability of a structural zero):** The baseline probability of a structural zero is low (~2.1%), but it **increases with later `ArrivalTime`** (+17.3% odds per unit) and is **much higher for Satiated nestlings** (~5.1× the odds versus Deprived), while it **decreases with larger `BroodSize`** (odds ×0.579 per unit increase).
- The notebook explicitly emphasizes a **critical, not purely automated, modeling mindset**: a good AIC score or successful convergence is not sufficient justification on its own — residual diagnostics (QQ plots, density plots, Cook's distance) must be re-examined even for the "winning" model before trusting its conclusions.
- Overall, the project illustrates that **zero-inflated GLMMs allow two distinct biological/behavioral questions to be answered simultaneously**: what drives the *magnitude* of begging behavior when it occurs, and what drives the *probability that no begging occurs at all*.

## Repository Structure

```
.
├── R13_Zero_Inflated_Models_in_R.ipynb   # Main tutorial notebook
└── README.md                              # Project documentation (this file)
```

##  Reference

Roulin, A., & Bersier, L.-F. (2007). Nestling barn owls beg more intensely in the presence of their mother than in the presence of their father. *Animal Behaviour*, 74(4).

## Credit

Notebook prepared by **Ms. Mrunalini** (Data Science Trainer)
📧 mrunalini0107@gmail.com
