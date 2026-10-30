# Part 1: Raw Data Inspection and Cleaning

## Objective

The objective of this stage was to inspect and preprocess the raw REFIT electricity consumption data for House 1. The focus was on identifying data-quality issues, handling duplicate timestamps, managing missing values, and preparing the time series for subsequent analysis.

## Dataset Inspection

The raw House 1 dataset contains aggregate household electricity consumption and individual appliance-level readings. Both available data files were examined and combined to obtain the complete observation set.

The following aspects were investigated:

- **Timestamp validity:** Timestamp fields were parsed and checked for invalid or inconsistent values. Unix timestamps were used as the canonical time reference.
- **Chronological ordering:** Observations were sorted by timestamp to ensure correct temporal ordering.
- **Duplicate observations:** Duplicate timestamps were identified, including cases where multiple observations contained identical or conflicting power readings.
- **Missing values:** Missing readings were examined across the aggregate channel and individual appliance channels.
- **Sampling irregularities:** Time differences between consecutive observations were analysed to identify irregular intervals and gaps.
- **Extreme readings:** Negative readings and unusually high power values were investigated to identify potentially erroneous observations.

## Data Cleaning Methodology

A simple and reproducible cleaning pipeline was implemented.

1. **Timestamp conversion:** Unix timestamps were converted into datetime objects and used as the primary time index.
2. **Sorting:** All observations were sorted chronologically.
3. **Duplicate handling:** Exact duplicate observations were removed. Where multiple non-identical readings remained at the same timestamp, the available readings were aggregated using their mean.
4. **One-minute resampling:** The data was resampled into one-minute intervals using the mean of the available observations within each interval.
5. **Short-gap interpolation:** Internal missing intervals of up to two consecutive minutes were interpolated using time-based linear interpolation.
6. **Long-outage handling:** Missing intervals longer than two consecutive minutes were flagged rather than interpolated, preventing the introduction of potentially misleading values.
7. **Extreme-value handling:** Unusually high readings were retained for investigation rather than automatically removed, as they could represent genuine household or appliance activity.

## Before-and-After Comparison

The aggregate household power readings and selected appliance channels were visualised before and after cleaning.

The comparison was used to assess the effects of duplicate removal, timestamp alignment, resampling and short-gap interpolation.

The cleaned dataset provides a consistent one-minute time index while preserving longer periods of missing data for further investigation.

## Remaining Limitations

Although the preprocessing pipeline improves data consistency, some limitations remain. Averaging conflicting readings at identical timestamps may smooth genuine short-duration variations. Linear interpolation can also create plausible but uncertain values during short anomaly periods.

# Part 2: Exploratory Analysis of Washing Machine Usage

## Objective

The objective of this stage was to understand household washing-machine electricity consumption using the official cleaned REFIT dataset.

At least four households with available washing-machine readings were selected. All readings were analysed at one-minute resolution to understand usage frequency, operating patterns and differences between homes.

## Data Availability and Quality

For each selected household, the following characteristics were examined:

- Start and end dates of available observations.
- Total number of observations.
- Missing readings and missing intervals.
- Percentage of time during which the washing machine was inactive.
- Number of detected washing-machine operating events.
- Distribution of washing-machine power consumption.

This assessment helps establish whether the selected households provide sufficient and reliable data for comparative analysis.

The selected households were House [IDs], covering the periods [date ranges].

## Washing Machine Cycle Detection

A threshold-based approach was adopted to identify washing-machine operating events.

A washing-machine reading above the selected power threshold was considered an active observation. Consecutive active observations were grouped into operating events. Very short events were excluded to reduce false detections caused by noise or brief transient spikes.

Where applicable, short inactive gaps between active observations were merged to account for pauses during a washing-machine cycle.

The threshold and minimum event duration were selected based on the observed power distribution and the need to distinguish normal standby consumption from active operation.

**Detection parameters:**

- Active power threshold: [X] W
- Minimum event duration: [X] minutes
- Permitted gap between active periods: [X] minutes

This is a heuristic cycle-detection method rather than a verified record of actual washing-machine programmes.

## Usage Patterns

### Hourly Usage

Washing-machine consumption was grouped by hour of the day to identify periods of relatively high activity.

This analysis helps identify when households tend to operate washing machines and whether usage is concentrated during particular periods.

The observed peak usage period was [hour/period], while comparatively lower usage was observed during [hour/period].

