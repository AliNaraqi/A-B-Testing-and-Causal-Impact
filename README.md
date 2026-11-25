# A/B Testing and Causal Impact

Does a marketing campaign cause more purchases, or do the people who see the ad buy more because they were already likely to buy?

This project answers that on the [Criteo Uplift dataset](https://huggingface.co/datasets/criteo/criteo-uplift): 13,979,592 users from real incrementality tests. A random share of users was withheld from advertising. `treatment` is that assignment. `exposure` is whether an ad was actually shown. Only assignment is randomized.

Open `criteo_incrementality.ipynb`. The cells have already been run.

## Result

Assigning a user to the campaign raises the purchase rate by **0.115 percentage points** (95% interval 0.108 to 0.122). That is a **59%** relative lift over the control purchase rate. In the treated arm of 11,882,655 users, the assignment produces about **13,700** extra purchases (interval 12,878 to 14,479).

Comparing users who were shown an ad with everyone else gives **5.25 percentage points**. That exposure comparison is **46 times** the assignment effect. The platform chooses who sees the ad, so exposure is mixed up with purchase intent.

| Estimand | What is compared | Purchase-rate difference | 95% interval | Use it? |
|---|---|---:|---|---|
| Naive | Exposed vs not exposed | 5.25 pp | 5.18 to 5.32 | No. Confounded by targeting. |
| Per-protocol | Exposed vs control | 5.18 pp | 5.12 to 5.25 | No. Exposure happens after assignment. |
| ITT | Assigned vs withheld | 0.115 pp | 0.108 to 0.122 | Yes. Effect of running the campaign. |
| CACE | Reachable users | 3.20 pp | 3.01 to 3.38 | Yes, if the question is about people who can be shown the ad. |

Only **3.6%** of assigned users were actually shown an ad, and no control user was shown one. CACE is the assignment effect divided by that 3.6% exposure rate. It needs the exclusion restriction: being assignable does not change purchases for someone who would never see the ad. The file can confirm one-sided compliance. It cannot prove exclusion.

Visits move the same way. The assignment effect on visits is **1.03 percentage points** (relative lift 27%). The exposure comparison is **37.9 percentage points**.

At a placeholder margin of **$40** per purchase and a **$8** CPM, the expected value of assignment is about **$544,000** (interval $512,000 to $576,000). Spend on the observed exposures is about $3,400. Crediting every purchase by an exposed user to the ad produces about **$921,000** of revenue. The file has no prices and no costs. Change `MARGIN` and `CPM` in the notebook before using the dollars. Absolute rates in this file are also distorted: Criteo sub-sampled the rows so the original incrementality level cannot be read off them. Contrasts between arms are the part that remains meaningful.

The largest covariate imbalance on `f0`–`f11` is **0.049** standard deviations. The usual concern line is 0.10. On an 800,000-row sample, ranking users by a T-learner beat random targeting on the holdout.

## What is in the notebook

1. A small simulation where intent drives both exposure and purchases, so the exposure gap is visibly too large.
2. A streaming load of the gzip into parquet. The unpacked CSV is never written.
3. Checks: row count, nulls, binary flags, zero exposed controls, published base rates, covariate balance.
4. Absolute lift, relative lift, and Newcombe intervals for the four estimands, on purchases and on visits.
5. The same assignment effect from the two arm means, with a robust standard error.
6. Lin's covariate-adjusted estimator on the 800,000-row sample.
7. How much precision the 85/15 split gives up relative to equal arms.
8. Expected value, breakeven margin, breakeven CPM, and incremental ROAS under editable assumptions.
9. A T-learner and a Qini curve against random targeting.

## Run it

Python 3.11. The project folder name contains a colon, so the virtual environment lives outside it.

```bash
python3.11 -m venv "$HOME/.cache/ab-causal-venv"
"$HOME/.cache/ab-causal-venv/bin/pip" install -r requirements.txt
"$HOME/.cache/ab-causal-venv/bin/python" -m ipykernel install --user --name ab-testing --display-name "Python 3.11 (A/B testing)"
```

In Jupyter, select the kernel **Python 3.11 (A/B testing)** and open `criteo_incrementality.ipynb`. The first data cell downloads about 300 MB from Hugging Face and builds `data/criteo.parquet`. Later runs reuse both files. `data/` is gitignored.

`campaign_causal_impact.py` writes a new `campaign_causal_impact.ipynb` from scratch. It stops if that file already exists, so it does not erase a notebook that has results in it.

## License and citation

The Criteo Uplift dataset is **CC-BY-NC-SA-4.0**. Non-commercial use, with attribution. Do not commit the data file.

Diemert, Eustache, Artem Betlei, Christophe Renaudin, and Massih-Reza Amini. “A Large Scale Benchmark for Uplift Modeling.” AdKDD, KDD 2018.

The file used here is the rebalanced v2 release described in arXiv:2111.10106 (`criteo-research-uplift-v2.1.csv.gz`).

## Limits

There is no timestamp, so this file cannot support a sequential peeking analysis. There is no revenue column, so dollar figures are scenarios. Features are anonymized, so a high-uplift decile is a statistical group, not a business segment.
