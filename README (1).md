# QuickCart Stockout Risk — Project Summary

**Deliverable:** `QuickCart_Stockout_Risk.ipynb` (fully executed; figures also in `figures/`).

## Pipeline
1. Load 5 tables with explicit `na_values` -> all locked sanity numbers pass (row counts, 65.42 / 24.01 / 10.57 target split, 19,899 missing `lead_time_days_actual`).
2. Fix data-quality issues: `city_display` casing (ST03, ST09); `'N/A'` reliability for SUP06, SUP14, SUP15 imputed with primary-category median.
3. Validated many-to-one joins -> 21,600 rows.
4. Feature engineering (37 features): reorder_gap, cover ratio / cover-minus-lead-time, lagged reorder and unmet-demand flags, velocity/stock momentum, effective demand multiplier, temporal and category features.
5. Time-based split: train Oct 1-23 (16,560), test Oct 24-30 (5,040; Imminent = 15.4%).

## Results (test set, Oct 24-30)
| Model | Accuracy | Macro-F1 | Imminent recall | Imminent precision |
|---|---|---|---|---|
| Majority baseline | 0.623 | 0.256 | 0.000 | 0.000 |
| Logistic Regression | 0.906 | 0.842 | 0.923 | 0.673 |
| Random Forest | 0.948 | 0.904 | 0.777 | 0.878 |
| Gradient Boosting | 0.955 | 0.918 | 0.845 | 0.866 |
| **Gradient Boosting, tuned (w=6)** | - | 0.910 | **0.963** | 0.771 |

Tuning: Imminent probability scaled by w; w chosen on a validation slice inside the training period (Oct 21-23), never on test.

## Honest caveats
- **Cover features nearly define the label.** Without `days_of_cover`-type features, Gradient Boosting drops to macro-F1 0.824 / Imminent recall 0.724 (At-Risk recall 0.908 -> 0.691).
- **Unseen store-SKU pairs (GroupKFold):** macro-F1 0.868, Imminent recall 0.719 — lower than the time split, so generalisation to new pairs is harder.
- **Spec discrepancy:** reliability-bucket Imminent rates match the spec for `<0.75` (15.83%) and `>=0.85` (3.82%), but the `0.75-0.85` bucket gives 3.26% here vs 6.30% in the spec. Everything else (festival 9.51% -> 23.31%, perishable 9.3% / 12.8%) matches.
- Only 30 days / one festival: festival effect can't be validated across years.

## Business takeaways
1. Festival week multiplies Imminent rate 2.45x -> pre-load Snacks/Beverages/Bakery before Oct 22.
2. Suppliers with reliability < 0.75 run ~4x the Imminent rate of reliable ones -> escalate or dual-source.
3. Perishables carry modestly higher risk.
4. Use the tuned model daily: Imminent = expedite list, At-Risk = next-day watch list.
