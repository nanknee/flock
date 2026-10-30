# flock
# REFIT Data Cleaning & Washing Machine Energy Disaggregation

**Flock Energy – Data Science Take-Home Assignment**

## 1. Project Overview

This project explores household electricity consumption using the [REFIT Electrical Load Measurements Dataset](https://pureportal.strath.ac.uk/en/datasets/refit-electrical-load-measurements/).

The objective is to develop a reproducible data-processing and analysis pipeline that transforms raw household electricity readings into useful insights about appliance-level energy consumption, with a particular focus on washing-machine usage.

The project is divided into four stages:

1. **Raw Data Inspection & Cleaning:** Inspect and clean House 1 raw REFIT readings, handle duplicate timestamps and missing values, and resample the data to one-minute intervals.
2. **Exploratory Data Analysis:** Analyse washing-machine usage across at least four households using the official cleaned REFIT dataset.
3. **Appliance Disaggregation:** Develop and evaluate a machine-learning model to estimate washing-machine power consumption from aggregate household readings.
4. **Practical Energy Insights:** Identify potential customer-insight and energy-saving opportunities, discuss limitations, and consider applicability to different smart-meter sampling intervals.

The emphasis is on transparent preprocessing, interpretable decisions, leakage-safe evaluation and practical relevance rather than unnecessary model complexity.

## 2. Dataset

The project uses the REFIT Electrical Load Measurements Dataset, which contains aggregate household electricity consumption and individual appliance readings from UK homes.

Two dataset versions are used:

- **Raw REFIT data:** Used for the House 1 inspection and cleaning exercise.
- **Official cleaned REFIT data:** Used for washing-machine exploratory analysis and appliance disaggregation.

### Data characteristics

| Attribute | Description |
|---|---|
| Dataset | REFIT Electrical Load Measurements |
| Location | United Kingdom |
| Household coverage | 20 households in the original dataset |
| Target appliance | Washing machine |
| Sampling resolution | 1 minute for the analysis |
| Main input | Aggregate household power |
| Prediction target | Washing-machine power |

The specific household IDs, date ranges, appliance-channel mappings and valid observation counts used in the final analysis are documented in the results and data reports.

**Note:** The raw and official cleaned datasets are not redistributed in this repository. Refer to `DATA.md` for dataset sources, access instructions and preprocessing details.

## 3. Repository Structure

```text
REFIT-Energy-Disaggregation/
│
├── notebooks/
│   └── Flock_Energy_Analysis.ipynb
│
├── data/
│   ├── raw/                  # Locally downloaded raw data
│   └── cleaned/              # Locally downloaded cleaned data
│
├── output/
│   ├── house1_clean_1min.parquet
│   ├── tables/
│   └── figures/
│
├── README.md
├── DATA.md
├── RESULTS.md
└── Recommendation.md
```

Large datasets and generated files may be excluded from Git using `.gitignore`. The repository contains the code and documentation needed to reproduce the analysis, not the original REFIT data.

## 4. Methodology

### Part 1 – Raw Data Inspection and Cleaning

Raw House 1 data is inspected before applying a simple and documented preprocessing pipeline.

The inspection covers:

- Timestamp parsing and chronological ordering.
- Duplicate timestamps and conflicting readings.
- Missing readings across aggregate and appliance channels.
- Irregular sampling intervals.
- Negative, extreme and potentially implausible power readings.

The cleaning process includes:

1. Loading and combining the available House 1 raw-data files.
2. Parsing timestamps and sorting observations chronologically.
3. Removing exact duplicate observations.
4. Resolving remaining conflicting readings at identical timestamps using a documented aggregation rule.
5. Resampling readings to one-minute intervals.
6. Interpolating only short, internal missing gaps of up to two consecutive minutes.
7. Flagging longer missing intervals rather than filling them without sufficient evidence.
8. Exporting the cleaned dataset and associated quality reports.

Extreme readings are investigated and flagged rather than automatically removed, since unusually high readings may represent genuine appliance activity.

Before-and-after visualisations are generated for aggregate power and selected appliance channels to assess the effect of preprocessing.

### Part 2 – Washing Machine Usage Analysis

The official cleaned REFIT data is used to examine washing-machine consumption across at least four households where the relevant appliance channel is available.

The analysis includes:

- Data availability and date coverage.
- Missing readings and zero/off-time proportions.
- Washing-machine operating-event detection.
- Usage distribution by hour of day and day of week.
- Comparison of cycle characteristics across households.

An operating cycle is identified using a documented power threshold and event-duration rule. Any gap-merging rule used to combine short pauses into a single operating event is also documented.

Visualisations include:

- A representative 24-hour aggregate and washing-machine power trace.
- Detected washing-machine operating cycles.
- Cross-household comparisons of usage and cycle characteristics.

### Part 3 – Appliance Disaggregation Model

A machine-learning approach is used to estimate washing-machine power consumption from the corresponding household aggregate signal.

The modelling pipeline includes:

- Feature construction from aggregate power readings.
- Separation of training, validation and testing data.
- A simple baseline model for comparison.
- A gradient-boosting model using LightGBM.
- Evaluation using appropriate regression and appliance-detection metrics.
- Actual-versus-predicted visualisation on held-out evaluation data.

To reduce data leakage, the evaluation design uses household-based or chronological separation rather than randomly mixing overlapping time windows.

The final report discusses model performance, failure cases and situations where overlapping appliance signatures or unusual household behaviour may reduce prediction accuracy.

### Part 4 – Practical Implications

The final stage translates the analytical findings into a potential utility-facing application.

It considers:

- Opportunities for household energy-awareness and appliance-use insights.
- Possible ways to measure the impact of an energy-saving intervention.
- Limitations caused by overlapping appliance signatures, missing data and household variability.
- Challenges in transferring insights from UK households to Indian households.
- The loss of appliance-level information when using 15-minute or 30-minute smart-meter readings instead of one-minute data.

## 5. Technology Stack

| Tool / Library | Purpose |
|---|---|
| Python | Main programming language |
| Pandas | Data manipulation and preprocessing |
| NumPy | Numerical operations |
| Matplotlib | Data visualisation |
| Scikit-learn | Model evaluation and machine-learning utilities |
| LightGBM | Gradient-boosting model |
| Jupyter Notebook / Google Colab | Development and experimentation |
| Parquet | Storage of processed datasets |

## 6. Setup and Execution

### Requirements

- Python 3.10 or later
- Jupyter Notebook or Google Colab
- Access to the REFIT raw and official cleaned datasets

### Install dependencies

```bash
pip install pandas numpy matplotlib scikit-learn lightgbm pyarrow
```

### Download the datasets

Obtain the datasets from the official University of Strathclyde REFIT source:

- [Raw REFIT dataset](https://pureportal.strath.ac.uk/files/62090177/REFIT_RAW_081116.7z)
- [Cleaned REFIT dataset](https://pureportal.strath.ac.uk/files/62090184/CLEAN_REFIT_081116.7z)
- [Dataset documentation](https://pureportal.strath.ac.uk/en/datasets/refit-electrical-load-measurements/)

Extract the files locally and configure the data-directory paths in the notebook.

### Run the project

1. Clone or download this repository.
2. Install the required Python dependencies.
3. Download and extract the REFIT datasets.
4. Update the dataset paths in the notebook.
5. Open `notebooks/Flock_Energy_Analysis.ipynb`.
6. Execute the notebook cells in order.

The notebook generates cleaned datasets, data-quality reports, exploratory-analysis figures, model predictions and evaluation outputs.

## 7. Assumptions and Design Decisions

- One-minute resolution is used consistently for the washing-machine analysis and disaggregation task.
- Raw House 1 data is used for the initial cleaning exercise, while the official cleaned dataset is used for the subsequent analysis and modelling.
- Exact duplicate observations are removed before resolving remaining conflicting timestamp readings.
- Only short internal missing gaps are interpolated.
- Longer gaps are flagged and excluded where they would compromise analysis or model evaluation.
- Extreme values are investigated rather than removed solely on the basis of statistical thresholds.
- Washing-machine cycle detection uses an explicitly documented threshold and event rule.
- Model evaluation uses a split designed to prevent leakage from overlapping temporal windows.

## 8. Validation Strategy and Trade-offs

The modelling process prioritises realistic evaluation over model complexity.

A simple baseline is compared against the selected machine-learning model to establish whether the model provides meaningful predictive improvement.

Household-based or chronological splits are preferred to random splits because adjacent electricity readings are highly correlated. Randomly distributing nearby observations across training and testing sets can produce overly optimistic results.

The selected gradient-boosting approach offers a practical balance between predictive capability, computational cost and ease of explanation.

The analysis also recognises that aggregate household readings contain overlapping appliance signatures, making exact appliance-level reconstruction inherently challenging.

## 9. AI Tools and Assistance

AI tools were used as development assistance for:

- Structuring the analysis pipeline.
- Reviewing preprocessing and data-quality decisions.
- Debugging Python implementation issues.
- Improving documentation and explaining modelling trade-offs.

All analytical decisions, data processing, code execution and evaluation results should be reviewed and verified against the actual dataset before submission. AI-generated suggestions are not treated as experimental findings without validation.

## 10. Possible Improvements

Potential extensions include:

- Evaluating additional disaggregation approaches such as Random Forest, XGBoost or sequence-based models.
- Improving cycle detection by incorporating state transitions and short inactive gaps.
- Testing generalisation across more households.
- Incorporating additional aggregate-load features and temporal context.
- Evaluating robustness to missing readings and noisy measurements.
- Investigating model performance at 15-minute and 30-minute sampling intervals.
- Validating the approach using electricity-consumption data from Indian households.

## 11. Project Deliverables

| File | Description |
|---|---|
| `README.md` | Project overview, setup and methodology |
| `DATA.md` | Dataset coverage, data quality and preprocessing decisions |
| `RESULTS.md` | Exploratory analysis, model evaluation and findings |
| `Recommendation.md` | Utility-facing recommendations and practical implications |
| Notebook(s) | Reproducible source code and analysis |

---

**Project:** REFIT Data Cleaning & Washing Machine Energy Disaggregation  
**Prepared for:** Flock Energy – Data Science Take-Home Assignment
