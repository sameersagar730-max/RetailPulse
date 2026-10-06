# Demo Video Script — RetailPulse

**Target length:** 5 minutes. One screen recording of the running dashboard with a voiceover track.

Every number in this script is taken from `reports/RetailPulse_Report.md`. If a number on screen
disagrees with this script, the report is the thing to fix first, then re-record.

**Total speaking time:** about 4 minutes 40 seconds of the 5 minutes, leaving roughly 20 seconds
of silence for the transitions between pages.

---

## Before you record

```bash
streamlit run app/Home.py
```

- Browser window at 1440x900, devtools closed.
- Keep the terminal out of frame, or close it after the server starts.
- Confirm the sidebar shows all five pages before you start the capture.
- If the live URL is down, open `reports/screenshots/` in a second window and use the five PNGs.
  Say on the recording that these are screenshots and not the live demo, because a screenshot
  cannot show load time or that a filter responds.

---

## 0:00 — Hook (Home, 30s)

Open on the Home page, unmodified.

> "970,998 line items. 8,000 customers. 50 stores. Two years. One question: what should we
> stock, and who do we call first?"

Scroll slowly to the four KPI tiles. **Do not** scroll past the zero-demand finding yet.

---

## 0:30 — The fact that shapes everything (Home, 30s)

Point at the 77.9% zero-demand callout.

> "Before any model: 77.9% of store-product-weeks have zero sales. That one number decides our
> metric, our architecture, and the inventory method. MAPE is undefined on three quarters of this
> panel, so we score forecasts with WAPE instead."

This is the spine of the project. Cut it and the rest is unexplained.

---

## 1:00 — Promotions (Home, 30s)

Point at the promotion finding.

> "The business configured a 1.56x promotion lift. We measure 0.9999x. Promoted lines sell 3.8836
> units versus 3.884 unpromoted. The 26% discount is the entire difference — realised revenue per
> unit drops to 0.6931x."

**If asked why the lift is zero:** promotions are targeted, not randomised, so this is an
association, not a causal effect.

---

## 1:30 — Demand forecasting (30s)

Navigation → Demand Forecasting.

> "Three horizons, five candidate models, scored on WAPE. Prophet wins at one week — 0.0068.
> Seasonal naive wins at two and four weeks. The four-week path is what matters, because '30
> days ahead' is four weeks on a weekly panel."

Show the model selection table, then the chart: 12 actual weeks followed by the forecast.

> "Fitted on the pooled weekly total and split by historical share. Fitting 7,500 separate series
> will not finish inside the batch budget."

**If asked about the 12% MAPE target:** say the pooled number is 0.0068 but it excludes
zero-demand weeks by construction, so it is not evidence at the panel level. Do not claim the
target was met.

---

## 2:00 — Customer segments (30s)

Navigation → Customer Segments.

> "k=6, silhouette 0.1998. The raw best was k=5 at 0.2021 — a 0.002 gap, which is noise at 8,000
> customers. DBSCAN found 2 clusters with 38.1% noise, so it did not give us a usable alternative
> here."

Show the segment table and the revenue-vs-customers chart. Note that segments above the diagonal
earn more than their headcount share.

> "Silhouette near 0.20 means these clusters overlap. They are a targeting summary, not separate
> populations."

---

## 2:30 — Churn risk (40s)

Navigation → Churn Risk.

> "XGBoost, split by date rather than randomly — snapshots repeat the same customers, so a random
> shuffle puts near-duplicates on both sides of the split."

Metrics, in this order and with no spin:

> "Precision at the top 20%: 0.7703, against a 0.75 target. Lift 1.81. That part worked.
> AUC is 0.7315 against a target of 0.88 — we missed that by 0.15."

**Say the miss out loud.** It is in the report, and volunteering it is the difference between a
demo that is trusted and one that is not.

> "SHAP says recency dominates at 0.52, then frequency at 0.23. The model is largely measuring
> whether people have stopped showing up."

---

## 3:10 — Inventory recommendations (50s)

Navigation → Inventory Recommendations.

> "Newsvendor reorder quantity, one-week lead time. Purchase cost 60%, holding 25%, lost margin
> 40%."

> "496 pairs to reorder, 2,081 units, INR 48,771."

Expand the comparison:

> "A normal approximation would order 5,384 more units across 1,022 more pairs. We do not ship
> that. With 78% zero weeks, mean plus z times sigma has no meaning in the upper tail, so safety
> stock comes from a Poisson quantile."

