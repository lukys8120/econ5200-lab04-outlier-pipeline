# Outlier Detection on California Housing

**ECON 5200 · Applied Data Analytics in Economics · Lab 04**

## Objective

I fixed three bugs in an outlier detection pipeline, then compared three detection methods on the California Housing data to work out which one a policy team should actually use before modelling.

## Methodology

- Found and fixed three bugs in a broken pipeline: a modified Z-score that used the mean and standard deviation instead of the median and the MAD, a Tukey fence with its `k` left at 1.0 instead of the conventional 1.5, and an Isolation Forest with `contamination` set to 0.5 instead of 0.05.
- Wrote `outlier_detector.py` around an `OutlierDetector` class following the scikit-learn estimator pattern. It takes `method`, `threshold`, `k` and `contamination`, checks all four when you build it and raises `ValueError` on a bad one, then exposes `fit_detect()` and `summary()`.
- Ran all three methods on the same data and compared which rows each one flagged, using set intersections and a Venn diagram. The two univariate methods ran on `MedInc`; the Isolation Forest ran on all nine columns.
- Wrote a method selection memo for a policy team allocating affordable-housing funding, covering which method I would pick, what the alternatives would cost, and whether a flagged row should ever simply be deleted.
- Built an interactive explorer with ipywidgets and plotly that lets you pick a column, move each method's parameter and watch the flagged counts change.

## Key Findings

**The three methods disagree more than they agree.** On 20,640 rows the modified Z-score flagged 400, Tukey flagged 681 and the Isolation Forest flagged 1032. Only 322 rows were flagged by all three. Across the 1336 rows that any method flagged, just 455 — about a third — were flagged by more than one.

**Every row the Z-score flagged, Tukey flagged too.** That is not a coincidence. At the conventional settings Tukey's fence sits at roughly 2.7 standard deviations while the modified Z-score's threshold of 3.5 sits at 3.5, so Tukey is simply the looser of the two rules and the Z-score's flags nest inside it.

**The Isolation Forest is looking at something different.** It flagged 655 rows that neither univariate method touched, because it sees all nine columns at once and can flag a row for an unusual combination — many rooms alongside a low income, say — even when no single column looks strange on its own. The other two methods look at one column at a time and can never find that.

**I recommended Tukey for this use case.** House prices are skewed to the right with a heavy tail, and Tukey's fences are asymmetric, so they follow that shape instead of assuming symmetry around the centre.

**A flagged row is not the same thing as a row to delete.** The block groups with the lowest median incomes get flagged as outliers, and they are exactly the neighbourhoods affordable-housing funding exists for. Removing them would remove the point of the analysis. `MedHouseVal` is also top-coded at 5.00, so the pile-up of block groups sitting at that ceiling is a recording limit rather than an anomaly.

## Repository Contents

| File | Description |
|---|---|
| `lab-ch04-diagnostic.ipynb` | Full lab notebook with rendered output |
| `outlier_detector.py` | `OutlierDetector` class: modified Z-score, Tukey fences, Isolation Forest |

## Data

California Housing, from the 1990 US Census, loaded with `sklearn.datasets.fetch_california_housing`. 20,640 rows and 9 columns. One row is a census block group, not a single house, so every column is already a neighbourhood-level aggregate.