### Weekly Usage

Usage was also grouped by day of the week to identify differences between weekday and weekend activity.

The analysis showed [insert actual finding from the weekly usage plot].

These patterns may provide useful information for understanding household routines and identifying opportunities for energy-awareness interventions.

## Cross-Household Comparison

The selected households were compared using operating-event characteristics such as:

- Number of detected cycles.
- Average cycle duration.
- Maximum power consumption.
- Average power consumption.
- Estimated energy consumption per cycle.

The comparison showed [insert actual comparison].

Differences between households may reflect variations in household size, appliance characteristics, washing habits and operating schedules.

## Visualisation Interpretation

Three types of visualisations were generated:

1. **24-hour aggregate and washing-machine trace:** This visualisation compares total household consumption with washing-machine consumption over a representative day. It illustrates the contribution of machine use to overall demand.
2. **Detected operating cycles:** This visualisation highlights periods identified by the cycle-detection algorithm and demonstrates how the threshold-based method separates active and inactive periods.
3. **Cross-household comparison:** This visualisation compares washing-machine usage characteristics across the selected households, highlighting differences in operating frequency and power consumption.

## Key Observations

The exploratory analysis provides an understanding of household washing-machine usage patterns and demonstrates the importance of appliance-level measurements.

Although aggregate electricity consumption contains information about individual appliance activity, the contribution of a single appliance can be difficult to distinguish when several appliances operate simultaneously or when background demand is high.

# Part 3: Washing Machine Appliance Disaggregation

## Objective

The objective of this stage was to develop a machine-learning model capable of estimating washing-machine power consumption using only the aggregate household electricity consumption signal.

This represents a Non-Intrusive Load Monitoring (NILM) problem, where individual appliance consumption is estimated without requiring separate monitoring equipment for every appliance.

## Problem Formulation

The model uses aggregate household power readings as input features and washing-machine power readings as the target variable.

The task can be represented as:

**Input:** Aggregate household power consumption at one-minute resolution.

**Output:** Estimated washing-machine power consumption at one-minute resolution.

The official cleaned REFIT dataset was used for this stage.

## Feature Engineering

Temporal information from aggregate power readings was used to construct input features.

The feature set includes current and neighbouring aggregate power observations, allowing the model to capture short-term changes in household demand.

The target variable is the corresponding washing-machine power reading.

Missing observations and incomplete feature windows were excluded from model training and evaluation to avoid introducing invalid input-target pairs.

## Model Selection

A LightGBM gradient-boosting regression model was selected for the disaggregation task.

LightGBM is suitable for this problem because it can model non-linear relationships between aggregate electricity consumption and individual appliance demand while remaining computationally efficient.

The model was trained to estimate continuous washing-machine power values rather than simply classifying whether the appliance was active or inactive.

## Baseline Comparison

A simple always-off baseline was included to establish a minimum reference performance.

The baseline predicts zero washing-machine power for every observation.

This provides a useful comparison because washing machines are inactive during a substantial portion of the day. A model that predicts zero consistently may achieve reasonable performance on some metrics, especially when the appliance is idle for long periods.

The machine-learning model was therefore evaluated against the baseline to determine whether it provides meaningful improvement in appliance-level estimation.

## Validation Strategy

A leakage-aware validation strategy was adopted using household-based and time-based data separation.

The training, validation and testing datasets were separated by household IDs or chronological periods to prevent information leakage between time-adjacent observations and across homes.

Randomly mixing neighbouring observations was avoided because consecutive electricity readings are temporally correlated. Such mixing could allow the model to learn patterns from nearly identical observations across train and test splits.

The final evaluation was performed only on data that was excluded from model training and model selection.

## Evaluation Metrics

The model was evaluated using regression metrics and appliance-level detection measures.

- **Mean Absolute Error (MAE):** Measures the average absolute difference between actual and predicted washing-machine power. It is easy to interpret in watts.
- **Root Mean Squared Error (RMSE):** Penalises large prediction errors more strongly and helps identify substantial deviations.
- **R² Score:** Measures how much of the variation in actual washing-machine power is explained by the predictions.
- **Precision and Recall:** When predictions are converted into active/inactive states, these metrics help assess whether the model correctly identifies washing-machine operation.

The metrics are considered together because no single metric fully captures appliance-disaggregation performance.

