# MSASFormer

Data accompanying the manuscript:

**MSASFormer: A Multi-Scale Adaptive Selection Transformer for Multi-Target Effluent Quality Forecasting in Wastewater Treatment Plants**

This repository provides the simulation dataset used to evaluate MSASFormer for simultaneous forecasting of effluent five-day biochemical oxygen demand (BOD₅), chemical oxygen demand (COD), and total suspended solids (TSS).

## Repository status

The dataset is available in this repository. The model code and instructions for reproducing the experiments are being prepared and are scheduled for release before **1 November 2026**.

## Dataset

**Data file:** `bsm2_open_loop_15min.csv`, provided inside [`bsm2_open_loop_15min.zip`](./bsm2_open_loop_15min.zip). Extract the ZIP archive before loading the CSV.

[Download the dataset ZIP](https://raw.githubusercontent.com/Sameen7777/MSASFormer-data/main/bsm2_open_loop_15min.zip)

The dataset was generated using the open-loop configuration of Benchmark Simulation Model No. 2 (BSM2). It contains simulated process variables and effluent-quality indicators, rather than measurements collected from a full-scale wastewater treatment plant.

The reference BSM2 implementation is available from the [WWTModels repository](https://github.com/wwtmodels/Benchmark-Simulation-Models).

| Property | Description |
|---|---|
| File format | CSV with a header row |
| Number of observations | 34,944 |
| Number of columns | 154, including the time column |
| Sampling interval | 15 minutes |
| Modelling period | Simulation day 245 up to, but excluding, day 609 |
| Time column | `time_days`, expressed in simulation days |

Data before simulation day 245 are excluded from this file. The 15-minute interval represents the resolution of the exported simulation outputs; it does not imply that laboratory measurements of effluent BOD₅ are available at this frequency.

The CSV contains the full set of exported variables. **Only the 19 upstream variables listed below are used as model inputs in the study.** The remaining columns are retained for completeness and should not automatically be included as predictors.

## Model inputs

The following variables are listed in the order used by the model.

| No. | CSV column | Description | Unit |
|---|---|---|---|
| 1 | `influent_q` | Influent flow rate | m³/day |
| 2 | `influent_snh` | Influent ammonium nitrogen | mg N/L |
| 3 | `influent_tss` | Influent total suspended solids | mg SS/L |
| 4 | `primary_effluent_q` | Primary-clarifier effluent flow rate | m³/day |
| 5 | `primary_effluent_snh` | Primary-clarifier effluent ammonium nitrogen | mg N/L |
| 6 | `primary_effluent_tss` | Primary-clarifier effluent total suspended solids | mg SS/L |
| 7 | `reactor2_sno` | Oxidized nitrogen in reactor 2 | mg N/L |
| 8 | `reactor2_snh` | Ammonium nitrogen in reactor 2 | mg N/L |
| 9 | `reactor2_tss` | Total suspended solids in reactor 2 | mg SS/L |
| 10 | `reactor4_so` | Dissolved oxygen in reactor 4 | mg O₂/L |
| 11 | `reactor4_sno` | Oxidized nitrogen in reactor 4 | mg N/L |
| 12 | `reactor4_snh` | Ammonium nitrogen in reactor 4 | mg N/L |
| 13 | `reactor4_tss` | Total suspended solids in reactor 4 | mg SS/L |
| 14 | `reactor5_so` | Dissolved oxygen in reactor 5 | mg O₂/L |
| 15 | `reactor5_sno` | Oxidized nitrogen in reactor 5 | mg N/L |
| 16 | `reactor5_snh` | Ammonium nitrogen in reactor 5 | mg N/L |
| 17 | `reactor5_tss` | Total suspended solids in reactor 5 | mg SS/L |
| 18 | `reactor1_q` | Flow rate in reactor 1 | m³/day |
| 19 | `reactor1_temp` | Temperature in reactor 1 | °C |

Historical and current values of the effluent prediction targets are **not included as model inputs**. The column `time_days` identifies the observation time and is not one of the 19 input variables.

## Prediction targets

| CSV column | Description | Unit |
|---|---|---|
| `effluent_bod5` | Effluent five-day biochemical oxygen demand | mg O₂/L |
| `effluent_cod` | Effluent chemical oxygen demand | mg O₂/L |
| `effluent_tss` | Effluent total suspended solids | mg SS/L |

For each forecast origin, the model uses 96 consecutive observations ending at that origin, corresponding to a 24-hour input window. The primary forecasting task is to predict all three effluent indicators 16 sampling intervals, or 4 hours, ahead.

## Data partitioning and preprocessing

Observations are split chronologically into training, validation, and test periods using a 70%/15%/15% partition. The observations are not randomly shuffled before splitting.

| Partition | Observation rows¹ | Number of observations | Number of prediction samples |
|---|---|---:|---:|
| Training | 1–24,460 | 24,460 | 24,349 |
| Validation | 24,461–29,701 | 5,241 | 5,130 |
| Test | 29,702–34,944 | 5,243 | 5,132 |

¹ Row numbers are one-based and exclude the CSV header. Small differences in partition sizes result from integer rounding.

Sliding windows are constructed separately within each partition, using a stride of one sampling interval. Neither the input window nor its prediction target crosses a partition boundary.

Each input variable and prediction target is standardized using the mean and standard deviation calculated from the training period only. The same training-period statistics are applied to the validation and test data. Target-specific evaluation metrics are calculated after predictions are transformed back to their original physical units.

## Loading the data

The CSV can be loaded in Python using pandas:

```python
import pandas as pd

data = pd.read_csv("bsm2_open_loop_15min.csv")

input_columns = [
    "influent_q",
    "influent_snh",
    "influent_tss",
    "primary_effluent_q",
    "primary_effluent_snh",
    "primary_effluent_tss",
    "reactor2_sno",
    "reactor2_snh",
    "reactor2_tss",
    "reactor4_so",
    "reactor4_sno",
    "reactor4_snh",
    "reactor4_tss",
    "reactor5_so",
    "reactor5_sno",
    "reactor5_snh",
    "reactor5_tss",
    "reactor1_q",
    "reactor1_temp"
]

target_columns = ["effluent_bod5", "effluent_cod", "effluent_tss"]

time = data["time_days"]
inputs = data[input_columns]
targets = data[target_columns]

print("Dataset shape:", data.shape)
print("Input shape:", inputs.shape)
print("Target shape:", targets.shape)
```

This example loads the time series only. Chronological splitting, standardization, and input–target window construction must be performed before model training.

## File integrity

The SHA-256 checksums of the ZIP archive and the extracted CSV are provided in [`SHA256SUMS`](./SHA256SUMS). After downloading the ZIP archive and checksum file, extract the CSV into the same directory and verify both files on macOS with:

```sh
shasum -a 256 -c SHA256SUMS
```

On Linux, use `sha256sum -c SHA256SUMS`.

## Scope

The dataset represents a BSM2 open-loop simulation setting. Results obtained using this dataset do not by themselves establish forecasting performance under feedback-controlled operation or in full-scale wastewater treatment plants.

## Contact

For questions about the dataset, please open an issue in this repository.
