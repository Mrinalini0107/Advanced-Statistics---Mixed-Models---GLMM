# On Labs and Eggs — Nested Random Effects Models in R

## 📘 Description

This project analyzes a classic **inter-laboratory consistency experiment** using **nested random-effects models** (a special case of linear mixed models with no fixed predictors) in R, with the `lme4` and `lmerTest` packages. The dataset (`eggs`, from the `faraway` package) comes from a study designed to test whether fat-content measurements of a homogenized dried egg powder are consistent across different laboratories, technicians, and repeated samples.

A single, homogenized batch of egg powder — with a known, identical true fat content — was split into samples and sent to **six laboratories**. Each lab gave two samples (labelled "G" and "H", though actually identical) to each of **two technicians**, and each technician split their sample into **two replicate measurements**. This nested design (`Sample` nested within `Technician`, nested within `Lab`) produces 48 total measurements and lets us statistically separate how much of the variation in reported fat content is due to:

- differences **between laboratories**,
- differences **between technicians within the same lab**, and
- differences **between repeated samples/measurements** (pure replication/measurement error).

The notebook walks through the complete workflow:

1. Reading and inspecting the `eggs` dataset
2. Verifying the nested structure of `Lab` / `Technician` / `Sample` (and creating a uniquely coded technician factor, since technician labels repeat across labs)
3. Exploratory visualization of fat content by lab, technician, and sample
4. Fitting nested random-effects models with `lmer()` using both shorthand (`(1 | Lab/Technician/Sample)`) and expanded notation
5. Model diagnostics: residual plots, Q-Q plots of residuals and of each random effect, Shapiro-Wilk normality tests, and dot plots of random effects with confidence intervals
6. Formal hypothesis testing of variance components via **Likelihood Ratio Tests (LRT)** to decide whether the `Sample` and `Technician` random effects are needed
7. Comparing REML vs. ML fitting, and computing profile-likelihood confidence intervals for variance components
8. Sensitivity checking by refitting after removing two extreme/outlying observations
9. A **parametric simulation** from the final model, comparing simulated data densities to the observed data density to visually assess model adequacy

## 🎯 Learning Objectives

By working through this notebook, you will be able to:

1. **Distinguish a random-effects (variance components) model from a full mixed model** — recognizing that a model with only an intercept and nested random effects has no fixed-effect predictors (aside from the grand mean).
2. **Verify and correctly specify a nested experimental design** in R, including creating a uniquely coded factor when factor levels (e.g., "Technician one/two") repeat across different groups (e.g., different labs) but do not represent the same underlying unit.
3. **Translate mixed-model shorthand notation** — e.g., `(1 | Lab/Technician/Sample)` — into its fully expanded equivalent, `(1 | Lab) + (1 | Lab:Technician) + (1 | Lab:Technician:Sample)`.
4. **Fit nested random-effects models using `lmer()`**, and understand the difference between fitting with **REML** (for variance-component estimation and inference) versus **ML** (for likelihood-ratio model comparisons).
5. **Explore variance at each level of a hierarchy** — computing and visually comparing the variance of lab means and technician means to build intuition before formal modeling.
6. **Perform mixed-model diagnostics**, including:
   - residual vs. fitted plots to check homogeneity of variance
   - Q-Q plots of residuals *and* of each set of random effects (BLUPs) to check normality
   - Shapiro-Wilk tests for normality of random effects
   - dot plots of random effects with confidence intervals to identify which groups differ meaningfully from zero
7. **Identify and reason about outlying/extreme random-effect levels**, and understand why removing a small number of "unusual" observations from an already small, nested dataset is often unjustified rather than routinely done.
8. **Use Likelihood Ratio Tests to test the significance of variance components**, progressively simplifying a model (full → drop `Sample` → drop `Technician`) to identify the best-supported random-effects structure.
9. **Compute and interpret profile-likelihood confidence intervals** for variance components in a mixed model, and understand how model simplification changes how variance is partitioned (e.g., variance from a dropped `Sample` effect being reabsorbed into `Technician` and residual variance).
10. **Use simulation from a fitted mixed model** (`simulate()`) as an additional, visual model-checking tool, and interpret why small sample sizes can produce large variability between simulated replicate datasets.

