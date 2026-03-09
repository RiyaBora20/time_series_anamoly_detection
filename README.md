##  Project Overview

This project implements a proactive monitoring system designed to forecast system incidents before they occur. Unlike reactive threshold alerts, this model uses a **multivariate sliding-window approach**,  to analyze system telemetry and predict whether a "System Incident" will occur within a future time horizon ($H$).

The solution is built using **XGBoost**, with a specialized evaluation strategy that balances the need to catch critical failures (Recall) with the necessity of minimizing false alarms (Precision).

---

## Methodology

### 1. Data Synthesis & Labeling

To ensure a rigorous "ground truth" for the  task, a handcrafted dataset was generated simulating 10 distinct infrastructure metrics.

* **Complex Derivation:** Anomalies are not simple threshold breaches. An "Incident" ($y=1$) is triggered only when multiple non-linear conditions intersect:
* **Condition A:** High Resource Overload ($Metric_0$ & $Metric_1$ are high).
* **Condition B:** Rapid State Divergence ($Metric_4$ drops while $Metric_9$ spikes).


* **Temporal Persistence:** Incidents are modeled with a duration (rolling window) to simulate real-world system recovery times.

### 2. Problem Formulation: The Sliding Window

To transform time-series data into a supervised learning task for XGBoost:

* **Lookback ($W=20$):** The model "sees" the previous 20 timestamps of all 10 metrics.
* **Lookahead ($H=10$):** The target label is positive if an incident occurs *anywhere* within the next 10 steps.
* **Flattening:** Each training sample is a flattened vector of $W \times 10$ features, enriched with window statistics (rolling mean, std, and max).

### 3. Feature Engineering

* **Raw Lags:** Captures the immediate sequence of events.
* **Statistical Aggregates:** Provides context on system volatility and "level" shifts, helping the model distinguish noise from trend.
* **Temporal Split:** Data is split chronologically (80/20) to prevent data leakage and ensure the model is tested on "the future."

---

##  Model Evaluation

The model was evaluated using a **Precision-Recall (PR) Curve** rather than a standard ROC curve, due to the inherent class imbalance (2.78% incident rate).


That is a fantastic set of results to present for your interview. You have moved from a model that was essentially "silent" (10% recall) to a highly effective early-warning system (81% recall) while maintaining an impressive **0.91 precision**.

Here is a structured analysis of your results, formatted as a "Discussion of Results" section for your final report or README.

---

##  Analysis of Results

### 1. Modeling Choices & Design Philosophy

The core challenge was to detect a **non-linear, state-based incident** hidden within a noisy, imbalanced dataset.

* **Model Selection:** XGBoost was chosen for its ability to learn complex feature interactions (e.g., *Metric A* rising while *Metric B* falling) that simple linear models or single-threshold alerts would miss.
* **Feature Engineering:** By enriching the 200 raw temporal features with 30 window statistics (mean, std, max), we provided the model with "contextual awareness." This allowed the gradient booster to distinguish between momentary spikes (noise) and sustained system divergence (incident signature).

### 2. Evaluation Strategy: Prioritizing Actionability

The initial evaluation used a standard **0.50 threshold**, which yielded a perfect precision (1.00) but a disastrous **recall of 0.10**. In a production environment, this represents a system that is "too quiet"—it only alerts when a total collapse is already occurring.

**The Threshold Shift:**
By utilizing the **Precision-Recall Curve**, we identified that the optimal decision boundary for our infrastructure was **0.2457**.

* **Target:** 80% Recall.
* **Result:** 81% Recall with 0.91 Precision.

This shift transformed the model from a lagging indicator into a **leading indicator**. We are now catching 8 out of 10 incidents before they fully manifest, with only a 9% false-alarm rate.

### 3. Metric Significance & Interpretation

The feature importance graph validates the "Ground Truth" of our handcrafted dataset.

* **Primary Drivers:** `metric_9` and `metric_1` showed the highest information gain. This confirms the model prioritized the "System Overload" and "Divergence" signals we injected.
* **Support Drivers:** `metric_4` and `metric_0` acted as secondary indicators.
* **Temporal Logic:** The model didn't just look at the most recent timestamp; it weighted the **rolling statistics** heavily, proving that the *trend* of the metrics over the 20-step window was more predictive than any single point in time.

### 4. Conclusion & Real-World Impact

The final model achieves an **F1-score of 0.86** on the incident class, which is exceptionally high for a dataset with only ~3% anomaly density.


### Key Results:

* **Recall Targeted at 80%:** By optimizing the decision threshold using the PR-curve, the model successfully identifies 80% of all future incidents.
* **Precision Optimization:** Once the 80% recall requirement was met, the threshold was calibrated to maximize Precision. This ensures that "Alert Noise" is minimized, protecting engineering teams from alert fatigue while still maintaining a robust safety net.

### Feature Importance:

Analysis reveals that the model correctly identified $Metric_0, Metric_1, Metric_4, \text{and } Metric_9$ as the primary drivers of failure, validating the internal logic of the XGBoost ensemble.

---

### How to Run

1. Clone the repository.
2. Install dependencies: `pip install xgboost pandas numpy matplotlib seaborn scikit-learn`.
3. Run `anomaly.ipynb` to view the full pipeline and visualizations.
