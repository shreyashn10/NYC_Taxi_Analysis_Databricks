# NYC Taxi Big Data Analysis and Fare Prediction

An end-to-end data engineering, analytics, and machine learning project built with Databricks, Apache Spark, Spark SQL, Delta Lake, pandas, and scikit-learn.

The project processes nearly one billion NYC yellow and green taxi trip records, applies distributed data-quality rules, creates a unified analytical table, answers business questions using Spark SQL, and evaluates regression models for trip-amount prediction.

## Project overview

| Area | Summary |
|---|---|
| Raw data volume | 991,467,464 yellow and green taxi records |
| Final validated table | 959,791,969 records |
| Processing platform | Databricks and Apache Spark |
| Storage layer | Delta Lake / Unity Catalog table |
| Analytics | Spark SQL |
| Machine learning | scikit-learn |
| Best reported model | Random Forest Regressor |
| Best reported test RMSE | 11.01 |

## Pipeline architecture

```text
NYC Yellow Taxi Parquet Data     NYC Green Taxi Parquet Data
              \                       /
               \                     /
                v                   v
                 Spark ingestion and profiling
                              |
                              v
                 Distributed data-quality rules
                 - valid timestamp ordering
                 - accepted date range
                 - duration: 1–180 minutes
                 - distance: 0.1–50 km
                 - speed: 0–100 km/h
                 - passenger count: 1–6
                              |
                              v
                   Schema standardisation
                              |
                              v
                      Union by column name
                              |
                              v
             Pickup and drop-off taxi-zone joins
                              |
                              v
                  Delta table: bde.trips_final
                         /             \
                        v               v
              Spark SQL analytics    Sampled ML dataset
                                            |
                                            v
                               scikit-learn regression
```

## Data processing workflow

### 1. Data ingestion

Yellow and green taxi trip datasets are loaded from Parquet files into Spark DataFrames. The raw datasets contain:

- Yellow taxi records: 907,982,776
- Green taxi records: 83,484,688
- Combined raw records: 991,467,464

### 2. Data-quality validation

The Spark pipeline removes records with:

- Drop-off timestamps earlier than pickup timestamps
- Pickup or drop-off timestamps outside the accepted date range
- Negative or implausibly high calculated speeds
- Trip durations below 1 minute or above 180 minutes
- Trip distances below 0.1 km or above 50 km
- Passenger counts outside the range of 1 to 6

### 3. Schema standardisation

The yellow and green taxi schemas are aligned by:

- Renaming equivalent fields to common names
- Selecting shared analytical columns
- Adding a taxi-colour identifier
- Unioning both datasets into one Spark DataFrame

### 4. Location enrichment

The taxi-zone lookup dataset is joined twice:

- Pickup location ID to pickup borough
- Drop-off location ID to drop-off borough

### 5. Delta table creation

The cleaned and enriched dataset is persisted as:

```text
bde.trips_final
```

Final row count:

```text
959,791,969
```

## Business analysis

The Spark SQL notebook covers the following analytical areas:

1. Monthly trip volume, average passengers, busiest weekday, busiest hour, and average payment metrics
2. Trip duration, distance, and speed by taxi colour
3. Trip volume, average distance, average amount, and total revenue by borough and time segment
4. Revenue share of the top pickup–drop-off borough pairs in 2024
5. Percentage of trips receiving tips
6. Percentage of trips with tips of at least USD 15
7. Average speed and distance per dollar across trip-duration bins
8. Trip-duration segments associated with driver income

## Selected findings

- Taxi demand is concentrated during daytime and evening travel periods.
- Peak activity commonly occurs around 6–7 PM.
- Yellow and green taxis have broadly comparable trip durations and distances.
- Green taxis show a slightly higher median speed in the reported analysis.
- Manhattan-to-Manhattan trips account for the largest share of 2024 taxi revenue.
- Queens-to-Manhattan and Manhattan-to-Queens are the next major revenue flows.
- Approximately 63.05% of analysed trips include a tip.

## Machine learning workflow

The modelling stage predicts `total_amount` using a time-based train/test split.

### Data split

- Training period: trips before 1 October 2024
- Test period: 1 October 2024 to 31 December 2024

### Features

Categorical features:

- Taxi colour
- Pickup borough
- Drop-off borough
- Day of week

Numeric features:

- Passenger count
- Trip distance
- Month
- Hour
- Tip amount
- Payment type

### Preprocessing

- One-hot encoding for categorical variables
- Numeric feature passthrough
- Scikit-learn pipeline-based transformation and modelling

### Models evaluated

| Model | Test RMSE |
|---|---:|
| Aggregate baseline | 26.60 |
| Ridge Regression | 12.99 |
| Random Forest Regressor | **11.01** |

The Random Forest Regressor achieved the lowest reported test RMSE.

## Technology stack

- Databricks
- Apache Spark
- PySpark
- Spark SQL
- Delta Lake
- Unity Catalog
- Python
- pandas
- NumPy
- scikit-learn

## Running the project in Databricks

### Environment

The original workflow used Databricks serverless compute with 32 GB memory.

### Execution order

Run the Databricks source notebooks in numeric order:

```text
01_data_ingestion_preparation.py
02_business_analysis.py
03_fare_prediction.py
```

Notebook 1 creates the `bde.trips_final` table used by the analytics and modelling notebooks.

### Data paths

The required source data consists of:

- Yellow taxi Parquet files
- Green taxi Parquet files
- `taxi_zone_lookup.csv`

The full raw taxi dataset is not stored in this repository because of its size.

Spark ingestion and SQL execution require access to the source datasets and an equivalent Spark environment.

## Author

**Shreyash Narayane**  
Master of Data Science and Innovation  
University of Technology Sydney

## Acknowledgements

This project uses New York City taxi trip records and the NYC taxi-zone lookup dataset. Databricks, Apache Spark, Spark SQL, Delta Lake, pandas, and scikit-learn were used for processing, analysis, and modelling.
