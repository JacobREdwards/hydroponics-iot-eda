# Hydroponics IoT EDA and Preprocessing

Exploratory data analysis and preprocessing workflow for IoT sensor data from an autonomous hydroponics system.

## Overview
This repository focuses on my contribution to a team project on hydroponics IoT analytics. My work centered on exploratory data analysis, data cleaning, missing-value handling, segmentation of time-series gaps, and feature engineering for downstream modeling.

One downstream goal of the broader project was to help predict **pH drift before it entered a critical range**. In hydroponic systems, pH is one of the most important variables to monitor because it directly affects nutrient solubility and plant uptake. If pH drifts too far outside the target range, nutrients can become less available even when they are physically present in the water, which can reduce plant growth and create misleading signs of deficiency. For many hydroponic systems, a pH range around **5.5 to 6.5** is commonly treated as the workable zone, so tracking and anticipating drift is operationally important.

## My Contribution
I led the EDA and preprocessing portion of the project, including:
- loading and inspecting raw IoT sensor data
- cleaning timestamps and numeric sensor fields
- identifying outages and segmenting continuous sensor runs
- handling placeholder zero values in water-quality variables
- applying limited within-segment forward filling
- engineering time-based and rolling-window features
- visualizing sensor behavior, missingness, and trends

Model development and evaluation were completed by my teammates.

## Dataset
The project uses IoT sensor data from an experimental autonomous hydroponics platform. The sensors recorded readings approximately **every 5 minutes**, producing a multivariate time series that included:
- water temperature
- pH
- electrical conductivity (EC)
- dissolved oxygen (DO)

These variables are operationally meaningful in hydroponics:
- **pH** affects nutrient availability and uptake
- **EC** serves as a proxy for dissolved nutrient concentration
- **DO** reflects oxygen availability in the root zone and overall water health
- **temperature** influences plant stress, oxygen solubility, and nutrient dynamics

## EDA and Cleaning Workflow

### 1. Timestamp parsing
Timestamps were parsed as UTC-aware datetimes, sorted chronologically, and invalid timestamps were removed.

### 2. Numeric coercion
Sensor columns were converted to numeric values, with non-numeric entries coerced to missing values so they could be handled systematically.

### 3. Gap detection and segmentation
Because the sensors were expected to report about **once every 5 minutes**, large gaps were treated as operational outages rather than ordinary missingness. To avoid leaking information across disconnected periods, the time series was split into continuous segments whenever the gap between readings exceeded **10 minutes**.

This mattered in practice because the Sensor 1 pH series contains a very large break of roughly **37 days**. At a 5-minute sampling frequency, that corresponds to about **10,656 expected readings** with no continuity. Treating the data before and after that break as one uninterrupted sequence would have been analytically sloppy, so segmentation was used to preserve the real structure of the time series.

### 4. Placeholder zero cleanup
Exact zeros in **pH**, **EC**, and **dissolved oxygen** were treated as invalid sensor readings and converted to missing values rather than accepted as true measurements.

This was not an arbitrary cleanup choice. It was based on domain logic:

- **pH = 0** would indicate an extremely acidic solution that is far outside the range of a functioning hydroponic nutrient reservoir and would be biologically and operationally implausible under normal system conditions.
- **EC = 0** would imply essentially no dissolved ions in the water. In an active hydroponic system containing nutrient solution, that is not realistic unless the sensor failed, disconnected, or returned a placeholder value.
- **DO = 0** would indicate completely anoxic water. While severe oxygen depletion can occur in some water systems, repeated exact zeros embedded among otherwise normal readings are much more consistent with sensor error, communication failure, or bad logging than with true root-zone conditions in a monitored hydroponic setup.

Because these exact zeros were scientifically inconsistent with the rest of the operating data and the physical meaning of the variables, they were treated as **false readings** rather than legitimate observations.

### 5. Missing-value handling
Short missing stretches were forward-filled only within the same sensor and continuous segment, with a limited fill window. This avoided carrying values across long outages or across breaks where the physical system may have changed.

### 6. Feature engineering
I added time-based features and 1-hour rolling mean features to support downstream modeling. These features were intended to preserve local trends and provide model-ready summaries of recent sensor behavior without smoothing across major outages.

## EDA Highlights

### pH Distribution by Sensor
This histogram compares the distribution of pH readings across the two sensors. It helps show typical operating ranges, differences in sensor behavior, and the presence of unusual values that informed later cleaning decisions.

![pH Distribution by Sensor](figures/ph-distribution-by-sensor.png)

### Sensor 1 pH Over Time
This time-series plot shows how pH readings from Sensor 1 changed over time, including gradual drift, instability, extreme low outliers, and a major multi-week outage. Because readings were expected every 5 minutes, the large break visible in the series represents a substantial loss of continuity rather than a minor missing-data issue.

![Sensor 1 pH Over Time](figures/sensor1-ph-over-time.png)

## Tools Used
Python, pandas, matplotlib, seaborn, Jupyter Notebook

## Repository Structure
```text
.
├── notebooks/
│   └── eda.ipynb
├── figures/
│   ├── ph-distribution-by-sensor.png
│   └── sensor1-ph-over-time.png
├── requirements.txt
└── README.md
