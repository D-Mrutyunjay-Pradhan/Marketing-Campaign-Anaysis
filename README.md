# Marketing-Campaign-Anaysis
# 📊 Facebook vs AdWords: A/B Testing Analysis of 2019 Ad Campaigns

### Which ad platform delivers more conversions, more clicks and better cost-effectiveness?

A data analytics project comparing two digital ad campaigns that ran every day of 2019, one on **Facebook** and one on **Google AdWords**, using exploratory analysis, hypothesis testing, regression and time series analysis to recommend where marketing budget should go.

---

## 📌 Table of Contents
1. [Business Problem](#-business-problem)
2. [Key Result](#-key-result)
3. [Dataset](#-dataset)
4. [Methodology](#-methodology)
5. [Findings](#-findings)
6. [Recommendations](#-recommendations)
7. [Limitations](#-limitations)
8. [How to Run](#-how-to-run)

---

## 💼 Business Problem

A marketing agency wants to maximise return on investment for client advertising. Two campaigns ran in 2019, one on Facebook and one on AdWords. To allocate budget efficiently, the agency needs to know which platform performs better on **clicks, conversions and cost-effectiveness**.

**Primary research question:** Which ad platform is more effective in terms of conversions, clicks, and overall cost-effectiveness?

**Supporting questions**
- How often does each platform reach high-conversion days?
- Do more clicks really lead to more conversions on both platforms?
- Is the Facebook conversion advantage statistically significant?
- How many Facebook conversions can be expected for a given number of clicks?
- How do conversions and cost per conversion change over months and weekdays?

---

## 🏆 Key Result

> **Facebook is the stronger platform.** It produced about **2× the daily conversions** (11.74 vs 5.98) at about **one-third the cost per conversion** ($7.48 vs $22.57). AdWords attracts more clicks, but far fewer of them convert.

| Metric (2019 totals) | Facebook | AdWords |
|---|---:|---:|
| Total conversions | 4,286 | 2,183 |
| Total spend | $32,040 | $49,266 |
| Conversion rate (conversions / clicks) | 26.7% | 9.9% |
| Click-through rate (clicks / views) | 2.02% | 1.28% |
| Cost per click | $1.99 | $2.24 |
| **Cost per conversion** | **$7.48** | **$22.57** |

---

## 🗂️ Dataset

`marketing_campaign.csv` has **365 rows × 17 columns**, one row per day from 1 Jan to 31 Dec 2019, with both campaigns side by side. Each campaign is split into monthly codes (e.g. `FB_Jan19`, `AW_Jan19`).

| Field | Description |
|---|---|
| `Date` | Calendar day (1 Jan to 31 Dec 2019) |
| Facebook / AdWords Ad Campaign | Monthly campaign code (12 per platform) |
| Ad Views | Times the ad was shown |
| Ad Clicks | Clicks received |
| Ad Conversions | Conversions resulting from the ad |
| Cost per Ad | Daily spend in US dollars |
| Click-Through Rate | Clicks ÷ views |
| Conversion Rate | Conversions ÷ clicks |
| Cost per Click | Ad cost ÷ clicks |

**Data preparation**
- Converted `Date` to datetime.
- Stripped `$` and `%` symbols from text columns and converted them to numbers.
- No missing values and no duplicate rows.
- The supplied cost-per-click column differs from cost ÷ clicks by about 0.6% per day on average (likely rounding), so all cost metrics are **recomputed from totals**.

**Daily descriptive statistics**

| Metric | Platform | Mean | Std dev | Min | Max |
|---|---|---:|---:|---:|---:|
| Ad views | Facebook | 2,179.7 | 618.1 | 1,050 | 3,320 |
| | AdWords | 4,717.2 | 561.1 | 3,714 | 5,760 |
| Ad clicks | Facebook | 44.0 | 12.1 | 15 | 73 |
| | AdWords | 60.4 | 14.4 | 31 | 89 |
| Ad conversions | Facebook | 11.7 | 2.9 | 5 | 19 |
| | AdWords | 6.0 | 1.6 | 3 | 9 |
| Daily spend ($) | Facebook | 87.8 | 28.3 | 34 | 144 |
| | AdWords | 135.0 | 32.3 | 73 | 197 |

---

## 🔬 Methodology

| Step | What was done | Tools |
|---|---|---|
| 1. Prepare | Loaded 365 daily rows, parsed dates, cleaned `$` and `%` fields | pandas |
| 2. Explore | Distributions of clicks and conversions, conversion buckets, click-to-conversion correlation | pandas, seaborn, matplotlib |
| 3. Test | Welch two-sample t-test (unequal variances), one-tailed, α = 0.05 | SciPy |
| 4. Model | Simple linear regression of conversions on clicks (R², MSE) | scikit-learn |
| 5. Trend | Monthly and weekday aggregation, cost per conversion, Engle–Granger cointegration test | statsmodels |

Rates are calculated from **2019 totals** rather than by averaging daily percentages, so low-traffic days don't get too much weight.

---

## 🔍 Findings

### 1. Facebook reaches high-conversion days far more often
Facebook hit 11+ conversions on **236 of 365 days (65%)**, including 47 days above 15. AdWords **never reached 11**: 156 days had under 6 conversions and the other 209 had 6–10. The gap is structural, not a handful of lucky days.

### 2. Facebook leads on every efficiency measure
Higher CTR (2.02% vs 1.28%), higher conversion rate (26.7% vs 9.9%, about 2.7× better), cheaper clicks ($1.99 vs $2.24) and roughly **3× cheaper cost per conversion**.

### 3. Clicks matter on Facebook, much less on AdWords
Click-to-conversion correlation is **r = 0.87 on Facebook** but only **r = 0.45 on AdWords**, suggesting other factors (landing page, keyword intent, audience fit) limit AdWords.

### 4. The difference is statistically and practically significant

| Statistic | Value |
|---|---|
| H0 / H1 | µ Facebook ≤ µ AdWords / µ Facebook > µ AdWords |
| Test | Welch two-sample t-test, one-tailed, α = 0.05 |
| Mean conversions per day | Facebook 11.74 vs AdWords 5.98 |
| Difference in means | 5.76 (95% CI 5.42 to 6.11) |
| t statistic | 32.88 |
| p-value | 9.3e-134 |
| Cohen's d | 2.43 (large) |
| Decision | **Reject H0** |

A supplementary test shows AdWords gets significantly more clicks per day (60.4 vs 44.0, p < 0.001): **AdWords wins on traffic volume, Facebook wins on what that traffic achieves.**

### 5. Regression: how many conversions per click?

| Item | Facebook | AdWords |
|---|---:|---:|
| R² | 76.35% | 20.1% |
| MSE | 2.02 | 2.11 |
| Slope (conversions per extra click) | 0.211 | 0.051 |
| Intercept | 2.47 | 2.92 |

Fitted Facebook model: **Conversions ≈ 2.47 + 0.21 × Clicks** (every ~5 extra clicks add about 1 conversion). For example, 50 clicks → 13.0 conversions and 80 clicks → 19.3 conversions.

### 6. Time series
- **Facebook trends upward:** conversions rose from 334 (Jan) to 413 (Dec), +24%. The fitted trend is +0.0075 conversions/day each day (p < 0.001).
- **AdWords is flat** (p = 0.88).
- **Cost per conversion** stayed between $6.88 and $8.38 on Facebook and $21.55 and $23.93 on AdWords.
- **Weekday timing is a minor lever:** every weekday is within about 4% of the weekly average (Facebook peaks Tue/Wed, lowest Thursday).
- **Cointegration caveat:** the Engle–Granger test rejected "no cointegration" (p ≈ 2e-26), but the direct spend-vs-conversions correlation is weak (r = −0.13). The data do **not** show that spending more on a given day raises that day's conversions.

---

## ✅ Recommendations

1. **Shift budget toward Facebook.** Illustratively, $10,000 of spend at 2019 average rates yields about 1,338 conversions on Facebook vs about 443 on AdWords. Returns will diminish at scale, so move budget in stages.
2. **Plan Facebook volume with click targets.** Use ~0.21 conversions per click to turn conversion goals into click and budget targets.
3. **Diagnose AdWords before funding it.** Audit keywords and search intent, ad-to-landing-page match, landing-page speed and offer, and audience targeting. Re-test with a small budget.
4. **Time the spend.** Track monthly cost per conversion and lean into historically low-cost months (e.g. November).
5. **Validate with a controlled test.** Run a randomised budget-split test in 2020 with comparable audiences and the same offer on both platforms.

---

## ⚠️ Limitations

- **Not a randomised experiment.** The platforms reach different audiences and were not randomly assigned, so part of the gap may reflect audience differences rather than platform quality alone.
- **One year, one campaign pair.** Results may not hold in other periods, markets or creative formats.
- **Conversions are not revenue.** No order value or lifetime value in the data, so dollar ROI cannot be confirmed.
- **Constant returns assumed.** Cost per conversion may rise if spend is scaled up sharply.
- **Simple models.** The regression uses one predictor and was fitted and scored on the same 365 days (no holdout).
- **Data rounding.** Spend is in whole dollars and the supplied cost-per-click column doesn't exactly match cost ÷ clicks.

---

## ▶️ How to Run

1. Clone the repo
   ```bash
   git clone https://github.com/<your-username>/<repo-name>.git
   cd <repo-name>
   ```
2. Install dependencies
   ```bash
   pip install pandas numpy matplotlib seaborn scipy scikit-learn statsmodels jupyter
   ```
3. Make sure `marketing_campaign.csv` is in the same folder as the notebook.
4. Launch the notebook
   ```bash
   jupyter notebook AB_Testing_Analysis.ipynb
   ```

**Sanity check:** total conversions Facebook 4,286 / AdWords 2,183 · t = 32.88 · Facebook regression R² = 76.35%.

**Notebook notes:** the original notebook labels cost per conversion as "CPC" and computes it only for Facebook, and its weekday labels are shifted by one day (pandas weekday 0 is Monday). The report corrects both.
