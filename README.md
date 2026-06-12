# BCG Data Science Virtual Experience – Project

> **Disclaimer:** This project was completed as part of the BCG Forage Virtual Experience Program. It does not represent employment, work experience, or an internship with BCG.

## Repository Structure

```
repo/
├── Data/                                          # Raw datasets provided for the project
├── Outputs/                                       # Initial email and final deliverables
│   ├── 00_initial_email_my_solution.md            # My initial client email (business framing, data needs, plan)
│   ├── 00_initial_email_forage_solution.md        # Forage's reference email
│   ├── 04_executive_summary_my_solution.pdf       # My final executive summary
│   ├── 04_executive_summary_forage_solution.pdf   # Forage's reference executive summary
│   └── out_of_sample_data_with_predictions.csv    # Final model predictions
├── 01_EDA_my_solution.ipynb                       # My exploratory data analysis
├── 01_EDA_forage_solution.ipynb                   # Forage's reference EDA
├── 02_feature_engineering_my_solution.ipynb       # My feature engineering work
├── 02_feature_engineering_forage_solution.ipynb   # Forage's reference solution
├── 03_modeling_my_solution.ipynb                  # My modeling notebook - Random Forest
├── 03_modeling_forage_solution.ipynb              # Forage's reference modeling solution
└── README.md
```

## How to Navigate This Repo

This project is built around a side-by-side comparison: for every step, I completed my own solution first, then compared it against Forage's reference solution. The differences between the two are the foundation of the **Key Takeaways** and **Extended Notes** sections below.

Each task is numbered in the order it was completed (00–04):
- `*_my_solution` – my independent attempt
- `*_forage_solution` – Forage's reference solution for that step

The `Outputs/` folder contains the initial client email (step 0) and the final executive summary and predictions (step 4).

---
## Key Takeaways

Quick summary of what I learned at each step.

**Step 0 – Initial Client Email**
- Lead with a clear, bulleted plan. Conversational writing buries the structure clients need.
- Always explicitly request the target variable (churn) up front.

**Step 1 – EDA**
- Examine every variable, not just the ones tied to the main question. This is how you catch issues like skew early.
- Skewness shows up often and should be tracked for treatment during feature engineering.

**Step 2 – Feature Engineering**
- Think in two dimensions: across time (seasonal/year-over-year) *and* across categories within the same period (e.g., peak vs. off-peak pricing).
- Feature engineering isn't done until cleanup is done: log-transform skewed columns, encode categoricals, drop raw date columns, run a correlation check.
- Validate new features against the target (`groupby('feature').agg({'churn':'mean'})`) before keeping them.

**Step 3 – Modeling (Random Forest)**
- Always establish a baseline model before tuning.
- The deliverable is the predictions, not the model. Export results to a usable file.
- Close the loop: tie model results back to the original business hypothesis.
- Going beyond the reference solution (e.g., SMOTE, threshold tuning) is good practice even with modest gains.

**Executive Summary**
- Lead with the business implication, not the raw finding ("discounts may not reduce churn" beats "price sensitivity is weak").
- Scale percentages with real numbers: "10% churn" lands differently as "~1,500 customers."
- For executive audiences, less is more: one headline conclusion per point.

---

<details>
<summary><strong> Extended Notes (click to expand)</strong> — full self-review and comparisons against the Forage reference solutions</summary>

### Step 0: Initial Client Email — Extended Notes

This email is much more concise than mine, partly because I had less context on the audience. I like the example's use of a clearly defined, bulleted plan — mine was more conversational and didn't get to the point as clearly. One thing I'd keep from my version is the extra detail in the data request section, broken into specific categories, which could reduce back-and-forth with the client later.

One important miss: I never asked for the churn variable itself (whether a customer had churned), which is the target variable that all predictions are based on. Important to remember going forward.

### Step 1: EDA — Extended Notes

