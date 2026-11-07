## Task 1

### 1. Raw-Data Coverage and Quality Findings
The raw REFIT dataset for House 1 contained significant irregularities in sampling frequency, data entry, and sensor reliability. 
* **Initial Volume:** 8,518,763 raw readings across 12 columns.
* **Duplication:** The raw data contained duplication, with one check revealing 595,940 duplicated rows (6.99%). 
* **Missing Data:** Significant gaps existed prior to any processing. Appliance 9 was having null value for 22.44% of the raw timeline (1,911,811 missing values), while the main Aggregate meter missed roughly 2.2% of its readings.
* **Extreme Anomalies:** Sensor glitches produced physically impossible power readings. The `Aggregate` channel recorded spikes up to 23,965W. Individual appliances, showed absurd maximums: Appliance 1 hit 65,549W, and Appliance 4 hit 98,301W.

### 2. Cleaning Steps
* **De-duplication:** All duplicated timestamps were removed by keeping the first recorded occurrence, collapsing the dataset to uniquely timestamped events.
* **Temporal Regularization:** The chaotic, irregularly spaced sensor readings (often 8 to 14 seconds apart) were resampled to a strict 1-minute frequency using `.resample('1min').mean()`. The resulting cleaned dataset contains exactly 920,031 rows representing a continuous minute-by-minute timeline.

### 3. Decisions About Gaps and Outliers
* **Physics-Based Outlier Removal vs. Statistical:** Standard statistical outlier detection (IQR) falsely flagged 10.93% of `Aggregate` data and 21.73% of `Appliance1` data as "outliers" simply because appliances spend most of the day at 0W. Values were not erased because large number of outliers is possible in this case where appliances operates at 0 or max power. So we only flagged the outliers but not removed.
* **Short Gap Interpolation:** Temporary sensor dropouts (3 minutes or less) were bridged using forward-fill (`ffill`), as appliance power states typically remain static over short durations.
* **Long Outage Flagging:** Gap sizes were calculated independently for every column. If an appliance or aggregate meter dropped offline for more than 3 consecutive minutes, those rows were deliberately left as `NaN` (null). A boolean `Long_Outage_Flag` column was added to explicitly track these system failures.

### 4. Remaining Concerns and Final Dataset State
* **Intentional Missing Values:** The cleaned dataset retains a high volume of null values by design. Appliance 9 remains at 23.04% null (211,988 minutes), and the Aggregate meter is at 12.43% null (114,435 minutes). These are not pipeline errors; they represent true, prolonged sensor outages where injecting synthetic data would corrupt downstream machine learning models.
* **Flagged Minutes:** A total of 215,387 minutes are flagged as having at least one long outage across the household channels. While there were 0 instances of a complete whole-house blackout (where all 10 columns failed simultaneously), partial dropouts were extremely common.

## Task 2
### 1. Raw-Data Coverage and Quality Findings
Mechanical and Behavioral Anomalies: Expanding the analysis to multiple households for cycle detection revealed extreme operational outliers. House 5 exhibited washing machine cycles lasting up to ~3,000 minutes (approximately 48 hours) and drawing nearly 30 kWh.

### 2. Cleaning Steps
Cycle Extraction Filtering: While Task 1 focused on data cleaning, Task 2 filtering utilized an empirical 20-minute minimum duration threshold to classify and remove 1-to-10-minute active spikes as hardware noise rather than true mechanical wash cycles.

### 3. Decisions About Gaps and Outliers
Appliance-Level Outliers: The massive 48-hour cycle durations in House 5 were retained. While physically impossible for normal intentional laundry usage (representing a stuck relay, continuous pump failure, or unresolved error state), they represent true sensor captures of a failing appliance and are valid data points for hardware monitoring.

Cycle-Detection Hard Breaks: Missing data (NaN) was deliberately treated as a hard break for the state machine. Cycles are strictly prohibited from bridging across missing data intervals to prevent falsely gluing separate laundry loads together over a sensor blackout.

### 4. Remaining Concerns and Final Dataset State
Appliance Faults vs. Intentional Usage: The extreme cycle outliers in House 5 highlight that smart meter data captures both human routines and failing hardware. Downstream predictive models must account for the fact that high-energy appliance signatures do not always represent intentional human behavior, which introduces variance into behavioral forecasting models.
