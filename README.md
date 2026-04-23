# U.S. County Birth Trends

This repository analyzes **county-level birth counts in the United States from 2011 to 2020**. The goal is to explain why birth trends differ across counties and to determine how population size, socioeconomic deprivation, rurality, and demographic structure relate to both baseline birth counts and temporal decline.

The project combines **longitudinal count modeling**, **GEE-based inference**, and **principal components regression** on high-dimensional county demographics.

## Why this project matters

National birth counts declined during the 2010s, but local trends were not uniform. Some counties declined more quickly than others, and those differences may reflect more than just population size. This project asks:

1. Do birth counts scale proportionally with county population?
2. Are deprivation and rurality associated with birth rates after adjusting for population?
3. Do these county characteristics modify the rate of decline over time?
4. Does demographic composition add explanatory value beyond broad contextual variables?

## Project highlights

- built a longitudinal county-year panel using natality, population, deprivation, and rural-urban data
- modeled repeated county observations using Generalized Estimating Equations
- evaluated overdispersion and compared Poisson, Gamma, and Negative Binomial variance structures
- used log population as an offset to model birth intensity rather than raw counts
- summarized high-dimensional county demographics with principal components
- showed that deprivation, rurality, and demographic structure all help explain county-level heterogeneity

## Data sources

The analysis integrates several public data sources:

- **CDC WONDER** county-level natality counts, 2011 to 2020
- **SEER** county demographic population tables
- **USDA RUCC** rural-urban continuum codes
- **Area Deprivation Index (ADI)**

The full raw files are not included in the public-facing repo because of size. See `data/README.md` for the expected filenames.

## Methods

### 1. Longitudinal count modeling

Each observation is a county-year record. Since counties are observed repeatedly over time, the main models use **Generalized Estimating Equations (GEE)** with county as the clustering unit.

Core modeling choices:
- log population offset
- centered year variable for trend interpretation
- deprivation and RUCC covariates
- interactions between year and structural county characteristics
- variance-model comparison across Poisson, Gamma, and Negative Binomial specifications

### 2. Demographic structure with principal components regression

County demographic composition is represented using a high-dimensional matrix over race, ethnicity, sex, and age categories. These variables are log-transformed and summarized through SVD/PCA. The leading principal components are then added to the count model to test whether demographic structure improves fit.

## Key results

### Main model estimates

The primary GEE model shows a statistically strong downward time trend, a positive association with deprivation, and interaction effects indicating that county characteristics modify the decline.

| Variable | Coef | Std. Err. | z | p-value |
|---|---:|---:|---:|---:|
| Intercept | -4.4382 | 0.0070 | -621.042 | ≤0.001 |
| Deprivation (Z) | 0.0346 | 0.0070 | 4.653 | ≤0.001 |
| RUCC (centered) | -0.0081 | 0.0080 | -0.959 | 0.338 |
| Year (centered) | -0.0077 | 0.0000 | -17.143 | ≤0.001 |
| Deprivation × Year | 0.0011 | 0.0000 | 2.182 | 0.029 |
| RUCC × Year | -0.0032 | 0.0010 | -6.010 | ≤0.001 |

### Variance structure comparison

The Poisson model shows strong overdispersion, while Gamma and Negative Binomial specifications produce more stable residual behavior.

| Model | Scale parameter | Interpretation |
|---|---:|---|
| Poisson GEE | 130.954 | Strong evidence of overdispersion |
| Gamma GEE | 0.032 | Improved variance stability |
| Negative Binomial GEE | 0.031 | Best residual behavior |

### Principal components regression comparison

Adding demographic principal components substantially improves model fit, though the gains taper at higher dimensions.

| Comparison | p-value |
|---|---:|
| 5 vs 0 factors | ≤0.001 |
| 10 vs 5 factors | ≤0.001 |
| 20 vs 10 factors | ≤0.001 |
| 30 vs 20 factors | ≤0.001 |
| 40 vs 30 factors | 0.000054 |
| 50 vs 40 factors | 0.043915 |
| 60 vs 50 factors | 0.000035 |
| 70 vs 60 factors | 0.414524 |
| 80 vs 70 factors | 0.032338 |
| 90 vs 80 factors | 0.354696 |