**EDA basics covered:**
- Price sensitivity is the degree to which demand responds to price changes, often quantified via price elasticity of demand (% change in quantity demanded / % change in price). A value with absolute value greater than 1 suggests an elastic product.

**Topics covered in the EDA:**
- **Churn** — overall churn rate visualized via stacked bar chart.
- **Sales channels** — churn rate by `channel_sales`, including one channel with a "MISSING" value.
- **Consumption** — electricity/gas consumption across time periods, visualized with histograms; strong right skew identified, addressed later via boxplots and feature engineering.
- **Forecasts** — forecasted electricity, discount, meter rental, and energy values by period, also right-skewed.
- **Contract type** — roughly even split between types, suggesting it's not a strong churn predictor.
- **Margins** — boxplots revealed significant outliers to address later.
- **Subscribed power** and other secondary columns were also reviewed for completeness.

**Takeaway:** I initially focused only on churn-related variables, but realized the first EDA pass should cover *every* variable — this is how you catch things like skew that matter for later steps. TL;DR: look at everything; it uncovers issues you didn't know to look for.

### Step 2: Feature Engineering — Extended Notes

**What I got right:**
- **Dec–Jan pricing features:** I extended the reference solution's December vs. January off-peak price comparison to also cover peak and mid-peak periods, building `peak_diff_dec_january_energy/power` and `mid_peak_diff_dec_january_energy/power` using the same groupby approach. The reference instead computed average price *differences between* pricing periods (off-peak vs. peak, peak vs. mid-peak, etc.), both as an overall mean and a per-month maximum — capturing pricing-plan structure rather than seasonal movement. Both angles are valid; the reference's approach is arguably richer.
- **Contract duration:** I computed `contract_duration` as days between `date_end` and `date_activ`, conceptually matching the reference's `tenure` feature (in years). The reference also ran a churn-rate-by-tenure check to validate the feature before using it — something I skipped.
- **Days until renewal:** I created `days_until_renewal` using the same 2016-01-01 reference date as the model answer. The reference went further, converting *all four* date columns into "months from reference" features (`months_to_end`, `months_activ`, `months_modif_prod`, `months_renewal`) via a reusable `convert_months()` helper, then dropped the original datetime columns — I only did one of these.
- **Other independent features:** I created `cons_12m_diff` (actual vs. forecast consumption) and `power_cost` (gross minus net margin), both reasonable ideas not present in the reference solution.

**What I missed:**
- **Inter-period price differences** (mean and max, off-peak/peak/mid-peak pairwise) — the reference's most impactful pricing features, generating 12 new columns. These likely matter because price-sensitive customers react to *within-month* price spread.
- **Converting all date columns to month-based features** and dropping the originals — I only handled one of four date columns and left raw datetimes in the dataset, which can break a model.
- **Encoding categorical/boolean columns** — the reference converted `has_gas` to binary and one-hot encoded `channel_sales`/`origin_up` (dropping rare categories below 100 occurrences). I left these untouched.
- **Log-transforming skewed numeric columns** via `np.log10(x + 1)` — I identified skew during EDA but never acted on it during feature engineering.
- **Correlation analysis** — the reference finished with a correlation heatmap and dropped two highly correlated features (`num_years_antig`, `forecast_cons_year`). I skipped this step entirely.

**Major takeaways:**
- Think in multiple dimensions when engineering features: across time *and* across categories within a time window.
- Feature engineering should end with cleanup (transforms, encoding, dropping raw columns, correlation check) — not just creation.
- Validate every new feature against the target variable before keeping it.
- Build reusable helper functions for repeated logic instead of copy-pasting.
- Before finalizing: check dtypes, confirm no raw datetimes or object/string columns remain, and check for near-zero-variance features.

### Step 3: Modeling — Extended Notes

