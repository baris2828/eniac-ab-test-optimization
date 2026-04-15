# Eniac A/B Test — Optimizing the Homepage Call-to-Action

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/baris2828/eniac-ab-test-optimization/blob/main/notebooks/01_eniac_ab_test_analysis.ipynb)

Statistical analysis of a four-way A/B test designed to find the highest-performing call-to-action (CTA) button on the Eniac homepage. The study combines a Chi-Square test of independence with Bonferroni-corrected pairwise post-hoc tests and complements statistical significance with business-oriented metrics (relative lift, drop-off rate, homepage-return rate).

---

## Project Overview

Eniac, an online electronics retailer, suspected that its current CTA button was underperforming. Four variants of the button — crossing **color** (white vs. red) and **label** (`SHOP NOW` vs. `SEE DEALS`) — were randomly served to homepage visitors over the course of the test window. This repository contains the full analysis pipeline: data preparation, hypothesis testing, post-hoc corrections, and the final business recommendation.

![Current Eniac homepage](images/current_homepage.jpeg)

---

## Business Problem

The marketing team needs a data-driven answer to two questions:

1. **Does the choice of CTA button affect click-through rate (CTR) at all?**
2. **If yes, which variant should be rolled out to 100% of traffic?**

The four tested variants:

| Version | Color | Label       |
|:-------:|:-----:|:------------|
| A       | White | SHOP NOW    |
| B       | Red   | SHOP NOW    |
| C       | White | SEE DEALS   |
| D       | Red   | SEE DEALS   |

![Variant designs after feedback](images/web_design_after_feedback.png)

The business agreed on a significance level of `alpha = 0.10` — a relatively permissive threshold that trades a slightly higher false-positive risk for greater statistical power.

---

## Methodology

### 1. Data preparation
- Per-variant CSV exports with click counts for every page element.
- CTA clicks extracted by filtering on the button name (`SHOP NOW` / `SEE DEALS`).
- Visitor counts taken from the snapshot row of each export.
- A 4 × 2 contingency table (`clicks` vs. `non-clicks` by variant) is the input to the statistical tests.

### 2. Global Chi-Square test
A Pearson chi-square test of independence on the 4 × 2 table answers the global question *"Are the four variants equivalent?"*. Rejection of the null means at least one variant differs from the others.

### 3. Post-hoc pairwise tests with Bonferroni correction
With four variants there are `C(4, 2) = 6` pairwise comparisons. To control the family-wise error rate we apply the Bonferroni correction:

`alpha_adj = alpha / k = 0.10 / 6 ≈ 0.0167`

Each pair is then tested at the adjusted threshold.

### 4. Business framing — relative lift
Statistical significance is complemented by the **relative lift** of each variant over the current baseline (A), which expresses the difference in business-friendly percentage terms.

![Purchase funnel](images/Purchase_Funnel.png)

---

## Key Results

### Click-Through Rate

| Version | Color | Label     |   CTR   |
|:-------:|:-----:|:----------|:-------:|
| A       | White | SHOP NOW  |  2.02%  |
| B       | Red   | SHOP NOW  |  1.14%  |
| **C**   | **White** | **SEE DEALS** | **2.12%** |
| D       | Red   | SEE DEALS |  0.76%  |

- **Global Chi-Square test:** p-value ≈ `2.71e-48` — we reject H0. The variants are not equivalent.
- **Pairwise post-hoc (alpha_adj ≈ 0.0167):** both white buttons (A, C) significantly outperform both red buttons (B, D). The A-vs-C difference is **not** statistically significant.
- **Relative lift of C over A:** ~5%.

### Supplementary metrics

| Version | CTR (↑) | Drop-off rate (↓) | Homepage-return rate (↓) |
|:-------:|:-------:|:-----------------:|:------------------------:|
| **A**   |  2.02%  |     **62.0%**     |           5.3%           |
| B       |  1.14%  |       n/a         |           n/a            |
| **C**   | **2.12%** |     71.5%       |           4.7%           |
| **D**   |  0.76%  |       69.5%       |         **2.6%**         |

Drop-off and homepage-return rates were not available as CSV exports; values are read from the LMS dashboards. Version B tracking failed during the window, so B is missing on both supplementary metrics. **Bold cells mark the best (lowest / highest) value per column.**

![Eniac A/B test results](images/Eniac_A_B_Test_Results.png)

**Interpretation.**

- **Color is the dominant driver.** White buttons win decisively over red buttons on CTR.
- **Label is a secondary effect.** `SEE DEALS` (C) edges out `SHOP NOW` (A) on CTR, but the gap is not statistically significant on its own.
- **Supplementary metrics are a trade-off, not a clean win.** Version **A** has the lowest drop-off rate (~62% vs. C's ~71.5%) — more users stay on the landing page. Version **D** has the lowest return rate (~2.6% vs. C's ~4.7%) — fewest post-click regrets. Version **C** sits between them on both: it keeps post-click users engaged better than A, at the cost of more users dropping off the landing page before they click.
- The LMS dashboards report point estimates without confidence intervals, so these supplementary metrics are directional — they support the CTR analysis rather than override it.

---

## Final Recommendation

**Roll out Version C — the white `SEE DEALS` button.**

1. **Statistically**, C is tied with the current-best baseline A and significantly beats both red variants. Color is the dominant, statistically proven driver.
2. **Directionally**, C has the highest observed CTR (2.12%) and a ~5% relative lift over A — the largest observed lift on the only metric we can test rigorously.
3. **Supplementary metrics are mixed, not a clean win for C.** A has the lower drop-off rate, D has the lower return rate, and C sits between them on both. These LMS numbers come without confidence intervals (and B is missing entirely), so they are directional — they do not override the CTR result but do introduce a caveat.
4. **Risk is contained.** A and C are statistically equivalent on CTR, so switching from A to C cannot hurt CTR at the sample sizes tested. The drop-off gap is the main open question, which the follow-up test below is designed to resolve.

**Next steps.**
- Monitor post-launch CTR and downstream conversion for two weeks.
- Run a confirmatory **A vs. C head-to-head** test instrumented across the full funnel (landing → click → purchase → homepage return) so the drop-off / return-rate trade-off can be resolved on measured revenue rather than on directional dashboards.

![Power calculator for experiment duration](images/power_calculator_to_calculate_the_experiment_duration.png)

---

## Repository Structure

```
github_repo_ready/
├── data/                 # Raw per-variant CSV exports (eniac_a–d.csv)
├── images/               # Figures used in the README and supporting materials
├── notebooks/
│   └── 01_eniac_ab_test_analysis.ipynb
├── .gitignore
├── README.md
└── requirements.txt
```

---

## Reproducing the Analysis

```bash
# 1. Create and activate a virtual environment (optional but recommended)
python -m venv .venv
source .venv/bin/activate        # Linux / macOS
.venv\Scripts\activate           # Windows

# 2. Install dependencies
pip install -r requirements.txt

# 3. Open the notebook
jupyter notebook notebooks/01_eniac_ab_test_analysis.ipynb
```

The notebook reads the CSVs from `../data/` relative to the notebook directory; no internet access is required.

---

## Tech Stack

- Python 3.11
- `pandas`, `numpy` — data handling
- `scipy.stats` — chi-square test of independence
- `matplotlib`, `seaborn` — visualization
