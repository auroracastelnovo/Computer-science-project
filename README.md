# 🚕 NYC Yellow Taxi Data Analysis — Foundations of Computer Science Final Project

![Python](https://img.shields.io/badge/Python-3.8%2B-blue.svg)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Analysis-150458.svg)
![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626.svg)
![Status](https://img.shields.io/badge/Status-Completed-success.svg)

Final project for the **Foundations of Computer Science** course (Master's Degree). The project consists of in-depth data analysis, data cleaning, and the implementation of a **Trip Chaining** algorithm on the **NYC Yellow Taxi (TLC)** dataset.

---

## 📌 Project Overview

This project analyzes a dataset of New York City taxi trips (`data.csv` provided by the NYC Taxi and Limousine Commission - TLC). The objective is to address 17 specific analytical tasks covering data ingestion, data cleaning, aggregation, statistical and temporal analysis, and the implementation of an advanced algorithm based on `pd.merge_asof` and connected components (*Path Compression*) to reconstruct chains of consecutive trips completed by the same taxis.

---

## 🛠️ Tech Stack

* **Python 3.x**
* **Pandas**: Data manipulation, aggregation, cleaning, datetime parsing, and asynchronous merges (`pd.merge_asof`).
* **NumPy**: Vectorized operations and numerical computations.
* **Jupyter Notebook**: Interactive development environment and data presentation.

---

## 📊 Analysis Workflow & Notebook Structure

### 1. Data Ingestion & Cleaning (Preprocessing)
* **Optimized Data Types**: Used nullable integer types (`Int64`) for columns like `VendorID`, `RatecodeID`, `passenger_count`, and `payment_type` to properly handle missing values (`NaN`).
* **Datetime Parsing**: Automatic conversion of timestamp columns (`tpep_pickup_datetime`, `tpep_dropoff_datetime`).
* **Missing Value Imputation**: Imputed missing `store_and_fwd_flag` values with `'Undefined'`.
* **Temporal Anomaly Removal**: Filtered out invalid trips where pickup time occurred after dropoff time.

---

### 2. Milestone Tasks (1 - 17)

| # | Task / Milestone | Methodological Approach |
|---|---|---|
| **1** | **Long Trip Filtering** | Extracted all trips with `trip_distance > 50` and removed extreme outliers (e.g., `trip_distance > 210,000`). |
| **2** | **Missing Payments** | Extracted trips where `payment_type` was missing (`isnull()`). |
| **3** | **Pickup-Dropoff Pair Counts** | Grouped by `(PULocationID, DOLocationID)` pairs and computed trip counts using `.size()`. |
| **4** | **Invalid Data Handling (`bad` dataframe)** | Isolated rows containing missing values (`NaN`) across critical columns into a dataframe named `bad`, and removed them from the main `nyc` dataset. |
| **5** | **Trip Duration Calculation** | Created a `duration` column in minutes calculated as `(dropoff - pickup).dt.total_seconds() / 60`. |
| **6** | **Trips per Pickup Location** | Calculated the total number of trips originating from each `PULocationID`. |
| **7** | **30-Minute Interval Clustering** | Divided the day into 48 30-minute intervals (`cluster = (hour * 60 + minute + second/60) // 30`). |
| **8** | **Average Passengers & Fare per Interval** | Calculated the mean (`mean()`) of `passenger_count` and `fare_amount` for each 30-minute time cluster. |
| **9** | **Average Fare by Payment Type & Interval** | Grouped by `(payment_type, cluster)` and computed the average `fare_amount`. |
| **-** | *Refund & Cancellation Handling* | Identified trips with negative monetary values, converted them to absolute values to find matching duplicate pairs (cancellations), and removed both negative and positive pairs (`nyc_new`). |
| **10** | **Peak Average Fare per Payment Type** | Determined the 30-minute interval with the maximum average fare for each payment type (`idxmax()`). |
| **11** | **Peak Tip-to-Fare Ratio** | Computed the ratio of total tips to total fares (`tip_sum / fare_sum`) per payment type and cluster, identifying the peak interval. |
| **12** | **Highest Average Fare Locations** | Identified the `PULocationID` and `DOLocationID` associated with the highest average fare amounts. |
| **13** | **Top 5 Destinations Dataframe (`common`)** | Filtered the dataset to retain only the top 5 most common dropoff destinations for each pickup location. |
| **14** | **Average Fare on `common` Dataframe** | Recalculated average fares by `(payment_type, cluster)` strictly on the top 5 destination routes. |
| **15** | **Fare Difference Calculation** | Computed the absolute difference between average fares in `common` and the general dataset (`diff = fare_common - fare_nyc`). |
| **16** | **Relative Fare Variation Ratio** | Calculated the relative percentage/proportional change in fare amounts (`diff / fare_nyc`). |
| **17** | **Trip Chaining Algorithm** | Reconstructed sequential trip chains for each vehicle (`VendorID`) where trip $N+1$ picked up passengers at the exact dropoff location of trip $N$ within **2 minutes**. Used `pd.merge_asof`, duplicate filtering, and **Path Compression** graph traversal. |

---

## 🔬 In-Depth: Trip Chaining Algorithm (Task 17)

The trip chaining algorithm solves a complex sequential tracking problem on large-scale trip data:
1. **Temporal As-Of Merge**: Leverages `pd.merge_asof` with `tolerance=pd.Timedelta('2 minutes')` and `direction='forward'`, pairing dropoff times of trip $N$ with pickup times of trip $N+1$ matching on `VendorID` and `location`.
2. **Disambiguazione**: Keeps only the earliest subsequent dropoff event to avoid circular references or multiple branching.
3. **Graph Union-Find & Path Compression**: Initializes each trip with a unique chain ID (its index), iteratively updates parent relationships, and traces to the root ancestor (*ultimate root*) using path compression to assign all connected trips to a single unified `chain` ID.

---

## 🚀 How to Run the Notebook

### Prerequisites
Ensure Python 3.8+ and the following packages are installed:
```bash
pip install pandas numpy jupyter
```

### Setup & Execution
1. Clone the repository or download the project files:
   ```bash
   git clone https://github.com/your-username/NYC-Taxi-Data-Analysis.git
   cd NYC-Taxi-Data-Analysis
   ```
2. Place the `data.csv` dataset in the project directory or update the file path in the notebook:
   ```python
   nyc = pd.read_csv('data.csv', low_memory=False, dtype=..., parse_dates=...)
   ```
3. Launch Jupyter Notebook:
   ```bash
   jupyter notebook "Final project AFN.ipynb"
   ```
4. Run all cells sequentially.

---

## 📄 License

This project was developed for academic purposes for the **Foundations of Computer Science** course. All data originates from the publicly available **NYC Taxi & Limousine Commission (TLC)** trip record dataset.
