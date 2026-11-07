# REFIT Smart Home Energy Analysis & Event Detection

## Project Overview (What Was Built)

This project is a reproducible data science pipeline built to process, analyze, and model high-frequency household energy consumption data from the REFIT dataset. The objective is to ingest raw, irregular smart meter readings, clean and regularize the timeline, perform exploratory data analysis (EDA) on specific appliances (washing machines), and develop a machine learning model to detect appliance activation events.

## Repository Structure

`README.md` : High-level project overview, setup instructions, and preprocessing assumptions.
`DATA.md` : Detailed documentation of data quality, missing gaps, anomaly handling, and cleaning decisions.
`RESULTS.md` : Exploratory analysis findings, visualizations, and machine learning model evaluation.
`code` : Source code for each task

## Set up and Run instructions

#### Initialize the Virtual Environment:
Open your Git Bash terminal in the project root and run:
```
python -m venv .venv
source .venv/Scripts/activate
```

#### Install Dependencies
Install the required libraries for data manipulation, visualization, and modeling:
```
pip install -r requirements.txt
```

#### Execution
1. Place the Raw REFIT CSV file as ``raw_dataset/`` in root directory.
2. Place the Raw REFIT CSV file as ``cleaned_dataset/`` in root directory.
3. Run the Jupyter Notebook to execute the pipeline sequentially. 

## Assumptions

* A washing machine is considered "ON" and operating if its power draw strictly exceeds a 50W threshold.

* Households with multiple washing machines (e.g., House 4) were excluded to maintain a consistent target variable mapping across the evaluation loop.

* Background noise from identical heating elements (like kettles and dishwashers) is the primary source of classification error.

## Preprocessing Decisions

* Time-Series Regularization: Data was resampled to a strict 1-minute frequency to handle missing readings and synchronize the aggregate signal with the appliance signal.

* Feature Engineering: Implemented rolling means and lagged features to capture the temporal shape of power spikes rather than just instantaneous wattage.

* Target Isolation: Dropped all non-essential appliance columns to optimize memory usage during training.

## Validation Design

Chronological Split (80/20): Standard random cross-validation was rejected to prevent data leakage. The first 80% of the timeline was used for training and the final 20% for testing. This strictly enforces the arrow of time, forcing the model to predict completely unseen future appliance usage based solely on historical patterns.

## Trade offs

* Inference Latency vs. Accuracy: The current model uses rolling average and leading data to predict output. If you want live results, these features must be dropped. The current system runs with a 7-minute operational lag and cannot be deployed in real-time without modification.

* F1-Score over Absolute Error (The Sparsity Paradox): Because the washing machine is off for ~95% of the year, predicting 0W minimizes Mean Absolute Error (MAE). We accepted a slightly worse MAE to optimize for the F1-Score, prioritizing actual cycle detection over mathematical safety.

## AI Tools Used
* Gemini
* Chatgpt

## Possible Improvements

* Deep Learning Integration: Implementing Seq2Point or a 1D Convolutional Neural Network (CNN) to automatically extract temporal features, removing the need for manual lag/lead engineering.

* Time-Context Features: Injecting cyclical time-of-day and day-of-week features, as washing machines are highly correlated with human scheduling (e.g., weekends).

* Multi-Target Disaggregation: Training a unified model to predict the washing machine, kettle, and microwave simultaneously, allowing the algorithm to cross-reference and eliminate coincident noise.