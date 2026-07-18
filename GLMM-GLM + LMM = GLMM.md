# GLMM: GLM + LMM = GLMM — Species Richness on Beaches (RIKZ Dataset)

## 📘 Description

This project demonstrates, step by step, why and how a **Generalized Linear Mixed Model (GLMM)** combines the strengths of a **Generalized Linear Model (GLM)** — for non-normal response distributions — and a **Linear Mixed Model (LMM)** — for grouped/hierarchical data with random effects. The analysis uses the well-known **RIKZ dataset** from Zuur et al., *"Mixed Effects Models and Extensions in Ecology with R"*, which records marine species richness on Dutch beaches.

Species richness (the number of different species found) was recorded at **5 sampling stations on each of 9 beaches**, along with each station's height relative to mean sea level (`NAP`, *nieuw Amsterdams peil*) and each beach's wave/surf **exposure classification**. Because `Exposure` is a property of the beach itself (a beach cannot have two exposure levels), **beaches are nested within exposure level**, and repeated stations on the same beach are correlated — making this a natural candidate for a mixed-effects approach.

The notebook walks through the complete workflow:

1. Reading and cleaning the RIKZ data (recoding `Beach` and `Exposure` as factors, collapsing a sparse exposure level)
2. Exploratory data summary and visualization (richness vs. `NAP`, faceted by beach and exposure) to reveal the nested design and differing slopes across beaches
3. Fitting a sequence of **linear mixed models (`lmer`)** — random intercept, random slope, and both — assuming a Gaussian response, followed by model comparison via **AIC** and **Likelihood Ratio Tests (LRT)**
4. Diagnosing the LMM residuals and discovering **heteroscedasticity and a right-skewed, Poisson-like pattern** — evidence that a Gaussian LMM is the wrong tool for **count data**
5. Refitting the same nested random-effects structure as a **Poisson GLMM (`glmer`)**, again comparing random-intercept vs. random-slope specifications
6. Diagnosing the GLMM residuals with **DHARMa** simulated residuals, identifying a remaining non-linear pattern
7. Improving the model by adding a **quadratic term for `NAP`** to capture the non-linear relationship between elevation and richness
8. Simplifying the model by removing non-significant fixed and random effects (interaction term, random slope) via **AIC** and **LRT**
9. Selecting and interpreting a **final, parsimonious Poisson GLMM**

This notebook is a practical illustration of the core message in its title: a **GLMM is what you get when you combine a GLM's flexible response distribution with an LMM's random-effects structure** — and of the diagnostic process used to decide, empirically, when that combination is actually needed.

## 🎯 Learning Objectives

By working through this notebook, you will be able to:

1. **Understand conceptually why GLMM = GLM + LMM** — a GLMM extends a GLM's non-Gaussian response distributions (e.g., Poisson for counts) with an LMM's random effects for grouped/nested/repeated-measures data.
2. **Recognize and correctly encode a nested experimental design**, including converting numeric codes into factors and collapsing/relabeling sparse factor levels (`Exposure` level 8 → 10) before modeling.
3. **Fit and compare linear mixed models (`lmer`)** with different random-effects structures — random intercept only, random slope only, and both — using `(NAP | Expos:Beaches)` style notation.
4. **Use AIC and Likelihood Ratio Tests to select among competing random-effects structures**, and recognize potential model *singularity* issues (e.g., a random-slope-only model failing to converge properly).
5. **Diagnose when a Gaussian LMM is inappropriate for count data** — recognizing heteroscedasticity and right-skew in residual plots and simulated-residual diagnostics as signs that a Poisson (or other count) distribution is needed instead.
6. **Fit Poisson GLMMs (`glmer`)** with the same nested random-effects structures used in the LMM stage, and understand why direct comparison of a Gaussian LMM against a Poisson GLMM must rely on diagnostics rather than AIC alone (different response distributions/likelihoods).
7. **Use DHARMa simulated residuals** to diagnose GLMMs specifically, interpreting QQ-uniform plots and residual-vs-predicted plots for signs of remaining misfit (e.g., non-linear/hyperbolic patterns).
8. **Detect and correct non-linear covariate effects** by adding a quadratic term (`I(NAP^2)`) to a GLMM, and re-run diagnostics to confirm the improvement.
9. **Iteratively simplify a GLMM** by removing non-significant fixed-effect interactions and unnecessary random slopes, using AIC and LRT at each step to justify the simplification.
10. **Interpret final GLMM fixed-effect and random-effect estimates and confidence intervals** on the Poisson (log) scale, and use random-effect dot plots (with conditional variances) to visualize beach-level and exposure-level variability.

