# Replicating Card (1995): Using Geographic Variation in College Proximity to Estimate the Return to Schooling

ECON 5200 — Midterm Replication Study

## The paper and its headline claim

- **Paper:** Card, D. (1995). "Using Geographic Variation in College Proximity to Estimate the Return to Schooling." In *Aspects of Labour Market Behaviour*, edited by Christofides, Grant, and Swidinsky. University of Toronto Press.
- **Identification strategy:** Instrumental variables (IV).
- **Headline claim:** The paper reports an IV estimate of 0.132 for the return to schooling in Table 4, Row 1.
- **My replication:** My IV estimate is 0.1315, compared with the paper's estimate of 0.1320. The relative difference is approximately −0.38%. My specification is close to, but not identical to, the original specification.

| Estimate | Original paper | Replication | Difference |
|---|---:|---:|---:|
| OLS coefficient | 0.0730 | 0.0747 | +0.0017 |
| OLS SE | 0.0040 | 0.0035 | −0.0005 |
| IV coefficient | 0.1320 | 0.1315 | −0.0005 |
| IV SE | 0.0490 | 0.0550 | +0.0060 |

The replication uses 3,010 observations, and the first-stage F-statistic is 13.3.

## Reproduce the headline table

Python 3.11 or newer. From the repository root:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python reproduce.py
```

The command runs `replication_starter_card1995.ipynb` from top to bottom in a fresh kernel and generates `output/headline_table.csv`. The executed notebook is saved in `output/executed/`.

- **Expected runtime:** Approximately 15–18 seconds on Google Colab, excluding dependency installation.

## Data: source and licence

- **Source:** https://raw.githubusercontent.com/scunning1975/mixtape/master/card.dta
- **Original data:** National Longitudinal Survey of Young Men used by David Card (1995).
- **Included or downloaded:** Downloaded automatically by the notebook rather than included in the repository.

The dataset contains 3,010 observations. Downloading the data directly keeps the data source visible and avoids storing an additional copy in the repository.

## Robustness exercises

The notebook includes the following robustness checks:

- First-stage instrument strength.
- Anderson–Rubin weak-instrument-robust confidence interval.
- Alternative instrument specification using proximity to a two-year college.
- Regional subsample analysis.
- Two-instrument IV estimation and the Sargan overidentification test.
- Placebo test using family structure at age 14.
- Alternative specifications with additional endogenous variables and family background controls.

## Repository map

| Path | What it is |
|---|---|
| `README.md` | Project description, replication results, and instructions |
| `replication_starter_card1995.ipynb` | Baseline replication and robustness exercises |
| `requirements.txt` | Pinned package versions |
| `AI_Appendix.md` | Documentation of AI use and P.R.I.M.E. entries |

## Use of AI

[Complete this section with an accurate description of which sections involved AI assistance.]

The details of AI assistance are documented in [AI_Appendix.md](AI_Appendix.md).