**Then the backtest. Do not skip it — this is the weakest number in the project:**

> "Walk-forward over the last 13 weeks against a naive-mean baseline: a 2.09% reduction, against
> a 25 to 40% target. We missed it. The interesting part is why — overstock drops from 18,302
> units to 44, and understock rises from 2,399 to 20,224. The policy did not reduce total error,
> it moved the error from one side of the ledger to the other."

> "One thing that needs a decision rather than a re-tune: the cost assumptions give a 0.9928
> critical ratio, which implies a 99.28% service level. That is above the 95% we configured. It is
> a calibration question for whoever owns the brief."

---

## 4:00 — Drift monitoring (30s)

The drift monitor is not a dashboard page. Show it as a terminal cutaway, not narrated over a
chart.

```bash
python -m src.drift
```

> "We compare 2024 against 2025 — volume, price, discount and category mix — using PSI and total
> variation distance. On this data nothing drifted; every column came back under the 0.1 stable
> threshold."

**Then say why a clean result is weak evidence, because it is:**

> "A monitor that never fires is either reassuring or broken. So the thresholds are exercised by a
> synthetic injection test that forces a shift and confirms the PSI responds. The 2024-vs-2025
> null result is real, but it is not proof the detector works on your data."

The Evidently HTML report is at `reports/drift/evidently_drift.html`.

---

## 4:30 — Closing (30s)

Back to Home.

> "Seven functional requirements, 517 tests, and a dashboard that loads in about a tenth of a
> second because it reads precomputed aggregates and never the 103 MB of raw inputs."

> "Four targets were missed, and they are all in the report: churn AUC is 0.73 against 0.88, the
> panel-level MAPE target is not demonstrated because the forecast is pooled, the inventory
> reduction is 2% against a 25% target, and a silhouette near 0.20 means the segments overlap.
> Everything here is reproducible from the seeded generator."

---

## If your mentor asks — quick answers

**Why did you pick WAPE over MAPE?**
Because 77.9% of store-product-weeks have zero demand. MAPE divides by actual demand, so it is
undefined on three quarters of the panel. WAPE divides by total volume and stays defined.

**Why is the churn AUC only 0.73?**
Recency and frequency dominate the SHAP values, which means churn here is mostly "the customer
stopped visiting". That is a weak signal to predict from. The fix is enrichment data we do not
have — support tickets, email engagement, payment failures — not a different model. Splitting by
date rather than randomly already removed the leakage that would have inflated the score.

**Why is the inventory reduction only 2% when the target was 25%?**
The newsvendor policy is the right method, but the backtest shows overstock collapsing from 18,302
units to 44 while understock rises from 2,399 to 20,224. The policy did not reduce total error, it
moved it across the ledger. With a 0.9928 critical ratio the policy is optimised for a 99.28%
service level, far above the 95% we configured, so it deliberately runs lean. Calibrating that
ratio is a business decision, not a code change.

**Why silhouette 0.20 — are the segments any good?**
They are a targeting summary, not distinct populations. k=6 scored 0.1998 against a best-raw 0.2021
at k=5, a gap small enough to be noise at n=8,000. The useful output is the ranking: which
segments earn more than their headcount share.

**Why pool the forecast instead of fitting per product-store?**
7,500 separate series will not finish inside the batch budget. Pooling the weekly total and
splitting by historical share gives a defensible four-week path, and the honest limitation is
that the pooled WAPE of 0.0068 excludes zero-demand weeks, so it is not panel-level evidence for
the 12% MAPE target.

**Why did nothing drift between 2024 and 2025?**
The data is synthetic from a seeded generator, so the distributions are stable by construction. We
report it, but we also run a synthetic injection test that forces a shift and confirms the PSI
responds. A monitor that never fires is not evidence it works.

**How would this run in production?**
`Dockerfile.pipeline` builds the heavy stack; the dashboard image installs only pandas, numpy and
streamlit because it reads precomputed aggregates. A nightly Airflow DAG refreshes the aggregates
and runs the drift monitor, then serves the dashboard from committed CSV output.

---

## Recording notes

- **Do not** say "AI-generated". The brief disqualifies that. Speak in the first person about what
  you built and why.
- If a number on screen disagrees with this script, the screen is correct only if the report is
  also updated. Fix the report, then re-record.
- Skip the DBSCAN segment if you run short on time. **Do not skip the AUC miss, the inventory
  backtest miss, or the drift segment** — those are the three places where volunteering the
  weakness is what makes the rest of the demo credible.