**What I got right:**
- Correct 75/25 train/test split with `random_state=42`, matching the reference; dropped `id` and separated the `churn` target correctly.
- Identified that high accuracy was misleading due to class imbalance and focused on recall for churners — the reference notes this issue but doesn't attempt to fix it.
- Tried three strategies to improve recall: `class_weight='balanced'`, SMOTE, and threshold adjustment — going further than the reference.
- Correctly used `imblearn.Pipeline` to apply SMOTE inside cross-validation folds, avoiding data leakage.
- Extracted and interpreted feature importances, correctly reading the spread-out importances as a sign of weak overall signal rather than a feature problem.
- Made a clear, documented final model choice (original model at threshold 0.15 over the SMOTE pipeline) with a comparison table and plain-English explanations.

**What I missed:**
- **No baseline model** — the reference starts with a plain `RandomForestClassifier(n_estimators=1000)` before any tuning, to establish a benchmark. I jumped straight to grid search.
- **Unpacking the confusion matrix** — the reference explicitly prints and labels true/false positives/negatives individually:
```python
  tn, fp, fn, tp = metrics.confusion_matrix(y_test, predictions).ravel()
  print(f"True positives: {tp}")
  print(f"False positives: {fp}")
  print(f"True negatives: {tn}")
  print(f"False negatives: {fn}")
```
  This is far more readable than a raw matrix and forces interpretation in business terms.
- **Full feature importance bar chart** — the reference plots *all* features (not just the top 15) in a horizontal bar chart for a complete picture:
```python
  feature_importances = pd.DataFrame({
      'features': X_train.columns,
      'importance': model.feature_importances_
  }).sort_values(by='importance', ascending=True).reset_index()
  plt.figure(figsize=(15, 25))
  plt.barh(range(len(feature_importances)), feature_importances['importance'], color='b', align='center')
  plt.yticks(range(len(feature_importances)), feature_importances['features'])
  plt.show()
```
- **Exporting predictions to CSV** — the reference attaches predictions and probabilities to the test set and saves them:
```python
  X_test['churn'] = predictions.tolist()
  X_test['churn_probability'] = probabilities.tolist()
  X_test.to_csv('out_of_sample_data_with_predictions.csv')
```
  The model itself isn't the deliverable — the predictions are.
- **Closing the loop on the original hypothesis** — the reference explicitly revisits "is churn driven by price sensitivity?" and answers it using feature importances (price is a weak but present contributor). I noted price features appeared but didn't frame it as answering the original question.

**Major takeaways:**
- Always establish a baseline before optimizing.
- Unpack and label the confusion matrix — translate each value into a business cost.
- The deliverable is the predictions file, not the model itself.
- Always close the loop back to the original business hypothesis.
- Exploring beyond the reference solution (e.g., recall-improvement techniques) is valuable even with modest gains — the systematic comparison process is what matters.

### Executive Summary — Extended Notes

**What I got right:**
- Both my summary and the reference lead with the same core finding: price sensitivity is not the primary churn driver — net margin and consumption are.
- My churn rate stat (~10% of SME customers annually) matched the reference's "nearly 10% across ~15k customers."
- My recommendation to run a controlled discount experiment mapped directly to the reference's suggestion that discounts may not be the best retention strategy and more customer outreach is needed.

**What I missed:**
- **Framing:** the reference leads with the business implication ("discounts may not be the best strategy to reduce churn") rather than the raw finding ("price sensitivity is weak"). Recommendations land harder than findings for a steering committee.
- **Naming the methodology:** the reference explicitly calls out that a sensitivity analysis was conducted — naming the method adds credibility without requiring technical depth.
- **Scaling numbers:** the reference includes the total customer count (~15k), which reframes "10% churn" as "~1,500 customers" — a much more concrete business problem.
- **Layout philosophy:** my slide was a detailed report; the reference was a true executive summary — one headline conclusion with one supporting bullet per point, optimized for fast "so what" reading.

**Major takeaways:**
- Lead with the implication, not the finding.
- Always scale percentages with real numbers/denominators.
- For executive audiences, less is more — every line should be a conclusion, not a description.

</details>
