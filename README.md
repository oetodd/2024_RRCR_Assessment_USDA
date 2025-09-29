# Rhizoctonia Disease Field Evaluation Analysis

## Overview

R implementation for analyzing Rhizoctonia disease resistance in sugar beet field trials (2024 1R evaluation). This project replicates and modernizes legacy SAS analysis (PROC MIXED) using open-source R tools.

## Statistical Methods

### Experimental Design

- Completely randomized design (CRD)
- 5 replications per entry
- Non-parametric rank-based analysis

### Analysis Pipeline

**1. Data transformations**

- Arcsine square-root transformation: 57.2958 * asin(sqrt(proportion))
- Converts proportions to percentages
- Ranks MedianRating values for non-parametric analysis

**2. Primary analysis**

- One-way ANOVA on ranked disease severity ratings
- Tests hypothesis: Are there differences in disease resistance between entries?

**3. Post-hoc comparisons**

- Dunnett's tests comparing all entries against control varieties (FC901, FC709-2, FC709-4)
- Controls family-wise error rate at alpha = 0.05

**4. Summary statistics**

- Least squares means for each entry
- Descriptive statistics by treatment group

## Requirements

**R version:** 4.0.0 or higher

**Required Packages:**

```r
install.packages(c("dplyr", "emmeans", "car", "nlme"))
```

## Input File

CSV file with the following columns (SAS code uses an Excel file):

| Column | Type | Description |
|--------|------|-------------|
| Entry | character | Genotype identifier |
| Rep | numeric | Replication number (1-5) |
| MedianRating | numeric | Disease rating rounded to nearest integer |
| PHLTHY | numeric | Proportion healthy (0-1 DI / total plants) |
| PHRVST | numeric | Proportion harvestable (0-3 DI / total plants) |
| DI | numeric | Disease Index rating (weighted average) |

## Control Entries

- **FC901**: Susceptible check variety
- **FC709-2**: Resistant check variety
- **FC709-4**: Highly Resistant check variety

## Output

The analysis produces:

- ANOVA table with F-statistic and p-value
- Least squares means table (ranked values by entry)
- Dunnett's test results showing:
  - Comparison of each entry vs controls
  - Estimate, standard error, t-value, p-value
  - Significance indicators

## Why Rank-Based Analysis?

Disease severity ratings are ordinal data where:

- We don't know if the "disease gap" between 1 -> 2 equals 2 -> 3
- But we do know 3 > 2 > 1

Ranking preserves the relative ordering without assuming equal intervals, making this a non-parametric approach appropriate for ordinal disease scales.

## Why lm() instead of PROC MIXED?

The original SAS analysis uses PROC MIXED (mixed model), but this R implementation uses `lm()` (linear model). For this completely randomized design with fixed effects only, both approaches give equivalent results for:

- Overall treatment tests
- Least squares means
- Multiple comparisons (Dunnett's tests)

SAS PROC MIXED (and mixed models in general) can accommodate heterogeneous variances (different variance for each treatment group), which is theoretically more flexible than the homogeneous variance assumption in `lm()`. However, attempts to fit a heterogeneous variance model in R using `nlme::gls()` resulted in convergence failures, suggesting variances do not differ substantially enough between entries to justify the additional complexity.

The `lm()` approach is simpler, more stable, and maintains the same rank-based non-parametric methodology. For similar field trials with this experimental design, `lm()` and heterogeneous variance models produce similar p-values and treatment comparisons, leading to the same practical conclusions about which entries differ from controls.

**Note:** A heterogeneous variance model using `nlme::gls()` is included in the commented code for those interested in exploring this approach. In our analysis, this model failed to converge, suggesting variances are similar across groups. This may also occur with similar datasets when variances do not differ substantially between entries.

## Project History

- **Original analysis**: Developed in SAS by Kathleen Yeater (USDA)
- **Migration to R**: 2025, for improved accessibility and reproducibility
- **Dataset**: 2024 Rhizoctonia field evaluation (1R)

## Contact

**Sam McNeill**  
Samuel.McNeill@usda.gov  
USDA Agricultural Research Service

## Acknowledgments

Statistical methodology based on:

Shah, D. A., and Madden, L. V. 2004. Nonparametric Analysis of Ordinal Data in Designed Factorial Experiments. Phytopathology 94:33-43. DOI: 10.1094/PHYTO.2004.94.1.33

**Original SAS analysis:** Kathleen Yeater, USDA-ARS  
**R migration:** Sam McNeill, USDA-ARS, 2025, Fort Collins, CO