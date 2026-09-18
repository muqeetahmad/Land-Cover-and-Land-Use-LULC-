# Land-Cover-and-Land-Use (LULC)

Multi-decadal Land Use / Land Cover classification and glacier-retreat-aware LULC modification for the **Hunza District, Karakoram**, built on **Google Earth Engine (GEE)** Landsat time series (1995–2025) with a Random Forest classifier, rule-based post-classification correction, accuracy assessment, and elevation-distribution analysis.

## Contents (`LULC.ipynb`)

| Section | Purpose |
|---|---|
| **LULC Modification** | Converts glacier-class pixels to bareland at glacier margins, based on OGGM-projected debris/clean-ice area loss |
| **New LULC** | End-to-end GEE pipeline: multi-sensor Landsat compositing, spectral indices, Random Forest classification, and rule-based correction for 1995–2025 |
| **Accuracy / Kappa Assessment** | Validates the classified Hunza map against independent ground-truth points |
| **LULC Elevation Distribution** | Analyses the elevation distribution of each land-cover class using a DEM |

## LULC classification pipeline

**Study area:** Hunza District, Gilgit-Baltistan
**Years classified:** 1995, 2000, 2005, 2010, 2015, 2020, 2025 (April–October composites, growing season)

**Sensors used per period:**
| Period | Sensors |
|---|---|
| 1995 | Landsat 5 |
| 2000 – 2012 | Landsat 5 + 7 |
| 2013 – 2021 | Landsat 7 + 8 |
| 2022 – 2025 | Landsat 8 + 9 |

**Workflow:**
1. Cloud/shadow-mask each Landsat collection (`QA_PIXEL`) and harmonize Landsat 8/9 bands to the Landsat 4/5/7 band scheme
2. Build an annual median composite per year, clipped to the study area
3. Derive spectral indices — **NDVI, NDWI, NDBI, MNDWI, SAVI, BSI, NDSI**
4. Train a **Random Forest classifier** (`smileRandomForest`, 100 trees, seed 42) per year on labelled training points, using surface reflectance bands + indices as predictors
5. Apply **rule-based post-classification correction** — threshold rules on NDSI/NDVI/NDWI/BSI combined with elevation and slope (from SRTM) to clean up snow/glacier, water, built-up, forest, agriculture, grassland, and bareland misclassification
6. Visualize all years interactively with `geemap`
7. Export per-year and combined multi-band GeoTIFFs, plus a CSV of class area statistics (m², km², hectares) by year, to Google Drive

### LULC classes

| ID | Class |
|---|---|
| 1 | Forest |
| 2 | Agriculture |
| 3 | Grassland |
| 4 | Bare Land |
| 5 | Water Bodies |
| 6 | Built-up |
| 7 | Snow/Glacier |

## Glacier-retreat LULC modification

A companion routine (`modify_lulc_for_glacier_retreat`) takes a classified LULC raster and converts a specified area (km²) of debris-covered (class 7) and clean-ice (class 8) glacier pixels at the glacier margins into bareland (class 4) — driven by projected area loss (e.g. from the paired OGGM glacier-dynamics workflow) — so future-scenario LULC maps reflect expected deglaciation. A validation routine then compares pixel counts and area changes between the original and modified rasters.

## Accuracy assessment

Classifier accuracy is evaluated against an independent Hunza validation point set (not derived from the classifier's own output), using overall accuracy and Kappa statistics computed by GEE's confusion-matrix tools. Validation is restricted to years with independently labelled ground-truth points (2025 by default), since older years cannot be reliably ground-truthed from current imagery.

## Elevation distribution analysis

Resamples a DEM to match the LULC raster's grid, then computes and plots the elevation distribution of each land-cover class — useful for checking classification plausibility (e.g. glacier/snow classes should dominate at high elevation) and understanding the topographic controls on land-cover patterns in the basin.

## Data requirements

- GEE training-point and study-area boundary assets (`FeatureCollection`)
- Independent Hunza validation points (for accuracy assessment), with an integer class ID property
- SRTM DEM (auto-pulled from GEE: `USGS/SRTMGL1_003`)
- A separate DEM raster for elevation-distribution analysis
- A classified LULC GeoTIFF (for the modification/validation routines)

## Dependencies

`earthengine-api` (`ee`) · `geemap` · `rasterio` · `numpy` · `scipy` · `pandas` · `matplotlib` · `GDAL` (`osgeo`)

## Outputs

- `LULC_<year>.tif` — per-year classified LULC GeoTIFFs (1995–2025)
- `LULC_AllYears_Combined.tif` — multi-band GeoTIFF stacking all classified years
- `LULC_Area_Statistics.csv` — class area (m², km², ha) by year
- Modified/glacier-retreat LULC raster + validation report
- Elevation-distribution plots per land-cover class

---

*Land cover component of a broader Hunza Basin glacier and hydrology research workflow, paired with OGGM-based glacier dynamics projections.*