## Selected figures

### Population scaling

Birth counts scale almost proportionally with county population on the log scale, which supports using log population as an offset.

![log births vs log population](figures/log%28Births%29%20vs%20log%28Population%29.png)

### Overdispersion diagnostics

The next two figures show why a simple Poisson variance model is not adequate.

<p float="left">
  <img src="figures/Mean%E2%80%93Variance%20Relationship.png" alt="Mean variance relationship" width="48%" />
  <img src="figures/Residual%20Diagnostic%20%28Poisson%29.png" alt="Residual diagnostic Poisson" width="48%" />
</p>

### Better variance behavior under alternative models

The Negative Binomial specification produces more stable residual behavior across the fitted range.

![Residual Diagnostic Negative Binomial](figures/Residual%20Diagnostic%20%28Negative%20Binomial%29.png)

### Principal components regression

The cumulative variance explained curve and PCR fit panels show that a relatively small number of demographic components captures substantial structure.

<p float="left">
  <img src="figures/Cumulative%20PVE.png" alt="Cumulative PVE" width="48%" />
  <img src="figures/PCR%20factors.png" alt="PCR factors" width="48%" />
</p>

<p float="left">
  <img src="figures/PCR%285%20factors%29.png" alt="PCR 5 factors" width="32%" />
  <img src="figures/PCR%2810%20factors%29.png" alt="PCR 10 factors" width="32%" />
  <img src="figures/PCR%2820%20factors%29.png" alt="PCR 20 factors" width="32%" />
</p>

## Main takeaways

- Birth counts scale approximately proportionally with county population
- U.S. counties experienced a statistically robust downward birth trend from 2011 to 2020
- Socioeconomic deprivation remains positively associated with birth intensity after adjustment
- Rurality does not strongly shift baseline birth levels, but it does modify the rate of decline over time
- Demographic composition adds substantial explanatory value beyond deprivation and rurality
- Modeling variance carefully matters because the data are clearly overdispersed relative to Poisson assumptions

## Repository structure

```text
us_county_birth_trends_repo_package/
├── analysis/
│   ├── biplots.ipynb
│   ├── biplots.Rmd
│   ├── pcr.ipynb
│   └── pcr.Rmd
├── docs/
│   └── report.pdf
├── figures/
│   ├── Cumulative PVE.png
│   ├── log(Births) vs log(Population).png
│   ├── Mean–Variance Relationship.png
│   ├── PCR factors.png
│   ├── PCR(5 factors).png
│   ├── PCR(10 factors).png
│   ├── PCR(20 factors).png
│   ├── Residual Diagnostic (Negative Binomial).png
│   └── Residual Diagnostic (Poisson).png
├── scripts/
│   ├── pcr.jl
│   ├── prep.jl
│   ├── prep.py
│   └── prep.R
├── data/
│   └── README.md
└── .gitignore
```

## Reproducibility

### 1. Download the raw data

Download the files listed in `data/README.md` and place them in the `data/` directory.

### 2. Prepare the data

You can use any of the included prep scripts depending on your preferred language:

```bash
python scripts/prep.py
```

or

```r
source("scripts/prep.R")
```

or

```julia
include("scripts/prep.jl")
```

### 3. Run the analysis

Open the notebooks or R Markdown files in `analysis/`.

## Notes

- The original project was developed in a course setting and reorganized here into a portfolio-style repository.
- The public version focuses on code, report, figures, and reproducibility notes rather than shipping the large raw data files.
- The included tables in this README are drawn from the report so readers can understand the main findings quickly.

## Possible next improvements

- add county maps to visualize geographic heterogeneity directly
- refactor the analysis into reusable functions and model wrappers
- include confidence interval plots for the main interaction terms
- provide a lightweight sample dataset for fast replication

## Author

**Hao-Chun Shih**
