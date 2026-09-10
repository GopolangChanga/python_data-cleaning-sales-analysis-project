# Superstore Sales — Cleaning & Analysis

An end-to-end project on the Superstore retail sales dataset (9,800 orders,
18 columns): raw-file inspection, data cleaning, and exploratory
visualization.

## Source Data

https://www.kaggle.com/datasets/rohitsahoo/sales-forecasting

## Contents

| File | Description |
|---|---|
| `SuperStore_Cleaning_Script.ipynb` | Full cleaning pipeline — file validation, encoding/size checks, and column-by-column cleaning |
| `visualizations.md` | All chart-generation code (Python/matplotlib) built on the cleaned data |
| `data/cleaned_SuperStore_data.csv` | The cleaned dataset, output of the notebook |

## Cleaning pipeline overview

Before touching the data, the notebook validates the environment first:

1. **Confirms the file exists** at the expected path before attempting to load it
2. **Previews the raw file as plain text** (first 3 lines) to check delimiter and encoding before committing to a parser
3. **Checks file size on disk** to decide whether a chunked load is needed
4. Only then loads the file into a DataFrame with `pandas`

Then, per column:

- Strips whitespace and standardizes casing (`Title Case`) across text fields
  (`Customer Name`, `Segment`, `City`, `State`, `Region`, `Product Name`, etc.)
- Normalizes `Country` and `Category` label variants using explicit mapping
  dictionaries, with case normalized **before** mapping so replacements
  reliably match
- Parses `Order Date` / `Ship Date` with `format='mixed'` and
  `errors='coerce'`, so malformed date strings become `NaT` instead of
  crashing the pipeline
- Fixes `Postal Code` (cast to string, strips trailing `.0` from
  float-parsed values, zero-pads to 5 digits)
- Converts `Sales` to a clean rounded float
- Logs duplicate rows before dropping them, for an audit trail

## Visualization overview

Built on the cleaned dataset — see `visualizations.md` for full code:

1. Sales by Category
2. Sales by Sub-Category
3. Sales by Region
4. Sales by Customer Segment
5. Monthly Sales Trend (latest 12 months)
6. Shipping Delay by Ship Mode
7. Top 10 Customers by Sales

All charts share one consistent style (font, sizing, color) set once via
`matplotlib.rcParams`.


| `screenshots.md` | All 7 chart screenshots in one place |

## Key finding: shipping date data quality issue

Roughly **17% of rows (1,684 of 9,800)** had a `Ship Date` earlier than the
`Order Date` — a data integrity issue, since a product can't ship before
it's ordered. These were identified and capped at 0 days for the shipping
delay calculation rather than silently skewing the averages or being
dropped outright.

## Usage

```bash
pip install pandas numpy matplotlib jupyter
```

1. Place the raw source file at `train.csv`
2. Run `SuperStore_Cleaning_Script.ipynb` top to bottom — it writes the
   cleaned file to `data/cleaned_SuperStore_data.csv`
3. Open `visualizations.md` and run each code block against the cleaned
   file to reproduce the charts

## License

MIT