## 🧩 Dataset

The **eggs** dataset (from the `faraway` package) contains 48 observations across 4 variables:

| Variable | Description |
|---|---|
| `Fat` | Numeric — measured fat content of the sample |
| `Lab` | Factor — laboratory, levels: I, II, III, IV, V, VI |
| `Technician` | Factor — technician within lab, levels: one, two |
| `Sample` | Factor — sample label, levels: G, H (identical powder, used to test replicate consistency) |

Structure: 6 labs × 2 technicians per lab × 2 samples per technician × 2 replicate measurements per sample = 48 measurements. `Technician` and `Sample` are **nested** within `Lab` (technician "one" in Lab I is not the same person as technician "one" in Lab II), and all factors are treated as random effects since labs, technicians, and samples were effectively randomly selected/allocated.

## 🛠️ Requirements

This notebook is written in **R** and requires the following packages:

```r
install.packages("faraway")
install.packages("dplyr")
install.packages("ggplot2")
install.packages("lme4")
install.packages("lmerTest")
install.packages("car")
install.packages("lattice")
```

## 🚀 How to Run

1. Clone this repository.
2. Open `R14_Eggs_Labs.ipynb` in Jupyter Notebook (with an R kernel installed, e.g. via `IRkernel`) or run the R code chunks directly in RStudio.
3. Run the cells sequentially — later models (`Model`, `ModelLAB`, `Model1`–`Model3`, `Model2` refit, `Model_Out`) build on objects and diagnostics created earlier in the notebook.

## 📊 Key Results / Conclusion

- Exploratory analysis showed **clear differences between laboratory means** (with non-trivial variance across lab means) and **notable differences between technicians within the same lab**, particularly Technician 2 in Lab I, who measured a distinctly higher fat content than other technician/lab combinations.
- The full nested model, `Fat ~ 1 + (1 | Lab) + (1 | Lab:Technician) + (1 | Lab:Technician:Sample)`, was diagnostically sound: residuals were approximately normally distributed and reasonably homogeneous, though the `Sample`-level random effect showed limited variation.
- **Likelihood Ratio Tests** comparing progressively simpler models showed that the `Sample` random effect **could be dropped** without a significant loss of fit, but the `Technician` random effect was needed — meaning **both labs and technicians contribute meaningfully to the inconsistency** in measured fat content, while replicate samples within a technician were highly consistent (i.e., the "hardware"/measurement process of a given technician is the main reproducible source of variation, more so than the specific sample split).
- After settling on the simplified model (`Fat ~ 1 + (1 | Lab) + (1 | Lab:Technician)`), refitting with **REML** shifted some of the variance previously attributed to `Sample` into the `Technician` and residual components, which in turn made the technician-level variance component statistically significant — illustrating how model simplification changes variance partitioning and why the choice between REML and ML fitting matters for correct inference.
- A small number of extreme observations (associated with Lab I) were identified via Q-Q plots of random effects, but the analysis explicitly cautions **against simply discarding such points** in a dataset this small, since doing so risks removing genuine (and informative) lab-to-lab or technician-to-technician variability rather than true noise.
- A **posterior predictive-style simulation** (100 simulated datasets from the fitted model, compared visually to the observed density of `Fat`) showed considerable variability between simulated replicate datasets — a reminder that with only 48 observations spread across a nested design, uncertainty in the estimated variance components is naturally high.
- **Overall conclusion:** inter-laboratory measurement inconsistency in this experiment is driven by **both which laboratory performed the test and which technician within that laboratory did the measurement**, while repeated sample splits by the same technician were highly reproducible. This has a practical real-world implication: quality-control and calibration efforts should focus on standardizing lab and technician-level procedures, since these are the dominant sources of measurement disagreement — not the sampling/replication step itself.

## 📁 Repository Structure

```
.
├── R14_Eggs_Labs.ipynb   # Main tutorial notebook
└── README.md              # Project documentation (this file)
```

## 📚 Reference

`eggs` dataset, distributed with the **faraway** R package (Julian Faraway), commonly used to illustrate nested random-effects / variance-components models.

## 👤 Credit

Notebook prepared by **Ms. Mrunalini** (Data Science Trainer)
📧 mrunalini0107@gmail.com
