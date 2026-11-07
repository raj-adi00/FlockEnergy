## Washing Machine Load Disaggregation (Extra Trees)

This model utilizes an Extra Trees Regressor to disaggregate household washing machine power consumption from 1-minute resolution whole-house smart meter data. It is designed to evaluate historical energy consumption patterns by mapping total aggregate load to specific mechanical appliance cycles.

### Suitable Uses for Utility Operations

* **Historical Demand Response Planning:** The model excels at batch-processing historical smart meter data to identify peak appliance usage windows (e.g., determining what percentage of a neighborhood runs laundry during peak evening hours).
* **End-of-Month Consumer Energy Audits:** Highly suitable for generating itemized energy bills or consumer-facing insights ("Your washing machine accounted for X% of your electricity usage this month") by isolating binary ON/OFF cycles.
* **Privacy-Preserving Edge Deployment:** Because the Extra Trees architecture compiles into a fast set of decision rules, it requires very low computational overhead for inference. It can be deployed locally on residential smart meters or edge devices without transmitting high-resolution raw data to utility cloud servers.

### Operational Limits & Constraints

* **Incompatible with Real-Time Grid Monitoring:** The model relies on rolling averages and leading/lagging time-series features to accurately shape the power predictions. This introduces a mandatory 7-minute operational lag. It cannot be used for live, sub-second frequency regulation or instant load shedding.
* **The Sparsity Paradox and False Positives:** The model optimizes for cycle detection (F1-Score) over absolute mathematical error (MAE). When concurrent high-wattage appliances (like kettles and microwaves) run simultaneously, the model occasionally misclassifies this background noise as a wash cycle.
* **Peak Wattage Attenuation:** While the model successfully detects that a wash cycle is active, tree-based ensembles inherently average the predictions of multiple decision nodes. This results in "smoothing," where the model routinely under-predicts the absolute physical maximum of the appliance (e.g., predicting 650W during a physical 2,000W heating spike).
* **House-to-House Signature Variance:** The current architecture relies on a static 50W operational threshold. It may struggle to generalize across homes utilizing ultra-efficient "eco-mode" washing machines or modern appliances with high-wattage smart-standby features without household-specific recalibration.


## Task 4: Practical Implications
1. **Energy-Saving & Customer-Insight Opportunity**

The Opportunity: Time-of-Use (ToU) load shifting. By disaggregating washing machine usage, a utility company can identify households that consistently run laundry during peak evening hours (e.g., 6 PM to 9 PM). The utility can send targeted, personalized nudges via email or app notifications, offering financial incentives to shift laundry to off-peak hours (e.g., after 10 PM or mid-day on weekends).

Measurement: Success would be measured by calculating the percentage of a household's total weekly wash cycles that occur during peak hours. If a targeted household drops from 80% peak usage to 30% peak usage in the month following the intervention, the campaign is quantitatively successful.

2. **Limitations and Deployment Challenges**

Overlapping Signatures: The model struggles when appliances with similar resistive heating elements (e.g., kettles, dishwashers, electric ovens) operate simultaneously, creating aggregate power spikes that mimic washing machine cycles.

Missing Data: Smart meter data relies on continuous Wi-Fi/cellular connectivity. Brief network dropouts create missing rows. This disrupts the calculation of time-series features like rolling averages and lags, forcing the model to either skip predictions or rely on heavily interpolated, inaccurate data.

Household Differences: A static 50W ON/OFF threshold does not generalize well. A large family might use a heavy-duty washer with high baseline power, while a single occupant might use a compact, high-efficiency machine with an "eco-mode" that dips below 50W mid-cycle, causing the model to incorrectly predict that the cycle has ended.

Transferring UK Results to India: The REFIT dataset represents UK households, which rely heavily on electric space heaters and internal electric water boilers—creating a specific type of winter baseline noise. In India, the base household load is fundamentally different, dominated by heavy summer air conditioning (HVAC) cycles, ceiling fans, and refrigerators operating in higher ambient temperatures. Additionally, differences in standard appliance wattage ratings and grid voltage stability mean a model trained strictly on UK aggregate shapes will likely misclassify Indian HVAC compressor spikes as washing machines.

3. **The Impact of Lower Resolution (15 or 30-Minute Data)**
If only 15-minute or 30-minute smart meter data were available, appliance disaggregation for washing machines would become nearly impossible. The critical identifying feature of a washing machine is its initial, intense heating spike, which often lasts only 3 to 5 minutes. In a 15-minute sampling window, a 4-minute 2,000W spike is mathematically averaged alongside 11 minutes of 0W usage, flattening the recorded data point to around 530W. At a 30-minute resolution, it flattens even further, blending entirely into the background noise of the house. The loss of high-frequency temporal shapes means the model can no longer distinguish between a brief kettle boil, a dishwasher cycle, or a washing machine.