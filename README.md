# Demand Forecasting & Inventory Risk (Hackathon)

Weekly demand forecasting for three products (P0014, P0016, P0020), plus a dashboard that
compares the forecast with current stock and flags shortage / excess risk.

## Progress

- **Week 1:** baseline forecast (moving average) and data prep. Outputs are in `outputs/` and `processed/`.
- **Week 2: done.** Forecasting models compared against the baseline: Holt-Winters, XGBoost and an ensemble.
  The notebook is `week2_forecasting_v2.ipynb`. It reports MAE, WAPE and bias, and includes
  an agent recommendation per product with a human approval/override step.
- **Week 3: done.** Streamlit dashboard (`week3_dashboard.py`) with inventory-risk logic in `inventory_risk.py`.

## Week 2: forecasting

`week2_forecasting_v2.ipynb`

- Models: 4-week moving average (baseline), Holt-Winters, XGBoost, and an ensemble of the best pair.
- Checks whether the last week of data is incomplete, and whether Croston is needed.
- Model selection ranks on the **validation** window only. `test_clean` is used just to report the
  held-out score, so the accuracy numbers are not tuned to the data they are reported on
  (see `proposal_revision_patch.docx`).
- Saves model metrics, forecasts and the selected final forecast to `outputs/`.

`archive/week2_forecasting_v1.ipynb` is the earlier version, which selected models on the same window it reported.

> **Note:** the CSVs currently in `outputs/` were generated before the v2 fix. Re-run
> `week2_forecasting_v2.ipynb` to refresh them before relying on the numbers.

## Week 3: dashboard

`week3_dashboard.py` and `inventory_risk.py`

For each product it compares forecast demand over the planning horizon with current stock:

| Status | Rule |
| --- | --- |
| Shortage risk | stock < forecast + safety stock |
| Watch | covered, but inside the model's normal error (WAPE) |
| Balanced | within range |
| Excess risk | stock above forecast x excess ratio + safety stock |

Tabs: Portfolio Risk Matrix, Product Deep-Dive, AI Agent Copilot, Model Diagnostics & Drift,
Planner Decision & Audit.

The AI copilot rewords the rule-based recommendation and answers questions through an
OpenAI-compatible LLM gateway. A guardrail check compares numbers in the AI text against the
calculated values. If no gateway is configured, the dashboard falls back to the plain rule text.

## Run it

```bash
pip install -r requirements.txt

# optional: enable the AI features
cp .env.example .env     # then fill in the gateway URL, key and model

streamlit run week3_dashboard.py
```

`current_stock_SAMPLE.csv` is invented demo data. To use real stock, save a file named
`current_stock.csv` with the same columns (`product_id, current_stock, safety_stock, note`).

## Layout

```
week2_forecasting_v2.ipynb   Week 2 notebook
week3_dashboard.py           Week 3 Streamlit app
inventory_risk.py            risk logic (no Streamlit, testable on its own)
outputs/                     forecasts, metrics, charts
processed/                   weekly_sales.csv
scripts/                     gateway connection test scripts
archive/                     earlier notebook version
```