## 🧩 Dataset

The **RIKZ** dataset contains marine benthic species richness data collected from **9 beaches**, with **5 sampling stations per beach** (45 observations total). Key variables:

| Variable | Description |
|---|---|
| `Richness` | Response — number of different species observed at a sampling station (count) |
| `NAP` | Numeric — height of the sampling station relative to mean sea level (Dutch: *nieuw Amsterdams peil*) |
| `Beach` / `Beaches` | Factor — beach identifier (9 levels) |
| `Exposure` / `Expos` | Factor — beach exposure classification based on wave action, surf zone, slope, grain size, and depth of the anaerobic layer (recoded to 2 levels: 10 and 11) |

Since `Exposure` is a property of the beach, **`Beaches` is nested within `Expos`** — every beach has one and only one exposure level.

## 🛠️ Requirements

This notebook is written in **R** and requires the following packages:

```r
install.packages("ggplot2")
install.packages("lme4")
install.packages("car")
install.packages("lattice")
install.packages("DHARMa")

# Optional, only needed if downloading the data file from Google Drive
install.packages("googledrive")
```

The `RIKZ.txt` data file must be available in the working directory (the notebook shows an optional workflow for retrieving it from Google Drive via `googledrive::drive_download()`).

## 🚀 How to Run

1. Clone this repository and ensure `RIKZ.txt` is present in the working directory.
2. Open `R15_GLMM_GLM_and_LMM_is_GLMM.ipynb` in Jupyter Notebook (with an R kernel installed, e.g. via `IRkernel`) or run the R code chunks directly in RStudio.
3. Run the cells sequentially — later models (`Model1g`–`Model6g`) build on the data preparation and diagnostics established earlier in the notebook.

## 📊 Key Results / Conclusion

- Initial visualization revealed **different regression slopes of richness on `NAP` across beaches**, and confirmed that beaches are nested within the two exposure levels — motivating a mixed-effects (random-intercept and/or random-slope) approach from the start.
- Fitting the data first as a **Gaussian linear mixed model (LMM)**, `Richness ~ Expos + NAP + NAP:Expos + (NAP | Expos:Beaches)`, produced a **model with the lowest AIC using only a random intercept** (Model3), but residual diagnostics (residual-vs-fitted plots, Q-Q plots, and DHARMa simulated residuals) revealed **heteroscedasticity and a right-skewed pattern typical of count data** — clear evidence that treating `Richness` as Gaussian was inappropriate.
- Refitting the same random-effects structures as a **Poisson GLMM (`glmer`)** substantially improved residual behavior. Between the random-intercept-and-slope model (Model1g) and the random-intercept-only model (Model3g), AIC and LRT results were very close, and the **random-intercept-and-slope model was retained** since it preserves information about whether the `NAP` effect varies by beach.
- DHARMa diagnostics on the Poisson GLMM still showed a **residual non-linear (hyperbolic) pattern**, which was resolved by adding a **quadratic term for `NAP`** (`I(NAP^2)`) — after which DHARMa diagnostics showed no further flagged problems.
- Further model simplification via AIC/LRT showed that the **`Expos:NAP` interaction and the random slope for `NAP`** could both be dropped without meaningful loss of fit, leading to a **final, parsimonious model**: `Richness ~ Expos + NAP + I(NAP^2) + (1 | Expos:Beaches)`.
- **Overall conclusion:** species richness on these beaches is well described by a **Poisson GLMM with a quadratic effect of sampling-station elevation (`NAP`)**, a fixed effect of beach exposure, and a simple random intercept for beach nested within exposure — richness is **highest at intermediate `NAP` values and declines at both extremes**, and differs systematically by exposure category, while beach-to-beach variability beyond exposure and elevation is captured by a modest random intercept. The project as a whole illustrates the practical decision process for **when a Gaussian LMM is insufficient and a full GLMM (with an appropriate response distribution) is required**, and how residual diagnostics — not just AIC — should drive that decision.

## 📁 Repository Structure

```
.
├── R15_GLMM_GLM_and_LMM_is_GLMM.ipynb   # Main tutorial notebook
├── RIKZ.txt                              # Dataset (not included; see Requirements)
└── README.md                              # Project documentation (this file)
```

## 📚 Reference

Zuur, A. F., Ieno, E. N., Walker, N., Saveliev, A. A., & Smith, G. M. (2009). *Mixed Effects Models and Extensions in Ecology with R*. Springer. (Source of the RIKZ species richness dataset.)

## 👤 Credit

Notebook prepared by **Ms. Mrunalini** (Data Science Trainer)
📧 mrunalini0107@gmail.com
