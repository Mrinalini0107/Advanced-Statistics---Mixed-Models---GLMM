# Advanced Statistics: Mixed Models – GLMM

> **Author:** Ms. Mrunalini (Data Science Trainer)
> mrunalini0107@gmail.com |  Mumbai – 400095

---
<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/8468cfa4-2ae0-47f5-b21b-8a3456ff9b00" />


## 1. Zero-Inflated Models in R

### Table of Contents

1. **Zero-Inflated Model**
   - Why Zero-Inflated Models?
   - Type of Zero-Inflated Models?
   - Quick Visual: Excess Zeros (R)
   - glmmTMB: How to Specify Zero Inflation
2. **Example: Owl Nestlings Begging**
   - The dataset: Nestling Owls
3. **Import Data and Diagnostics**
   - Data import and changing names
   - Point 1: Data summary
   - Point 1: Visualisation I
   - Point 1: Visualisation II
   - Point 1: Visualisation III
   - Point 1: Visualisation IV
4. **ZIP/ZINB Model Specification**
   - ZIP and ZINB in glmmTMB
5. **Model Diagnostics**
   - DHARMa analysis: QQ Plot
   - DHARMa analysis: residual quartiles plot
   - Influential points
6. **ZIP/ZINB Model Statistics**
   - Model comparison: AIC and Log Likelihood
   - Model output
   - Model simplification
   - Model output: Random effect table
   - Model output: Conditional model
   - Model output: Zero-inflation model
   - Final visualisation
   - "BUT" — a critical look at the results

### Introduction — 1. Zero-Inflated Model

**Why Zero-Inflated Models?**

- Many count datasets show **more zeros** than a Poisson/Negative Binomial model would predict.
- There are two ways a zero can appear:
  1. **Structural zero:** the outcome is not possible — e.g., the process of interest is not present.
  2. **Sampling zero:** the outcome could occur but happened to be zero by chance.

**Type of Zero-Inflated Models?**

The Zero-Inflated model idea (ZIP / ZINB) is a **mixture**:

- With probability π_i (logit-linked), the process is a certain zero.
- Otherwise (probability 1 − π_i), the count comes from a Poisson/Negative Binomial distribution with mean μ_i = exp(X_i β).

Two models are effectively run at once:

- **Count Model** (conditional model): log link for μ_i.
- **Zero-Inflation Model** (ZI model): logit link for π_i.

Both the conditional model and the ZI model contain their own interpretation and information.

Closely related but different: **Hurdle models** — these separate a zero/non-zero hurdle from a truncated count model.

**Quick Visual: Excess Zeros (R)**

We:
- (a) look at a count histogram, and
- (b) compare the observed zero rate to what a plain Poisson GLM would predict.

*Observed counts show excess zeros relative to the Poisson expectation.*

**glmmTMB: How to Specify Zero Inflation**

Formula parts:

- **Conditional (count) model:** `count ~ predictors + (1 | random_effect)`
  - Link: **log** (μ_i = exp(η_i))
- **Zero-inflation model:** `ziformula = ~ predictors_for_structural_zero`
  - Link: **logit** for π_i = P("structural zero")
- **Family:** `poisson`, `nbinom1`, or `nbinom2`

---

## 2. On Labs and Eggs

### Table of Contents

1. Introduction
2. Reading the data into R
3. Data inspection
4. Data visualisation
5. Creating the model
6. Model diagnostics
7. Statistics
8. But these extreme values
9. Simulation

### 1. Introduction — Labs

Consistency between laboratory tests is important, and yet results may depend on **who** did the test and **where** the test was performed. In an experiment to test levels of consistency, a large jar of dried egg powder was divided into a number of samples. Because the powder was homogenized, the fat content of the samples is the same — but this fact is withheld from the laboratories.

Four samples were sent to each of **six laboratories**. Two of the samples were labelled as **G** and two as **H**, although they were in fact identical. The laboratories were instructed to give two samples to two different technicians. The technicians were then instructed to divide their samples into two parts and measure the fat content of each.

> **So each laboratory reported eight measures, each technician four measures — that is, two replicated measures on each of two samples.**

A data frame with **48 observations** on the following 4 variables:

- **Fat:** a numeric vector
- **Lab:** a factor with levels I, II, III, IV, V, VI
- **Technician:** a factor with levels one, two
- **Sample:** a factor with levels G, H

Note that although the samples are identical, they are labelled H and G with a purpose — this allows us to treat sample also as a random effect, nested within technician.

---

## 3. GLMM: GLM + LMM = GLMM

### Table of Contents

1. Introduction
2. Reading the data
3. Data Inspection
4. Data summary
5. Data Visualisation
6. Model Diagnostics
7. Diagnostics

### 1. Introduction

These data come from the book *"Mixed Effects Models with Extensions in Ecology with R"* by Zuur et al. It uses a dataset where species richness — the number of species — was recorded on different beaches, with 5 samples taken from each beach along with its macro-fauna, abiotic variables, and NAP (*nieuw Amsterdams peil*, i.e. height relative to Dutch mean sea level). Variables used:

- **Richness:** number of different species.
- **NAP:** height of sampling station.
- **Beach:** beach ID.
- **Exposure:** classification based on wave action, surf zone, slope, grain size, and depth of the anaerobic layer.

**Generalized Linear Mixed Model (GLMM) Analysis using the RIKZ Dataset**

---

## > **Author:**

Course materials prepared by **Ms. Mrunalini** (Data Science Trainer)
> mrunalini0107@gmail.com |  Mumbai – 400095
