# WHRI: Watershed Health Resilience Index

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Reproducible Research](https://img.shields.io/badge/research-reproducible-green.svg)](#reproducibility)
[![Python](https://img.shields.io/badge/Python-3.10%2B-blue.svg)](#requirements)

## Overview

This repository contains the research code used to compute, validate, assess uncertainty, and diagnose the **Watershed Health Resilience Index (WHRI)** for a watershed-scale spatial assessment.

The workflow is designed for reproducible environmental and watershed-resilience research and integrates:

- geospatial raster and vector data processing;
- fuzzy standardization of environmental and anthropogenic indicators;
- **Fuzzy Analytic Hierarchy Process (FAHP)** expert weighting;
- **Shannon entropy** objective weighting;
- **game-theoretic weight optimization**;
- weighted multi-criteria aggregation for WHRI construction;
- calibration against independent hydro-environmental observations;
- **Random Forest** diagnostic validation and feature-importance analysis;
- variance-based **Sobol global sensitivity analysis**;
- **Monte Carlo uncertainty propagation**;
- spatial uncertainty and priority mapping;
- diagnostic and robustness analyses;
- sensitivity analysis of alternative water-quality indicator combinations.

The complete executable pipeline is distributed as a single consolidated Python script, while the internal workflow remains organized into sequential analytical parts. The supplied pipeline explicitly records its execution order and dependencies between parts. 

> **Research-code status.** This repository is intended to accompany a scientific publication and to support methodological transparency and reproducibility. It is not intended to be a general-purpose Python package.

---

## Scientific workflow

The WHRI workflow consists of the following analytical stages:

| Stage | Component | Main function |
|---|---|---|
| 0–1 | Configuration and inventory | Input discovery, file inventory, indicator mapping, configuration, and data-structure checks |
| 2 | Data loading | Raster/vector/tabular loading, coordinate handling, well selection, and observation preparation |
| 3 | Fuzzy standardization | Transformation of heterogeneous indicators to comparable fuzzy membership values |
| 4 | Weight determination | FAHP, Shannon entropy, and game-theoretic reconciliation of subjective and objective weights |
| 5 | WHRI construction | Weighted aggregation, calibration, classification, and management-zone generation |
| 6 | Random Forest validation | Data-driven diagnostic validation, model comparison, feature importance, and FAHP–RF divergence |
| 7 | Sobol sensitivity | Global variance-based sensitivity analysis of weights, indicator values, fuzzy parameters, and model modifiers |
| 8 | Monte Carlo uncertainty | Probabilistic uncertainty propagation and spatial uncertainty mapping |
| 9 | Two-dimensional synthesis | Joint interpretation of WHRI and uncertainty for spatial priority and management grouping |
| 10 | Diagnostics and robustness | Collinearity, spatial autocorrelation, effective spatial resolution, coverage, calibration, and sensitivity audits |
| 11 | Water-quality sensitivity | Assessment of alternative water-quality indicator combinations |

The implementation documents Random Forest validation as a diagnostic analysis using observed water-level-related targets rather than treating WHRI itself as the primary prediction target. 

The Sobol component uses variance-based global sensitivity analysis and explicitly documents perturbations, weight compositional constraints, fuzzy-range checks, and boundary conditions. 

---

## Methodological framework

### 1. Fuzzy standardization

The pipeline transforms heterogeneous indicators into a common fuzzy-membership scale. Indicator-specific rules distinguish variables for which larger values are interpreted as favorable from variables for which smaller values are favorable.

The implementation includes automated curve selection and parameter fitting, with specific handling of variables such as vegetation indices, water-related indicators, subsidence, well density, anthropogenic stress, and distance-based indicators.

### 2. Hybrid weighting

The weighting framework combines three complementary sources of information:

1. **FAHP** for expert-derived subjective preferences;
2. **Shannon entropy** for data-driven information content;
3. **game-theoretic optimization** for reconciliation of weighting schemes.

The implementation records consistency diagnostics and the final reconciled weights as part of the analysis outputs. The code identifies the weighting framework as `FAHP_Shannon_GameTheory` and specifies a FAHP consistency threshold of 0.10. 

### 3. WHRI construction

The final WHRI is generated using weighted arithmetic aggregation of standardized indicators. The workflow also includes calibration and classification procedures and produces continuous WHRI, classified WHRI, and management-zone rasters. 

### 4. Random Forest diagnostics

Random Forest is used as a **data-driven diagnostic and validation component** rather than as a replacement for the MCDA-based WHRI.

The analysis includes:

- Random Forest regression;
- OLS/Ridge comparison;
- random cross-validation;
- spatial cross-validation;
- blocked leave-one-out diagnostics;
- feature importance;
- partial-dependence analysis;
- comparison between RF-derived importance and FAHP-based weighting.

The pipeline stores model objects, model-comparison tables, direct correlations, FAHP-divergence diagnostics, and a dedicated report. 

### 5. Sobol global sensitivity analysis

The Sobol analysis evaluates the sensitivity of WHRI to uncertainty in:

- criterion weights;
- fuzzy indicator values;
- fuzzy parameters;
- model modifiers;
- calibration-related parameters.

The implementation uses relative perturbations and generates first-order (`S1`) and total-order (`ST`) sensitivity indices. The 12-variable sensitivity design includes five weight variables, five fuzzy indicator variables, and two model-modifier variables. 

The workflow also reports methodological caveats associated with weight compositional constraints, spatial autocorrelation, parameter boundaries, second-order Sobol estimation, and possible fuzzy-range collapse. 

### 6. Monte Carlo uncertainty propagation

Monte Carlo simulation propagates uncertainty through the WHRI framework and produces both global summary statistics and spatial uncertainty layers.

The implementation includes:

- global Monte Carlo iterations;
- tile-based spatial Monte Carlo processing;
- checkpoint/resume functionality;
- mean WHRI;
- standard deviation;
- coefficient of variation;
- 5th, 50th, and 95th percentiles;
- 95% uncertainty intervals;
- convergence diagnostics;
- spatial uncertainty maps.

The number of global iterations is read directly from the analysis configuration, while the spatial phase is processed by tiles to control memory usage.  

### 7. Spatial synthesis

The uncertainty and WHRI dimensions are subsequently combined to produce spatial priority and management-group products. The synthesis explicitly uses uncertainty as a second analytical dimension rather than interpreting WHRI values alone. 

### 8. Robustness and diagnostic analysis

The diagnostic module evaluates:

- variance inflation factors;
- water-quality collinearity;
- NDVI–NDRE collinearity;
- effective spatial resolution;
- Sobol/Monte Carlo scope;
- spatial coverage representativeness;
- Moran's I;
- calibration robustness;
- sensitivity-index consistency.

These analyses are designed to document both strengths and limitations of the resulting WHRI assessment. 

### 9. Water-quality combination sensitivity

The final analytical stage evaluates whether alternative combinations of water-quality indicators materially alter WHRI ranking and sensitivity results. This provides an additional robustness check for the treatment of water-quality information within the index. 

---

## Input data

The pipeline expects a structured local input directory containing geospatial layers and tabular observations.

Typical inputs include:

### Spatial indicators

- NDVI
- NDRE
- EVI
- SAVI
- NDMI
- MNDWI
- NDWI
- land-cover and thematic masks
- DEM
- slope
- aspect
- hillshade
- TWI
- subsidence
- surface-water quality
- groundwater quality
- well density
- aquifer-related information
- anthropogenic stress
- distance to roads
- distance to wells
- composite resilience/stress layers
- fuzzy indicator rasters
- WHRI-related input layers

The configuration stage maintains an explicit indicator name map and path map so that input filenames can be reconciled with standardized analytical names. 

### Tabular observations

The workflow can use:

- groundwater-well observations;
- qanat observations;
- water-quality station observations;
- expert FAHP comparison matrices.

The code includes dynamic column detection for common coordinate, water-level, water-quality, identifier, and temporal fields. 

### Spatial reference

The analysis is CRS-aware and supports reprojection of raster and point datasets before spatial extraction and modelling. The analytical target resolution is configurable; the current configuration specifies a 10 m target resolution and 250 m aggregation for reporting maps. 

---

## Expected input structure

A recommended local structure is:

```text
Desktop/
└── input/
    ├── extract_by_mask/
    ├── gee/
    ├── arcmap/
    ├── agp/
    ├── expert_ahp.xlsx
    ├── wells.xlsx
    ├── water_quality.xlsx
    └── qanats.xlsx
```

The pipeline creates a separate analysis directory:

```text
Desktop/
└── WHRI_output/
    ├── config/
    ├── logs/
    ├── intermediate/
    ├── figures/
    ├── tables/
    ├── models/
    └── reports/
```

The output structure is created automatically by the configuration stage. 

---

## Requirements

The workflow requires Python 3.10 or later and the following principal scientific/geospatial libraries:

```text
numpy
pandas
rasterio
scipy
scikit-learn
matplotlib
geopandas
shapely
SALib
jenkspy
openpyxl
pyproj
```

A pinned or minimum-version `requirements.txt` should be maintained with the repository. The current code-generated environment specification uses minimum versions for the principal dependencies, including NumPy, pandas, Rasterio, SciPy, scikit-learn, GeoPandas, SALib, and related packages. 

Install dependencies with:

```bash
pip install -r requirements.txt
```

For a clean research environment, a dedicated virtual environment or Conda environment is recommended.

---

## Running the pipeline

### 1. Prepare the input data

Place the required raster, vector, and tabular datasets in the input directories described above.

### 2. Configure paths

The principal input and output paths are defined in the configuration section. In the current implementation, the default base directory is the user's Desktop, and the main output directory is `WHRI_output`. 

For use on another computer, modify the path definitions before execution.

### 3. Run the consolidated pipeline

```bash
python FINAL_PART.py
```

The script executes the analytical parts sequentially. Each stage consumes outputs generated by preceding stages, and a failure in an upstream stage prevents subsequent stages from being executed. Because some stages are computationally intensive, the complete workflow may require substantial processing time. 

### 4. Inspect diagnostic reports

After execution, inspect:

```text
WHRI_output/
├── logs/
├── reports/
├── tables/
├── figures/
├── models/
└── intermediate/
```

The generated reports should be checked before using numerical results in a manuscript.

---

## Reproducibility

Reproducibility is supported through:

- explicit configuration parameters;
- fixed random seeds where stochastic procedures are used;
- recorded input/output paths;
- standardized indicator names;
- automated data inventories;
- JSON configuration and result files;
- model-comparison tables;
- diagnostic reports;
- checkpoint/resume mechanisms for long-running Monte Carlo processing.

The current configuration uses random seed `42`. The analysis parameters also record the Random Forest, Sobol, Monte Carlo, fuzzy-standardization, weighting, calibration, spatial, clustering, and output settings. 

For publication-level reproducibility, users should additionally preserve the exact input-data version, software environment, Python version, and repository commit associated with each reported analysis.

---

## Computational considerations

Some stages are computationally demanding. In particular:

- raster-based processing can require substantial RAM and disk space;
- Random Forest diagnostics may involve repeated cross-validation;
- Sobol analysis can generate a large number of model evaluations;
- Monte Carlo spatial propagation is performed in tiles and can be resumed from checkpoints;
- high-resolution raster outputs may require considerable storage.

The consolidated pipeline explicitly warns that total runtime can exceed 12 hours, with the Monte Carlo component identified as a major computational stage. 

For large watersheds or high-resolution datasets, users should verify available memory, disk capacity, and processing time before a full run.

---

## Main outputs

Depending on the completed stages and input availability, the pipeline generates:

### WHRI products

```text
TBM_WHRI_*.tif
TBM_WHRI_*_Classified.tif
TBM_WHRI_*_Zones.tif
```

### Random Forest products

```text
rf_model.pkl
rf_results.json
model_comparison.csv
direct_correlations.csv
fahp_divergence.csv
```

The Random Forest stage explicitly records these outputs in its final summary. 

### Sobol products

Typical products include:

```text
Sobol sensitivity tables
S1 indices
ST indices
indicator sensitivity comparisons
Sobol diagnostic reports
```

### Monte Carlo products

```text
MC mean raster
MC standard-deviation raster
MC coefficient-of-variation raster
P05 raster
P50 raster
P95 raster
Monte Carlo convergence table
Monte Carlo report
```

The Monte Carlo results structure records these spatial uncertainty outputs explicitly. 

### Spatial synthesis products

The synthesis stage generates priority maps, management-group maps, uncertainty summaries, and publication-oriented figures. The implementation reports spatial coverage, uncertainty thresholds, management groups, and associated statistical diagnostics. 

### Diagnostic products

Diagnostic outputs include collinearity, spatial autocorrelation, sampling/coverage, calibration, sensitivity, and methodological-quality reports. 

---

## Interpretation and methodological cautions

The WHRI should be interpreted as a **spatial decision-support index**, not as a direct measurement of watershed health.

Several diagnostics in the workflow are intentionally designed to identify limitations rather than conceal them. In particular:

- RF importance and Sobol sensitivity quantify different aspects of the system and should not be treated as interchangeable;
- weight perturbations are subject to the compositional constraint imposed by renormalization;
- spatial autocorrelation can reduce effective sample size;
- second-order Sobol estimates may require larger sampling designs for stable interpretation;
- indicators with limited spatial variation can produce attenuated sensitivity estimates;
- calibration parameters reaching imposed boundaries should be reported as a methodological diagnostic;
- uncertainty should be interpreted jointly with WHRI rather than ignored after index construction.

The code itself documents these caveats in the Sobol report. 

---

## Data availability and privacy

This repository should contain **code and reproducibility metadata**, not restricted monitoring records or personally identifiable information.

Raw well, monitoring-station, expert-response, or other confidential datasets should only be included if their owners permit public redistribution.

If the original study data cannot be openly distributed, provide:

1. data descriptions;
2. variable definitions;
3. spatial and temporal coverage;
4. preprocessing procedures;
5. access conditions or repository links where applicable;
6. example or synthetic data when appropriate.

---

## Repository organization

A recommended GitHub repository structure is:

```text
WHRI/
├── README.md
├── LICENSE
├── CITATION.cff
├── requirements.txt
├── FINAL_PART.py
├── scripts/
│   ├── PART_0.py
│   ├── PART_1.py
│   ├── PART_2.py
│   ├── PART_3_4_5.py
│   ├── PART_6.py
│   ├── PART_7.py
│   ├── PART_8.py
│   ├── PART_9.py
│   ├── PART_10.py
│   └── PART_11.py
├── config/
│   └── example_config.json
├── docs/
│   └── methodology.md
├── examples/
│   └── README.md
└── outputs/
    └── README.md
```

For the manuscript, the **consolidated `FINAL_PART.py`** can be retained as the archival executable version, while the individual analytical scripts can be included to make the workflow easier to inspect.

---

## Citation

If you use this code, please cite the associated research article:

> **Ebrahimi, P.** *Watershed Health Resilience Index (WHRI): [insert final article title].* [Journal], [year], [volume], [article/pages]. DOI: [insert DOI].

Once the article is published, replace the placeholder citation above with the final bibliographic record.

For software citation, include the GitHub repository version or release tag used for the analysis.

A `CITATION.cff` file is recommended for the repository. The current code already contains a template for generating this file, but its author, affiliation, repository URL, and release metadata are placeholders and should be replaced before publication. 

---

## License

The code is intended to be released under the **MIT License**, subject to confirmation by the copyright holder and completion of the author information in the repository license file.

---

## Contact

**Payam Ebrahimi**  
Assistant Professor, Watershed Management Unit  
Hamadan Agricultural and Natural Resources Research and Education Center  
Agricultural Research, Education and Extension Organization (AREEO)  
Hamadan, Iran

For scientific questions concerning the WHRI methodology, please use the contact information provided in the associated publication.

---

## Versioning

For each published analysis, record:

- repository release/tag;
- Git commit hash;
- Python version;
- dependency versions;
- input-data version;
- analysis date;
- configuration file;
- random seed;
- key methodological parameters.

This information allows the numerical results reported in the manuscript to be traced to a specific computational state of the repository.
