# idx-exchange-summer-2026
Work for IDX Exchange Data Science Internship

## Setup

Install the pinned dependencies into the Python environment you'll run the notebooks with:

```
pip install -r requirements.txt
```

These versions are pinned because they're the ones confirmed to work together end-to-end (notably `pandas`/`geopandas`, which have had breaking incompatibilities across versions) — installing into a fresh/unrelated environment instead of matching these versions may not reproduce the notebooks' results.

## Dataset

CRMLS (California Regional Multiple Listing Service) sold-property data, pulled monthly via FTP from IDX Exchange (files prefixed `CRMLSSold`). Filtered to `PropertyType == 'Residential'` and `PropertySubType == 'SingleFamilyResidence'` only. Target variable is `ClosePrice`.

Data files are not in this repo. Notebooks expect the raw monthly CSVs in `/Users/jgd/IDXWORK/datasets/`. Change directories as necessary to reproduce results.

## Preprocessing

- Drop rows missing `LivingArea`, `BathroomsTotalInteger`, `Latitude`, `Longitude`, `LotSizeSquareFeet`.
- Impute `YearBuilt` and `Stories` with the median, `AssociationFee` with 0, boolean features with `False`. All medians computed from train rows only.
- Remove `ClosePrice` outliers using the 0.5th/99.5th percentile of train-only prices, applied to both train and test
- Remove rows outside California lat/long bounds, non-positive `LivingArea`, and unrealistic bed/bath/lot-size values.
- Chronological train/test split (train = everything before the cutoff month, test = the cutoff month onward) — never a random split.

Feature selection: candidate features were dropped for low correlation with `ClosePrice` (below ~0.1) or high null rate (above ~30%). Example: `BuildingAreaTotal` had a strong correlation (0.67) but was excluded anyway since it was ~93% null. Other columns like `TaxAnnualAmount` and `MiddleOrJuniorSchoolDistrict` were 100% null. See `Week2/01_exploration.ipynb` for the full correlation/null-rate check.

Location feature: checked `City`, `MLSAreaMajor`, and `CountyOrParish` as candidates. `City` (~1,000 unique values) and `MLSAreaMajor` (~1,020 unique values) were both too high-cardinality for one-hot encoding, so `CountyOrParish` (60 unique values) was used instead.

Feature engineering added on top of the base physical/location features:
- `BedBathRatio`, `PropertyAge`, `BedroomAreaRatio`
- `DistrictGrouped` — spatial join against CA school district boundaries, grouped to the top districts by count, rest bucketed as `Other`
- `CloseMonthSin`/`CloseMonthCos` — cyclical month encoding

## Models Tested

Linear Regression (baseline), log-transformed Linear Regression, Decision Tree, Random Forest, XGBoost, LightGBM. Full results in [`Week8/metrics_summary.csv`](Week8/metrics_summary.csv):

Log transform was tried because `ClosePrice` is right-skewed (a small number of very expensive homes stretch the distribution) — predicting `log(ClosePrice)` instead of the raw price is a standard way to handle that.

| Model | R² | MAE | MAPE | MdAPE |
|---|---|---|---|---|
| Linear Regression | 0.6901 | $332,998 | 30.35% | 21.98% |
| Log-transformed Linear Regression | 0.6114 | $285,860 | 20.29% | 14.65% |
| Decision Tree (depth 15) | 0.7577 | $239,687 | 18.14% | 11.83% |
| Random Forest (100 trees, depth 15) | 0.8568 | $197,695 | 15.76% | 10.23% |
| XGBoost (tuned) | 0.8851 | $167,793 | 12.70% | 8.48% |
| LightGBM (tuned) | 0.8868 | $180,152 | 14.49% | 10.32% |

R² for the log-transformed model is on the dollar scale for comparability with the rest of the table — see `Week8/06_evaluation.ipynb` for the log-scale R² (0.8147) and why the two differ.

## Best Result

XGBoost (tuned: `n_estimators=200, max_depth=10, learning_rate=0.1`) has the best MAE, MAPE, and MdAPE of every model tested, and wins every price-quintile band on MAPE (see `Week8/06_evaluation.ipynb`). LightGBM has a marginally higher R², but XGBoost is better on every dollar/percentage error metric.

Every model performs worse at both price extremes (entry-level and luxury) than in the middle of the market.

## Notebooks

Run in order:

1. `Week2/01_exploration.ipynb` — EDA
2. `Week3/02_preprocessing.ipynb` — cleaning, original train/test split
3. `Week4/03_baseline_model.ipynb` — Linear Regression baseline
4. `Week5and6/04_model_comparison.ipynb` — Decision Tree, Random Forest, feature engineering, saves enriched train/test CSVs
5. `Week7/05_advanced_models.ipynb` — XGBoost, LightGBM
6. `Week8/06_evaluation.ipynb` — full metric comparison, price-band breakdown, `metrics_summary.csv`

Notebooks 02/03 use the original train/test cutoff (`2026-05-01`). Notebooks 04 onward use a cutoff pushed forward one month (`2026-06-01`) — notebook 04 defines this once as `TRAIN_TEST_CUTOFF`.

Week 9 (optional Streamlit app) was skipped