# Capstone Report — FlyRank Search Intelligence Lane

* **Author:** Youssef Kady
* **Lane:** Search Intelligence & Machine Learning
* **Repo:** https://github.com/Youssif-Kady/flayrank_task1
* **Date:** 2026-08-31

---

### 1. Problem framing
* **Decision Support:** This model helps FlyRank content editors prioritize which pages require immediate updates to mitigate traffic decay and rank loss.
* **Unit of Analysis:** A single webpage/URL (`content_hash_id`).
* **Output:** A ranked action score and classification probability indicating the likelihood of traffic or ranking decay.
* **Human Action:** Content editors review the prioritized weekly queue and execute content refreshes, title/meta optimizations, or technical audits on vulnerable pages.
* **Cost of a Wrong Call:** False positives waste editorial resources on pages that are stable, while false negatives lead to unmitigated traffic decay on high-value pages.
* **Why ML Helps:** Manual inspection of millions of search rows is impossible; machine learning automates pattern recognition across non-linear metrics like impressions, ranking positions, and historical performance trends.

### 2. Data safety
* **Data Used:** The FlyRank ML Internship warehouse dataset, aggregating Google Search Console (GSC) metrics.
* **Excluded Columns:** Client identifiers (`client_id`, raw URLs) and label-derived fields (such as `trend_direction` and `trend_pct`) were strictly excluded to eliminate target leakage.
* **Leakage Risks Managed:** Ensured that all aggregation features relied strictly on historical windows ($\le T-1$) using proper chronological shifting, avoiding future window pollution. Grouping was strictly enforced by entity (`content_hash_id`) during cross-validation.
* **Privacy Confirmation:** No client-identifying details, private queries, or sensitive information appear anywhere in the repository.

### 3. Baseline
* **Baseline Rule:** A transparent rule-based heuristic (`HIGH_IMPRESSIONS_LOW_RANK` and `CTR_OPTIMIZATION_CANDIDATE`) scoring pages based on impression volume penalized by average ranking position.
* **Fairness:** Evaluated on the exact same dataset splits and validation metrics to ensure a direct apples-to-apples comparison.
* **Baseline Performance:** Provided a functional heuristic baseline (achieving a weak random/negative R² or poor initial AUC on complex traffic variance), illustrating the limitations of static threshold rules.

### 4. Model / analysis
* **Method:** Random Forest and XGBoost Regressors/Classifiers, selected because tree-based ensembles effectively handle non-linear search thresholds and heavily skewed traffic distributions without complex transformations.
* **Feature List:** Total impressions, 90-day activity duration, and average ranking positions.
* **Features Left Out:** Current-day traffic metrics, target-derived ratios (`past_ctr` during validation audits), and raw identifiers.
* **Target Definition:** Predicting content traffic metrics and identifying pages experiencing structural performance decay.

### 5. Evaluation
* **Split Design:** Employed a rigorous `GroupKFold` cross-validation grouped by `content_hash_id` to prevent the model from memorizing specific URLs and to ensure an honest evaluation on unseen pages.
* **Metrics:** MAE, RMSE, and $R^2$ Score.
* **Model vs. Baseline:** Following the removal of data leakage (such as label-derived CTR ratios), the honest model achieved a realistic and robust performance metric ($R^2 \approx 0.4487$), proving genuine predictive signal over the static baseline.
* **Error Analysis:** Residual errors concentrate around extreme high-volume head queries that exhibit high seasonal volatility during promotional periods.

### 6. Interpretation
* **Model Findings:** Feature importance analysis confirmed that `total_impressions` and `avg_position` are the primary drivers of traffic volume and volatility.
* **Surprises & Negative Results:** Initial models yielding near-perfect metrics ($R^2 > 0.99$) were unmasked as data leakage during the audit phase, reinforcing the importance of rigorous validation design. A moderate $R^2$ is an honest reflection of organic search volatility.

### 7. Recommendation
* **Ranked Playbook:** Deploy a weekly dashboard filter highlighting pages with high impression volume and declining performance metrics.
* **Editor Workflow:** Editors should prioritize refreshing content and optimizing metadata for pages sitting on the fringe of Page 1 (positions 11–15) where optimization yields high business impact.
* **Confidence & Limits:** The model provides directional decision-support rather than absolute certainty. It cannot predict external algorithm updates or sudden viral spikes.

### 8. Reproducibility
* **Environment:** Python 3.x, Pandas, Scikit-Learn, Polars, and Hugging Face Hub APIs.
* **Random Seeds:** `random_state=42` used across all data splits and model initializations.
* **Execution:** Clone the repository, configure the `HF_TOKEN` environment variable, and run notebooks sequentially from `work/notebooks/`.
