# PulseMetrics — Churn Prediction (Project 2)

**Tools:** Python (pandas, scikit-learn, matplotlib, seaborn) in Jupyter
**Project Type:** A portfolio project that builds on the PulseMetrics SaaS dataset used in
[Project 1](https://github.com/ganeshshiv-code/pulsemetrics-churn-retention-analysis)

## Business Context

Project 1 helped explain **who churned and when** using SQL and a Power BI dashboard. Building on that analysis, this project asks the next business question: **Can we identify customers who are at risk of churning before they actually leave?**

## Finding

Tenure and plan tier are the strongest predictors of churn — newer, lower-tier
customers are consistently more likely to churn. Support ticket volume and
satisfaction show very weak *linear* correlation with churn (0.06 and -0.06),
but Project 1's bucketed SQL analysis found a real, strong pattern concentrated
at the extreme (8+ tickets) — a reminder that correlation coefficients can miss
non-linear effects. The tuned model achieves **67.9% recall**, correctly
identifying roughly 2 out of every 3 customers who will actually churn.

**So what:** A model that only catches 1 in 3 at-risk customers would leave most
preventable churn undetected. By deliberately trading some accuracy for recall
— prioritizing catching real churners over avoiding false alarms — PulseMetrics
gets a usable early-warning signal, because missing a real churner costs the
full customer relationship, while a false alarm only costs an unnecessary
retention outreach.

**Recommendation:** Score all currently active customers monthly with this
model and prioritize proactive retention outreach — the same 60-day onboarding
check-in from Project 1 — for anyone flagged as high-risk, focusing especially
on new customers on lower-tier plans with high support-ticket volume.

## Methodology

1. **Feature engineering** — aggregated `support_tickets` and `monthly_usage`
   (originally one-row-per-ticket / one-row-per-month) down to one row per
   customer *before* merging, to avoid row-count fan-out. Nulls from customers
   with zero tickets were filled contextually (0 for count, dataset mean for
   satisfaction — not the same fill for both, since a 0 satisfaction score
   would falsely imply extreme dissatisfaction).
2. **Encoding** — `acquisition_channel` (nominal) one-hot encoded;
   `plan_id` (ordinal, real tier order) left as-is.
3. **Modeling** — logistic regression, with a baseline run vs. a
   `class_weight='balanced'` run to correct for the ~65/35 class imbalance.
   Baseline recall (28.3%) actually underperformed the naive "predict majority
   class" floor (65% accuracy) — the balanced model raised recall to 67.9% at
   a small accuracy cost, a deliberate and justified trade-off for this
   business problem.
4. **Feature importance** — standardized coefficients confirm `tenure_months`
   and `plan_id` as the strongest predictors, consistent with Project 1's SQL
   findings. `total_tickets` and `avg_satisfaction` are correlated with each
   other (-0.36, multicollinearity), which explains why the model concentrates
   weight on one over the other rather than double-counting the same signal.

## Visuals

![Tenure and Satisfaction by Churn Status](images/tenure_satisfaction_boxplots.png)
![Feature Correlation Heatmap](images/correlation_heatmap.png)

## Repo Structure

```
├── README.md
├── notebook/
│   └── project2_churn_prediction.ipynb
└── images/
    ├── tenure_satisfaction_boxplots.png
    └── correlation_heatmap.png
```
