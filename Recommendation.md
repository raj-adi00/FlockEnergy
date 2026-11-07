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