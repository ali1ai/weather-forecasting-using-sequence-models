# Deep Learning for Weather Forecasting Using Sequence Models

Comparative evaluation of Simple RNN, Deep RNN, LSTM, and GRU architectures for short-term weather forecasting under progressively more complex temporal, multivariate, and regional input configurations.

---

## Overview

This project investigates how recurrent neural network architectures perform under different weather-forecasting conditions. The experiments progressively increase forecasting complexity, moving from single-step univariate prediction to multi-step multivariate forecasting using regional weather observations.

The study focuses on three main questions:

1. How does forecasting performance change as the prediction horizon increases?
2. Does jointly modeling temperature and humidity improve forecasting performance?
3. Do weather observations from neighboring regions provide useful additional information for forecasting Basel weather?

Four recurrent architectures are evaluated under a consistent experimental framework:

- Simple RNN
- Deep RNN
- LSTM
- GRU

The models are evaluated using predictive accuracy, forecast-horizon performance, model complexity, training time, and inference time.

---

## Research Objectives

The primary objectives are to:

- Compare recurrent architectures for short-term weather forecasting.
- Evaluate single-step versus multi-step forecasting.
- Compare univariate and multivariate forecasting.
- Investigate the value of regional weather information.
- Examine temperature and humidity forecasting separately.
- Analyze the relationship between predictive accuracy and computational cost.
- Identify the most consistent recurrent architecture across different forecasting scenarios.

---

## Experimental Scenarios

The study consists of four progressively complex forecasting scenarios.

| Scenario | Input Configuration | Prediction Target | Forecast Horizon |
|---|---|---|---|
| **Scenario 1** | Basel temperature | Basel temperature | 1 day |
| **Scenario 2** | Basel temperature | Basel temperature | 3 days |
| **Scenario 3** | Basel temperature + humidity | Basel temperature + humidity | 3 days |
| **Scenario 4** | Regional temperature + humidity | Basel temperature + humidity | 3 days |

### Scenario 1 — Univariate Single-Step

A 14-day historical sequence of Basel temperature observations is used to predict the following day's Basel temperature.

**Task:** `14 → 1`

### Scenario 2 — Univariate Multi-Step

A 14-day historical sequence of Basel temperature observations is used to predict the next three days.

**Task:** `14 → 3`

### Scenario 3 — Multivariate Local Forecasting

Basel temperature and humidity are used as input features to jointly forecast Basel temperature and humidity for the next three days.

**Task:** `14 → 3 × 2`

### Scenario 4 — Regional Multivariate Forecasting

The model receives temperature and humidity observations from Basel and neighboring European locations, including Maastricht, Düsseldorf, München, and De Bilt, to forecast Basel temperature and humidity.

**Task:** `14 → 3 × 2`

This scenario evaluates whether regional weather information provides additional predictive value beyond Basel-only observations.

---

## Dataset

The experiments use historical daily weather observations for European locations including Basel, Maastricht, Düsseldorf, München, and De Bilt.

The study uses:

- Basel temperature
- Basel humidity
- Regional temperature observations
- Regional humidity observations

The dataset contains **3,654 observations**.

The data are treated as a chronological time series. No random shuffling is applied when creating the train/test split.

> **Data note:** The original dataset is not included in this repository. The notebook contains the data-loading and preprocessing workflow required to reproduce the experiments when the source data are available.

---

## Experimental Methodology

### Chronological Train/Test Split

A strict **70/30 chronological split** is used:

- **70% training**
- **30% testing**

Random shuffling is avoided to preserve the temporal structure of the data and prevent future observations from influencing the training period.

### Feature Scaling

Min-Max scaling is applied separately to input and target variables.

Importantly, scalers are fitted **only on the training data** before being applied to the test data.

This prevents information from the future test period from leaking into model training.

### Sequence Generation

A fixed **14-day historical window** is used to construct supervised learning sequences.

For multi-step experiments, the models predict the following **3 days**.

### Training

The recurrent architectures are trained using a consistent experimental configuration to support a fair comparison.

Early Stopping is used with restoration of the best model weights to reduce unnecessary training and limit overfitting.

---

## Models

### Simple RNN

A standard recurrent neural network used as the lightweight baseline.

**Strength:** Low computational cost.

**Limitation:** Less effective at learning long-term dependencies.

### Deep RNN

A stacked recurrent architecture providing greater representational capacity than the Simple RNN.

**Strength:** Increased model capacity.

**Limitation:** Additional depth does not necessarily translate into improved forecasting accuracy.

### LSTM

Long Short-Term Memory networks use gated memory mechanisms to regulate information flow and retain relevant information over longer sequences.

**Strength:** Effective long-term temporal modeling.

**Limitation:** Greater parameter count and computational cost.

### GRU

Gated Recurrent Unit networks provide a simplified gated recurrent architecture using update and reset mechanisms.

**Strength:** Strong predictive performance with fewer parameters than LSTM.

**Limitation:** Still more computationally expensive than a standard RNN.

---

## Results

### Best Model by Scenario

| Scenario | Best Model | Primary Metric | Result |
|---|---|---|---:|
| **Scenario 1** | Deep RNN | RMSE | **2.0286 °C** |
| **Scenario 2** | GRU | RMSE | **2.8579 °C** |
| **Scenario 3** | GRU | Mean normalized RMSE | **0.1089** |
| **Scenario 4** | GRU | Mean normalized RMSE | **0.1073** |

