# Capstone Report — Search Intelligence Content Refresh Queue

- **Author:** Sathwik Peddi
- **Lane:** Machine Learning Capstone
- **Repo:** https://github.com/sathwikpeddi0712/ml-internship-flyrank
- **Date:** 2026-07-22

## 0. Abstract

Organic search visibility is subject to decay due to changes in search engine algorithms, competitor content updates, and shifts in search demand. Identifying when an article is decaying and should be refreshed is a key task for content operations. This project builds an automated, machine-learning-driven priority queue for content editors to maximize visibility recovery under strict capacity constraints. By analyzing historical search console performance trends, the model ranks content according to its probability of decline. Evaluating on a client holdout validation split, our best Random Forest model achieves a **Precision@50 of 0.740**, representing a **3.08x lift** over a standard manual rule baseline.

## 1. Problem framing

Editorial teams face a strict capacity constraint: reviewing, rewriting, and re-publishing articles requires substantial human effort, meaning only 20 to 50 pages can be updated weekly. The business decision is deciding which pages in a large portfolio (often 10,000+ pages) are decaying and should be prioritized for a refresh.
- **Unit of Analysis:** A single page (content item) for a specific client site over a 90-day window.
- **Output:** A prioritized top-K review queue sorted by predicted probability of decay.
- **Human Action:** An editor reviews the page, updates statistics, adds fresh content, and re-submits the URL.
- **Cost of Wrong Call:** If a high-volume page is missed, organic traffic declines unchecked (high false negative cost). If a stable page is prioritized, editor time is wasted (false positive cost).
Machine learning is ideal here because the interaction of search rankings, search volume, CTR, and scroll depth is too complex for standard, flat manual rules to prioritize effectively.

## 2. Data safety

This project uses the anonymized starter dataset (`data/raw/content_refresh_anonymized.csv`), consisting of 30,000 rows across 32 client sites.
- **Columns Excluded:** All future-window metrics (e.g. `impressions_last_30d`, `clicks_last_30d`) were excluded to prevent future target leakage. Labels derived from future outcomes (`trend_direction`, `trend_pct`) were strictly removed. Client names, raw domains, and queries are replaced with pseudonymous IDs (`client_id`, `content_id`).
- **Data Leakage Risks:** Row-level randomized splits would leak client-specific domain authority or seasonality into the validation set. Grouping by `client_id` for splits eliminates this risk.

## 3. Baseline

We built a heuristic baseline score using a combination of visibility, age, and position metrics:
- **Formula:** `baseline_score = 0.40 * visibility_score + 0.30 * freshness_risk_score + 0.25 * position_opportunity_score + 0.05 * depth_gap_score`
- **Why it is a fair comparison:** It uses the same historical signals (impressions, age, and position) as our models.
- **Baseline Performance (Client Holdout):**
  - **ROC AUC:** 0.627
  - **Average Precision:** 0.468
  - **Precision@50:** 0.240
  - **Recall:** 0.189
  - **F1 Score:** 0.274

## 4. Model / analysis

We evaluated three machine learning classifiers using `scikit-learn`: Logistic Regression, Decision Trees, and Random Forests.
- **Target Definition:** `is_declining_label` = 1 when Search Console impressions drop by >20% month-over-month (`impressions_last_30d < 0.8 * impressions_prev_30d`). Otherwise, it is 0.
- **Features Used:** 52 features representing past-window activity, including total impressions, click-through rate (CTR), average position, scroll depth, engagement rate, content age, and page authority.
- **Model Choice:** Random Forest was selected as the best architecture because its bagging ensemble reduces variance, handles continuous and categorical signals without manual scaling, and learns non-linear interaction boundaries.

## 5. Evaluation

We used a **Grouped Client Split (`client_holdout`)** to evaluate models:
- **Design:** Partitioned by `client_id` so that 20% of sites (7 clients, 2,325 rows) are held out for testing, while 80% (25 clients, 27,675 rows) are used for training. This measures generalization to completely unseen websites.
- **Results Comparison (Holdout Test Split):**
  - **Base Rate of Decline:** 54.2%
  - **Baseline Rules:** Precision@50 = 0.240 · ROC AUC = 0.627
  - **Logistic Regression:** Precision@50 = 0.400 · ROC AUC = 0.700
  - **Decision Tree:** Precision@50 = 0.540 · ROC AUC = 0.742
  - **Random Forest (Best):** Precision@50 = 0.740 · ROC AUC = 0.750

### Error Analysis
- **False Positives:** High-impression, zero-click pages (e.g. content targeting informational search queries) get flagged as decaying by the model. However, their position is stable or growing, resulting in a false alarm.
- **False Negatives:** Very low-volume pages (e.g. 2-3 impressions over 90 days) trigger the binary label easily with minor random fluctuations, but are correctly assigned a low probability by the model since they have no search authority to lose.

## 6. Interpretation

The Random Forest model relies heavily on search volume and consistency signals:
1. `days_with_impressions` (15.8%): Measures query demand consistency.
2. `log_impressions_90d` (12.8%): Captures page search authority.
3. `avg_position` (10.9%): High rankings are sensitive to algorithm and competitor shifts.
4. `content_age_days` (9.5%): Reflects content staleness.

Negative/surprising results: Scroll depth and engagement rate had relatively low importance, suggesting that off-page search signals are much stronger predictors of near-term visibility decay than on-page user engagement.

## 7. Recommendation

We translate the model outputs into a **Content Action Playbook** for editorial teams:
- **Thin/High-Impression Content** -> `expand_and_refresh`
- **High-Position/Low-CTR Content** -> `refresh_and_review_ctr`
- **High-Traffic/Declining Content** -> `refresh`
- **Zombie/Low-Traffic Legacy Content** -> `prune_or_redirect`

### Operational Limits & Checklist
- **Pre-Execution Checklist:** Editors must check: (1) has query intent shifted? (2) are stats/facts accurate? (3) is the page canonical and indexable?
- **The No-Go List:** NEVER automate rewrites of flagship/YMYL pages, and NEVER auto-execute 301 redirects or deletions without manual approval.

## 8. Reproducibility

To reproduce this study from a fresh clone:
1. Clone the repository and install requirements:
   ```bash
   pip install -r requirements.txt
   ```
2. Run the complete pipeline:
   ```bash
   python scripts/run_all.py
   ```
This script cleans the data, runs the baseline, trains the models using random seed `42` for the client split, and writes all validation receipts to `work/outputs/` and charts to `work/figures/`.

## 9. Acknowledgments & data credit

Built on the FlyRank ML Internship dataset. For more details on the search intelligence and dataset context, visit [FlyRank](https://flyrank.ai).
