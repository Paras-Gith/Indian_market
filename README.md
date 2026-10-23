# Indian_market: NIFTY 50 End-to-End Data Science

A five-week data science project on the **NIFTY 50**, India's benchmark equity index. It follows the full workflow on one dataset: data acquisition and cleaning, visual storytelling, statistical hypothesis testing, machine learning, and a final report with recommendations.

**The two questions the project answers**

1. What has the Indian market actually done over the last ten years, and how risky was the ride?
2. Can its next-day direction be predicted from past prices?

**Short answer:** the index rose about 192% (roughly 11% a year, price return only), but with sharp, clustered risk, including a 38.4% fall in March 2020. Simple models could not predict next-day direction better than a coin toss.

---

## Contents

- [Dataset](#dataset)
- [Repository structure](#repository-structure)
- [How to run](#how-to-run)
- [Weekly summary](#weekly-summary)
- [Chart guide: what each chart shows and why it helps](#chart-guide)
- [Key results](#key-results)
- [Limitations](#limitations)
- [Author](#author)

---

## Dataset

| Item | Detail |
|---|---|
| Source | Yahoo Finance, via the `yfinance` library |
| Ticker | `^NSEI` (NIFTY 50 index) |
| Period | 1 June 2016 to 30 June 2026 |
| Size | 2,483 trading days, 7 columns |
| Columns | Date, Adj Close, Close, High, Low, Open, Volume |

The NSE website limits each download to 365 days, so a single scripted download was used instead. The download is skipped if the file already exists, so re-running the notebook does not call Yahoo again.

---

## Repository structure

```
Indian_market/
├── data/
│   ├── NIFTY50_stock_history.csv     # raw download (Week 1)
│   └── nifty_cleaned.csv             # cleaned data used by Weeks 2-4
├── week1_cleaning_eda.ipynb          # acquisition, audit, cleaning, EDA
├── week2_visualization.ipynb         # six-chart visual story
├── week3_hypothesis_testing.ipynb    # four hypothesis tests
├── week4_ml_model.ipynb              # next-day direction models
└── README.md
```

Week 1 produces `data/nifty_cleaned.csv`. Weeks 2 to 4 load that file, so **run Week 1 first**.

---

## How to run

```bash
git clone https://github.com/Paras-Gith/Indian_market.git
cd Indian_market
pip install pandas numpy matplotlib seaborn scipy statsmodels scikit-learn yfinance jupyter
jupyter lab
```

Open the notebooks in order (Week 1 to Week 4) and use **Kernel > Restart & Run All**.

Notes:
- `yfinance` may print harmless `Pandas4Warning` messages. They do not affect the results.
- If Yahoo is unreachable, place your own CSV with the same columns at `data/NIFTY50_stock_history.csv` and the notebook will use it.

---

## Weekly summary

### Week 1: Data acquisition, cleaning and EDA
Audited the raw data for missing values, duplicates, duplicate dates and inconsistent prices (all clean). Fixed the two real issues: dates stored as text were converted to datetime, and **28 days (1.1%) with a volume of exactly zero** were recoded as missing, since a zero volume on a normal trading day is not plausible. No rows were dropped, so the time series stays continuous. Engineered `Daily_Return_%`, `MA_50`, `MA_200` and `Year`.

### Week 2: Visualization and storytelling
Six charts tell one story, from growth, to risk, to losses, to the shape of daily returns (see the chart guide below).

### Week 3: Statistical hypothesis testing
Four hypotheses tested at the 0.05 level (see key results).

### Week 4: Machine learning
Logistic regression and a depth-3 decision tree predict whether tomorrow's close is up or down, using a **chronological** 80/20 split (1,826 training days, 457 test days) so that no future information leaks into training.

Features: the last three daily returns, the distance of price from its 50-day and 200-day moving averages (as ratios), and 20-day rolling volatility.

---

## Chart guide

Each entry says what the chart shows, what to look for, and how it helps the analysis.
![image alt](https://github.com/Paras-Gith/Indian_market/blob/fe445816c781e6763d78604659988bd80b992c66/close_price.png)

![image alt](https://github.com/Paras-Gith/Indian_market/blob/fe445816c781e6763d78604659988bd80b992c66/heatmap.png)

### Week 1: `week1_cleaning_eda.ipynb`

**1. Missing-values heatmap**
- *Shows:* every row and column, with missing entries highlighted.
- *Look for:* thin bright lines only in `Volume`, which are the 28 recoded zero-volume days. Every other column is solid.
- *Helps:* proves where the data gaps are, and that the cleaning touched only the unreliable field.

  

**2. Closing price trend**
- *Shows:* the NIFTY 50 close from 2016 to 2026.
- *Look for:* the steady climb to 2019, the sharp drop in March 2020 (low of 7,610), the strong recovery, and the dip in 2026.
- *Helps:* gives the overall picture and the one event that shaped the decade, before any detailed analysis.

**3. Correlation heatmap**
- *Shows:* how strongly each pair of numeric columns moves together (1 means identical, 0 means unrelated).
- *Look for:* Open, High, Low, Close and Adj Close all near 1.00, and Daily Return near 0 with everything.
- *Helps:* shows the price columns are redundant (Close alone is enough), and gives an early hint that returns are hard to predict.

**4. Histogram of daily returns**
- *Shows:* how often each size of daily move occurs.
- *Look for:* a tall peak near zero with long, thin tails on both sides.
- *Helps:* shows that extreme days are rarer but far larger than a normal bell curve would suggest, which motivates the risk analysis in later weeks.

### Week 2: `week2_visualization.ipynb`

**1. Growth trajectory**
- *Shows:* the index level over time.
- *Look for:* a gain from about 8,180 to 23,870 (+192%).
- *Helps:* sets the headline story that the next charts then complicate.

**2. 30-day rolling volatility**
- *Shows:* the typical size of daily moves in each 30-day window.
- *Look for:* a calm band of about 0.5% to 1.7%, and one huge spike of 4.74% on 23 April 2020.
- *Helps:* shows risk is not constant but arrives in short, intense bursts (volatility clustering).

**3. Annual returns bar chart**
- *Shows:* the return for each calendar year, green for gains and red for losses.
- *Look for:* ten of eleven periods positive, and 2020 still up 14.8%. The first and last bars (2016 and 2026) are part-years.
- *Helps:* gives a year-by-year view, and shows why one annual number can hide a turbulent year.

![image alt](https://github.com/Paras-Gith/Indian_market/blob/fe445816c781e6763d78604659988bd80b992c66/annual_returns.png)
  

**4. Drawdown chart**
- *Shows:* how far the index sat below its previous high at every point in time.
- *Look for:* the deepest point of -38.44% on 23 March 2020, recovered by 9 November 2020 (about 7.5 months), and a smaller 2026 decline of up to -15.2%.
- *Helps:* measures the real cost of holding through bad periods, and shows that declines of 10% to 17% are routine.

![image alt](https://github.com/Paras-Gith/Indian_market/blob/fe445816c781e6763d78604659988bd80b992c66/drawdown.png)

**5. Monthly returns heatmap**
- *Shows:* the sum of daily returns for each month of each year (green gains, red losses).
- *Look for:* the extremes (March 2020 at -23.75, April 2020 at +14.49), and otherwise scattered colours with no repeating pattern.
- *Helps:* checks for seasonality. No month is reliably good or bad, so calendar-based rules have no support here.

**6. Actual returns vs normal curve**
- *Shows:* the histogram of daily returns with a normal (bell) curve of the same mean and spread laid over it.
- *Look for:* a taller centre and heavier tails than the curve, with an excess kurtosis of 19.2 and a skew of -1.07.
- *Helps:* shows a normal-distribution model would understate the chance of extreme days, which matters for risk management.

### Week 3: `week3_hypothesis_testing.ipynb`

**1. Daily return distribution with zero line**
- *Shows:* the return histogram with a vertical line at zero.
- *Look for:* the bulk of returns sitting just to the right of the line.
- *Helps:* gives a visual for hypothesis 1, that the average daily return is slightly above zero.

**2. 2020 vs 2017-2019 boxplot**
- *Shows:* the spread of daily returns in the two periods. The box is the middle 50% of days, and dots are extreme days.
- *Look for:* a much taller box and many more outlier dots in 2020.
- *Helps:* visualises hypothesis 2, that volatility was far higher in 2020 (standard deviation 1.97% vs 0.76%).

**3. Returns by weekday boxplot**
- *Shows:* the distribution of returns for each weekday.
- *Look for:* boxes that look almost identical, plus one empty "Saturday" box (a single special Budget session on 1 February 2025).
- *Helps:* visualises hypothesis 3. The similarity matches the finding that there is no reliable weekday effect.

![image alt](https://github.com/Paras-Gith/Indian_market/blob/fe445816c781e6763d78604659988bd80b992c66/box.png)

**4. Volume vs absolute return scatter**
- *Shows:* one dot per day, with trading volume on one axis and the size of that day's price move on the other.
- *Look for:* a loose upward drift, with the largest moves mostly on higher-volume days.
- *Helps:* visualises hypothesis 4, a moderate positive link (r = 0.37), while showing it is far from a tight relationship.

### Week 4: `week4_ml_model.ipynb`

**1. Confusion matrix (logistic regression)**
- *Shows:* a grid of actual outcomes against predicted outcomes (0 = Down, 1 = Up). The diagonal cells are correct predictions.
- *Look for:* the second column holding almost everything (226 and 221), meaning the model predicts "Up" on 447 of 457 days.
- *Helps:* reveals that the 49.2% accuracy comes from defaulting to one class, not from real skill, which accuracy alone would hide.

![image alt](https://github.com/Paras-Gith/Indian_market/blob/fe445816c781e6763d78604659988bd80b992c66/confusion.png)

**2. ROC curve (logistic regression)**
- *Shows:* how well the model's probabilities separate Up days from Down days. The dashed diagonal is random guessing, and a good model bends toward the top-left.
- *Look for:* a curve hugging the diagonal, with an AUC of about 0.49 (chance is 0.50).
- *Helps:* confirms the model has no ranking ability, independent of any chosen threshold.

![image alt](https://github.com/Paras-Gith/Indian_market/blob/fe445816c781e6763d78604659988bd80b992c66/curve.png)

**3. Model comparison bar chart**
- *Shows:* the test accuracy of the logistic regression and the decision tree.
- *Look for:* two nearly equal bars at about 49% and 48%, both under the 50.3% majority-class baseline.
- *Helps:* shows the model type does not matter here. Neither gives a reliable edge.

---

## Key results

| Area | Result |
|---|---|
| Growth | 8,179.95 to 23,865.75, about +192% (roughly 11% a year, excluding dividends) |
| Worst fall | -38.44% on 23 March 2020; peak regained after about 7.5 months |
| Annual returns | 10 of 11 periods positive; 2020 finished +14.8% |
| Daily returns | Mean +0.048%, std 1.02%, skew -1.07, excess kurtosis 19.2 |
| H1: mean daily return is not zero | t = 2.36, p = 0.018; 95% CI 0.008% to 0.089% |
| H2: 2020 more volatile than 2017-2019 | Levene p = 8e-21, Welch p = 3e-10; spread about 2.6 times larger |
| H3: weekday effect | ANOVA p = 0.049 but no significant pair in the Tukey test; Kruskal-Wallis p = 0.39. Not robust |
| H4: volume vs size of moves | r = 0.37, p < 0.001; volume explains about 14% |
| ML: next-day direction | Baseline 50.3%, logistic regression 49.2%, decision tree 48.4% |

---

## Limitations

- One index and one decade, with a single major crash (2020), so conclusions about crash behaviour rest on one event.
- Annual figures for 2016 and 2026 are part-years.
- Yahoo's index volume is not the official NSE traded volume, so volume results are indicative.
- Daily returns are not strictly independent (volatility clusters), so test p-values are approximate. The weekday result is borderline and sensitive to 2020.
- The models use six price-derived features and one split; a 457-day test set cannot separate small differences in accuracy.
- Values are price-index levels and exclude dividends.

This is an academic project, not investment advice.

---

## Author

**Paras Nainwal**, Data Science Apprentice Intern, Yuva Intern
GitHub: [Paras-Gith](https://github.com/Paras-Gith)