### Overall Architecture Ranking

| Model | Average Rank | Scenario Wins |
|---|---:|---:|
| **GRU** | **1.5** | **3** |
| Deep RNN | 2.5 | 1 |
| LSTM | 3.0 | 0 |
| Simple RNN | 3.0 | 0 |

The GRU achieved the strongest overall performance, ranking first in three of the four scenarios.

---

## Regional Feature Analysis

Scenario 4 investigated whether regional weather observations improve Basel forecasting.

The regional features produced the clearest benefit for **temperature prediction**.

### Temperature RMSE Improvement: Scenario 3 → Scenario 4

| Model | Improvement |
|---|---:|
| **GRU** | **4.05%** |
| LSTM | **3.51%** |
| Simple RNN | **2.06%** |
| Deep RNN | **0.45%** |

The GRU reduced temperature RMSE from **2.8638 °C to 2.7477 °C**.

However, regional information provided little additional benefit for humidity forecasting. This indicates that highly correlated regional temperature observations can provide useful spatial information, while additional regional humidity observations may contain less incremental predictive information for Basel humidity.

---

## Computational Efficiency

The four architectures demonstrate a clear accuracy–computation trade-off.

| Model | Total Training Time | Avg. Training Time | Avg. Prediction Time | Avg. Parameters |
|---|---:|---:|---:|---:|
| **Simple RNN** | 99.41 s | 24.85 s | 0.397 s | 1,300 |
| Deep RNN | 166.61 s | 41.65 s | 0.635 s | 7,620 |
| LSTM | 172.27 s | 43.07 s | 0.553 s | 17,796 |
| **GRU** | 193.74 s | 48.43 s | 0.550 s | 13,604 |

The Simple RNN was the most computationally efficient architecture.

The GRU required the greatest total training time but consistently achieved the strongest forecasting performance.

Therefore:

- **Best computational efficiency:** Simple RNN
- **Best overall forecasting performance:** GRU
- **Best overall accuracy–cost trade-off when accuracy is prioritized:** GRU

---

## Key Findings

### 1. GRU was the strongest overall architecture

The GRU achieved an average rank of **1.5** and won **three of four scenarios**.

### 2. Increasing model complexity did not automatically improve accuracy

The Deep RNN provided greater capacity than the Simple RNN but did not consistently outperform it.

### 3. Temperature was easier to forecast than humidity

Temperature achieved substantially stronger predictive performance than humidity, indicating that atmospheric moisture dynamics were more difficult to capture using the available surface observations.

### 4. Multi-step forecasting became more difficult with increasing horizon

Forecast errors generally increased from **Day +1 to Day +3**, reflecting increasing uncertainty over longer prediction horizons.

### 5. Regional information benefited temperature forecasting

Adding regional weather observations improved temperature prediction, with the largest improvement achieved by the GRU (**4.05% RMSE reduction**).

### 6. Regional information provided limited additional value for humidity

Humidity performance changed very little after adding regional observations, suggesting that additional spatial information was less useful for this target.

---

## Limitations

The study has several limitations:

1. **Single chronological split:** Results are based on one 70/30 temporal split rather than rolling-origin validation.
2. **Limited hyperparameter optimization:** A common configuration was used to maintain architectural comparability.
3. **Fixed 14-day input window:** Other historical windows may capture different temporal dependencies.
4. **Direct multi-output forecasting:** Alternative encoder-decoder and recursive strategies were not evaluated.
5. **Limited meteorological variables:** Additional atmospheric variables such as pressure, precipitation, wind, and radiation were not included.
6. **Single experimental run:** Multiple random seeds would provide stronger estimates of model variability.
7. **Hardware-dependent timing:** Training times depend on the execution environment.
8. **Statistical rather than physical modeling:** The recurrent models learn historical statistical relationships rather than explicitly modeling atmospheric physics.

---

## Future Work

Potential extensions include:

- Walk-forward and rolling-origin validation.
- Multi-seed experimental replication.
- Systematic hyperparameter optimization.
- Evaluation of 7-, 30-, and 60-day input windows.
- Seasonal and cyclical temporal features.
- Additional meteorological variables.
- Encoder-decoder forecasting.
- Temporal and spatial attention mechanisms.
- Temporal Convolutional Networks.
- Transformer-based forecasting models.
- Regional feature selection and regularization.
- Probabilistic forecasting and uncertainty quantification.
- Statistical significance testing of model differences.

---

## Reproducibility

The main experiment is contained in:

[`weather_forecasting_using_sequence_models.ipynb`](weather_forecasting_using_sequence_models.ipynb)

The notebook contains the complete workflow:

1. Data loading
2. Exploratory analysis
3. Chronological train/test splitting
4. Feature scaling
5. Sequence construction
6. Model definition
7. Model training
8. Early stopping
9. Prediction
10. Inverse transformation
11. Evaluation
12. Cross-scenario comparison
13. Computational analysis
14. Visualization
15. Final interpretation

---

## Technology Stack

- Python
- TensorFlow / Keras
- NumPy
- Pandas
- Scikit-learn
- Matplotlib
- Seaborn
- Jupyter Notebook / Google Colab

---

## Project Structure

```text
weather-forecasting-using-sequence-models/
│
├── README.md
├── LICENSE
├── .gitignore
├── requirements.txt
└── weather_forecasting_using_sequence_models.ipynb
