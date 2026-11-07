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