## Model Performance

The final model achieved the following results on the held-out evaluation dataset:

- MAE: [value] W
- RMSE: [value] W
- R²: [value]
- Precision: [value]
- Recall: [value]

The always-off baseline achieved:

- MAE: [value] W
- RMSE: [value] W
- R²: [value]

The comparison indicates that [insert actual conclusion based on the evaluation results].

An actual-versus-predicted power plot was generated to visually assess the model's ability to reconstruct washing-machine consumption during both active and inactive periods.

## Failure Cases and Limitations

The model may struggle in the following situations:

1. **Overlapping appliance activity:** When several appliances operate simultaneously, their combined power signatures can make it difficult to isolate washing-machine consumption from the aggregate signal.
2. **Similar power signatures:** Other household appliances may exhibit power patterns similar to those of washing machines, leading to false detections or incorrect power estimates.

Additional challenges include differences in appliance ratings, household behaviour and missing observations.

## Conclusion

The modelling exercise demonstrates the potential of machine learning for estimating appliance-level electricity consumption from aggregate smart-meter readings.

The baseline comparison and leakage-aware evaluation provide a more realistic assessment of performance than training and testing on randomly mixed observations.

The results should be interpreted as estimates rather than exact replacements for direct appliance-level measurements.

# Part 4: Practical Implications and Recommendations

## Energy-Saving and Customer-Insight Opportunity

A practical application of washing-machine disaggregation is the development of appliance-level energy insights for residential electricity customers.

Utilities could use estimated washing-machine consumption to help customers understand when and how frequently their appliances operate.

For example, customers could receive periodic reports showing estimated washing-machine usage frequency, approximate energy consumption and recurring usage patterns.

These insights could encourage customers to make more informed decisions about appliance operation and electricity consumption.

## Measuring the Impact

The effectiveness of such an intervention could be measured through a pilot programme.

A group of participating households could receive appliance-level energy reports, while a comparable group continues to receive standard consumption information.

The following indicators could be measured before and after the intervention:

- Average estimated washing-machine energy consumption per household.
- Number of detected washing-machine operating events.
- Changes in operating schedules.
- Overall household electricity consumption.
- Customer engagement with energy reports.

A comparison between the two groups over a defined period would help determine whether providing appliance-level insights leads to measurable changes in energy usage.

The evaluation should account for seasonal changes and differences in household behaviour.

## Limitations

### Overlapping Appliance Signatures

Aggregate household electricity consumption represents the combined demand of multiple appliances. When several appliances operate simultaneously, distinguishing their individual contributions becomes difficult.

### Missing and Noisy Data

Missing observations and irregular readings can affect cycle detection and model predictions. Interpolation may also introduce uncertainty, particularly when longer gaps are present.

### Household Differences

Appliance power ratings, usage schedules, household size and consumer behaviour vary across homes. A model trained on a limited number of households may not generalise equally well to all residential settings.

### UK-to-India Transferability

REFIT represents UK households, whose appliance characteristics, electricity usage patterns, housing conditions and consumer behaviour may differ from those of Indian households.

Indian households may have different washing-machine types, appliance capacities, operating schedules and electricity-consumption patterns.

Therefore, a model developed using REFIT should not be assumed to generalise directly to Indian households without further validation.

Local household data would be required for calibration, retraining and performance assessment.

## Effect of Lower Sampling Resolution

The analysis uses one-minute readings because appliance operation can involve short-duration power changes and distinct operating stages.

If only 15-minute or 30-minute smart-meter readings were available, several challenges would arise.

Short-duration power variations would be averaged within longer intervals, reducing the temporal information available for identifying individual appliance signatures. Different appliances operating in quick succession could become indistinguishable within the same interval.

Washing-machine cycle boundaries would also become less precise, making it more difficult to estimate exact start times, end times, operating duration and peak power.

Consequently, appliance disaggregation at lower sampling resolutions would likely require different feature-engineering strategies, more contextual information and potentially more complex models.

## Final Recommendation

Appliance-level energy insights can provide utilities with an additional way to understand household electricity consumption without installing dedicated monitoring equipment on every appliance.

However, the model should initially be treated as a customer-insight and energy-awareness tool rather than a billing-grade measurement system.

Before large-scale deployment, the approach should be validated on a broader and more diverse set of households, with particular attention to local appliance characteristics, prediction errors and customer utility.
