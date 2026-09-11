# Cellphone Carrier Neighborhood Analysis

The project asks three questions:

1. How do neighborhoods differ by which carrier(s) have a presence there?
2. Can demographic variables explain or predict the presence of budget carriers (MetroPCS, Cricket, Boost Mobile) in a tract?
3. What types of neighborhoods emerge when tracts are grouped by demographic profile (geodemographic segmentation), and how does each carrier map onto those types?

## Data sources

- **Cellphone store locations**: SafeGraph point-of-interest data for U.S. cellphone/electronics retailers (2021 vintage), pre-filtered and flagged by carrier (Boost Mobile, AT&T, Cricket, MetroPCS, Sprint, T-Mobile, Verizon). This raw file is **not included** in this repo — SafeGraph data requires a commercial license. To recreate this analysis you'll need your own SafeGraph extract (or a substitute POI dataset) with the same carrier flag columns, or you can adapt the carrier-flagging logic to a different retail location source.
- **Census demographics**: 2020–2024 5-year American Community Survey (ACS), pulled at the tract level via the [`pytidycensus`](https://pypi.org/project/pytidycensus/) package, including tract boundary geometries.

## Setup

```bash
pip install pandas geopandas pytidycensus folium matplotlib scikit-learn statsmodels scipy
```

You'll also need a free Census API key: https://api.census.gov/data/key_signup.html

Set it as an environment variable rather than hardcoding it in the notebook:

```bash
export CENSUS_API_KEY="your_key_here"
```

```python
import os
import pytidycensus as tc
tc.set_census_api_key(os.environ["CENSUS_API_KEY"])
```

## Notebook structure

1. **Data importing and inspecting** — load the SafeGraph extract, drop Puerto Rico/Virgin Islands, plot each carrier's locations nationally.
2. **Adding Census data** — pull ACS variables (median household income, population density, race/ethnicity shares, marriage rate, household composition) for all Census tracts, then spatially join store points to tracts.
3. **Descriptive statistics (Q1)** — tract-level medians/std by carrier for each demographic variable.
4. **Logistic regression** — tests whether income, population density, and % people of color predict budget-carrier presence in a tract; includes multicollinearity (VIF), linearity-in-the-logit, and Box-Tidwell diagnostics, plus AUC and Hosmer-Lemeshow calibration checks.
5. **Geodemographic clustering (Q3)** — K-means segmentation of tracts into four neighborhood types based on the same demographic variables, with cluster maps and profile plots.
6. **Carrier–cluster association** — chi-square tests and standardized residuals to see which neighborhood types each carrier over/underrepresents in.

## Notes on reproducing this

- ACS pulls for all 50 states + DC take a while (one API call per state in the notebook) — expect several minutes for that step alone.
- The spatial join between store points and tract polygons isn't perfect; a small number of points (well under 1%) don't fall inside any tract boundary and are handled with a nearest-tract fallback.
- Output CSVs (tract-level data with cluster assignments) are written locally during the notebook run rather than being version-controlled here, since they're large and easily regenerated.
