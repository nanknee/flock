# DATA.md
# Dataset Description and Preprocessing

## 1. Dataset Overview

This project uses the [REFIT Electrical Load Measurements Dataset](https://pureportal.strath.ac.uk/en/datasets/refit-electrical-load-measurements/) from the University of Strathclyde.

Two dataset versions are intended for different stages:

- **Raw REFIT:** House 1 data for inspection and cleaning.
- **Official Cleaned REFIT:** Washing-machine analysis and appliance disaggregation.

The raw House 1 dataset consists of two CSV files containing aggregate consumption and nine appliance channels.

| Attribute | Description |
|---|---|
| Household | House 1 |
| Files | RAW_House1_Part1 and Part2 |
| Original rows | 8,518,763 combined |
| Measurement channels | Aggregate, Appliance1–Appliance9 |
| Timestamp fields | Time and Unix |
| Final resolution | 1 minute |

## 2. Data Quality Assessment

Both House 1 files were combined and inspected.

| Metric | Part 1 | Part 2 |
|---|---:|---:|
| Original rows | 4,377,877 | 4,140,886 |
| Rows with duplicate timestamps | 1,004,404 | 14,755 |
| Conflicting timestamps | 118,191 | 14,362 |

Combined dataset findings:

- Total rows: **8,518,763**
- Unique timestamps: **7,307,715**
- Rows with duplicate timestamps: **2,167,063**
- Distinct duplicated timestamps: **956,015**
- Timestamps with conflicting readings: **313,575**
- Exact duplicate observations: **884,707**
- Intervals longer than 60 seconds: **4,040**
- Invalid Unix timestamps: **0**
- Invalid textual timestamps: **0**
- Timestamp disagreements exceeding one second: **0**
- Negative readings: **0 across all ten measurement channels**

Missing readings were also identified in the raw dataset, with Appliance9 showing the highest number of missing values (1,911,811).

Unusually high readings were observed in multiple appliance channels and flagged for investigation rather than automatically removed.

## 3. Data Cleaning Process

The following preprocessing steps were applied:

1. Combined both raw House 1 CSV files.
2. Converted Unix timestamps into datetime values and sorted observations chronologically.
3. Removed exact duplicate observations.
4. Averaged conflicting readings at identical timestamps.
5. Resampled the data to one-minute intervals.
6. Interpolated complete internal missing gaps of up to two consecutive minutes.
7. Flagged longer missing intervals instead of filling them.
8. Exported the processed dataset in Parquet format.

The resulting one-minute dataset contains **920,031 rows and 11 columns**, including ten measurement channels and a long-outage flag.

## 4. Missing Data and Outlier Handling

Missing values were examined separately for aggregate and appliance channels.

After resampling, missing readings were handled using time-based linear interpolation for short internal gaps. Longer gaps were retained and flagged to avoid introducing misleading consumption values.

No negative readings were found. Extreme readings were retained for further investigation because unusually high consumption may represent genuine appliance activity.

The outage flag indicates that at least one channel has a longer missing interval; it does not necessarily indicate a complete household power outage.

## 5. Washing Machine Dataset

The notebook currently contains the following household mappings and outputs:

| Household | Channel | Date range | Valid coverage |
|---|---|---|---:|
| House 1 | Appliance8 | 09 Oct 2013 – 10 Jul 2015 | 87.31% |
| House 2 | Appliance12 | 01 Jan 2014 – 14 Jan 2014 | 100%* |
| House 3 | Appliance5 | 25 Sep 2013 – 02 Jun 2015 | 88.72% |
| House 4 | Appliance10 | 01 Jan 2014 – 14 Jan 2014 | 100%* |

\*House 2 and House 4 currently use synthetically generated fallback readings in the notebook because the required raw appliance columns were not found. These are not valid REFIT household observations and must be replaced with official cleaned REFIT data before final analysis.

The household mappings and date coverage must also be checked against the official cleaned dataset before being treated as final.

## 6. Remaining Limitations

- Averaging conflicting duplicate readings may smooth genuine short-duration variations.
- Linear interpolation may not represent actual appliance behaviour during missing intervals.
- Long missing intervals reduce usable data.
- Extreme readings may include measurement anomalies.
- Household behaviour and appliance characteristics vary.
- REFIT represents UK households, so its results may not directly generalise to Indian households.
- The current washing-machine analysis and model evaluation require replacement of synthetic fallback data with actual official cleaned REFIT observations.

## 7. Reproducibility

The preprocessing pipeline is implemented in the project notebook.

To reproduce the analysis, download the raw and official cleaned REFIT datasets, extract them into the configured directories, update the dataset paths and execute the notebook sequentially.

The original datasets are not included in this repository.
