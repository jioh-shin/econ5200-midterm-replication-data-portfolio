# AI Appendix — Replication Study

> **Before submission:** Confirm that the statement is accurate. This appendix reflects the six visible AI requests about Section 3 (including the follow-up request for the two-instrument test). If you used AI in Sections 4–6 or elsewhere, add those entries as well. Text in [square brackets] requires your own confirmation and must not remain as an unverified claim. The original prompts were not written in explicit P.R.I.M.E. labels; the mapping below describes their content without pretending those labels were in the original prompts.

## Entry 1: Section 3, exercise 1 — Regional subsample analysis

### P.R.I.M.E. mapping of the original prompt

- **[Prep]** No specific role was assigned in the original prompt.
- **[Request]** Card (1995) replication using `card.dta`, 3,010 observations, `educ` and `nearc4`, restricting the sample to `south == 0`.
- **[Iterate]** Use existing `card` and `CONTROLS`; define `MY_SPEC` and two-sentence `MY_SPEC_WHY`; keep code simple; do not invent results.
- **[Mechanism Check]** Explain what each setting does and why excluding the South is a robustness check.
- **[Evaluate]** Assess whether the IV return-to-schooling estimate depends on regional sample composition.

### Verification log

**1. Prompt, pasted in full**

I'm working on a replication project based on David Card's (1995) paper, "Using Geographic Variation in College Proximity to Estimate the Return to Schooling."

I am using Python in Google Colab with the card.dta dataset (3,010 observations).

In Section 2, I estimated the return to schooling using OLS and IV (2SLS). My results were:
- OLS coefficient on educ: 0.0747 (SE = 0.0035)
- IV coefficient on educ: 0.1315 (SE = 0.0550)
- Instrument: nearc4
- First-stage F-statistic: 13.3

My baseline controls are:
CONTROLS = ["exper", "expersq", "black", "south", "smsa", "smsa66", "reg662", "reg663", "reg664", "reg665", "reg666", "reg667", "reg668", "reg669"]

The data is already loaded as a pandas DataFrame named card.

For Section 3, I need to conduct robustness checks by changing at least one modeling choice. The starter notebook asks me to define:

MY_SPEC = {"controls": [...], "instrument": "nearc4" or "nearc2", "sample": None or a pandas query string}
MY_SPEC_WHY = two sentences explaining the choice.

I want to exclude observations from the South (south == 0) to examine whether the IV estimate is sensitive to regional differences.

Can you help me write simple Python code for MY_SPEC and MY_SPEC_WHY using my existing CONTROLS variable? Please briefly explain the code and why this is a reasonable robustness check. Keep the code simple and do not generate artificial results.

**2. AI-generated response, pasted in full**

## 1. Python Code for Section 3

You can keep your original controls and instrument while restricting the sample to individuals who do not live in the South.

```python
MY_SPEC = {
    "controls": CONTROLS,
    "instrument": "nearc4",
    "sample": "south == 0"
}

MY_SPEC_WHY = (
    "I exclude observations from the South to examine whether regional differences affect the estimated return to schooling. "
    "I keep the original controls and instrument to make the results comparable with the baseline model."
)
```

## 2. Explanation

| Component | Meaning |
|---|---|
| `"controls": CONTROLS` | Keeps your original control variables |
| `"instrument": "nearc4"` | Uses proximity to a four-year college as the instrument |
| `"sample": "south == 0"` | Restricts the sample to non-Southern observations |
| `MY_SPEC_WHY` | Explains the motivation for the robustness check |

## 3. Why is this a reasonable robustness check?

Regional differences may affect college access, educational attainment, and wages. By excluding Southern observations, you can examine whether your IV estimate depends heavily on including individuals from that region.

**How to interpret your results:**

- **If the IV coefficient remains similar to 0.1315:** This provides some evidence that your baseline estimate is not highly sensitive to excluding Southern observations.
- **If the IV coefficient changes substantially:** This suggests that the estimated return to schooling may be sensitive to regional sample composition.
- **If the standard error increases:** This may reflect the smaller sample size or weaker instrument relevance.

**Important:** Check the first-stage F-statistic again. Your baseline F-statistic was 13.3, but restricting the sample could change the strength of `nearc4`.

One technical detail: Because `south` is included in `CONTROLS` but equals zero for every observation in the restricted sample, it becomes a constant column. Most regression implementations handle this automatically, but if your notebook reports a rank-deficiency or collinearity error, use:

```python
"controls": [c for c in CONTROLS if c != "south"]
```

This removes only the redundant `south` variable without changing the substantive control specification.

**3. What I changed, and why**

I kept the short specification code, but rewrote the explanatory prose in my own words for the final report. I did not modify the code because the dictionary entries directly matched the notebook fields and I did not identify a useful change. [Confirm whether the optional suggestion to drop `south` was used.]

**4. What I verified, and against what**

Check that `south == 0` selects non-South observations and compare the resulting N, IV coefficient and first-stage F with my own notebook output. Record the actual notebook cell/output used as evidence; do not claim a check not performed.

---

## Entry 2: Section 3, exercise 2 — Family background controls

### P.R.I.M.E. mapping of the original prompt

- **[Prep]** No specific role was assigned.
- **[Request]** Test how adding `fatheduc`, `motheduc`, and `momdad14` changes the IV estimate for `educ` instrumented by `nearc4`.
- **[Iterate]** Use `card`, `CONTROLS`, pandas, statsmodels and linearmodels; report IV coefficient, SE and N; no invented results.
- **[Mechanism Check]** Explain why full sample versus complete cases differs from holding the complete-case sample fixed while adding controls.
- **[Evaluate]** Separate the impact of missing-data sample selection from family-background adjustment on the return-to-schooling estimate.

### Verification log

**1. Prompt, pasted in full**

My baseline IV model uses educ as the endogenous variable and nearc4 as its instrument.

CONTROLS = ["exper", "expersq", "black", "south", "smsa", "smsa66", "reg662", "reg663", "reg664", "reg665", "reg666", "reg667", "reg668", "reg669"]

Baseline IV: 0.1315 (SE 0.0550), N = 3010.

I have already run a non-South subsample analysis, which gave IV = 0.0555 (SE 0.0566), first-stage F = 11.5, N = 1795.

I now want to test whether adding family background controls changes the IV estimate. The dataset contains fatheduc, motheduc, and momdad14, some of which have missing values.

Please write simple Python code using pandas, statsmodels, and linearmodels to compare:
1. Baseline IV on the full sample.
2. Baseline IV on observations with complete family background data.
3. IV with family background controls on the same complete-case sample.

Print the IV coefficients, standard errors, and sample sizes. Explain briefly how this comparison separates sample selection from the effect of adding controls. Do not invent numerical results.

**2. AI-generated response, pasted in full**

The key is to estimate **three IV models**, keeping `nearc4` as the instrument for `educ`. Models 2 and 3 must use exactly the same observations to isolate the effect of adding family background controls.

## 1. Python code (Google Colab)

This code assumes `card` and `CONTROLS` are already defined.

```python
import pandas as pd
import statsmodels.api as sm
from linearmodels.iv import IV2SLS

FAMILY = ["fatheduc", "motheduc", "momdad14"]

# Function to estimate IV
def run_iv(df, controls):
    y = df["lwage"]
    X = sm.add_constant(df[controls], has_constant="add")
    endog = df["educ"]
    instrument = df[["nearc4"]]

    result = IV2SLS(y, X, endog, instrument).fit(
        cov_type="robust"
    )

    return {
        "IV coefficient": result.params["educ"],
        "SE": result.std_errors["educ"],
        "N": result.nobs
    }

# Model 1: Baseline IV, full sample
baseline_data = card.dropna(
    subset=["lwage", "educ", "nearc4"] + CONTROLS
)
model1 = run_iv(baseline_data, CONTROLS)

# Create complete-case sample
complete_data = card.dropna(
    subset=["lwage", "educ", "nearc4"] + CONTROLS + FAMILY
)

# Model 2: Baseline IV, complete-case sample
model2 = run_iv(complete_data, CONTROLS)

# Model 3: IV with family background controls
model3 = run_iv(complete_data, CONTROLS + FAMILY)

# Compare results
results = pd.DataFrame(
    [model1, model2, model3],
    index=[
        "1. Baseline (full sample)",
        "2. Baseline (complete cases)",
        "3. Family controls (complete cases)"
    ]
)

print(results.round(4))
```

**Note:** The code uses heteroskedasticity-robust standard errors (`cov_type="robust"`). If your original Section 2 used a different covariance estimator, use the same setting for a directly comparable standard error.

## 2. How to interpret the comparison

| Comparison | What it measures |
|---|---|
| Model 1 vs. Model 2 | Effect of restricting the sample to complete cases |
| Model 2 vs. Model 3 | Effect of adding family background controls, holding the sample fixed |
| Model 1 vs. Model 3 | Combined effect of sample restriction and additional controls |

For example, if the IV coefficient decreases from Model 1 to Model 2, this suggests that excluding observations with missing family data changes the estimated coefficient, even without including family controls.

If the coefficient changes between Models 2 and 3, the difference is attributable to adding the family variables to the specification rather than changing the estimation sample.

## 3. Suggested interpretation for your report

You can adapt the following after running the code:

> I compare three IV specifications to distinguish changes due to missing data from changes due to additional family background controls. First, I estimate the baseline IV model using the full sample. Second, I estimate the same model on observations with complete family background information. Third, I add father's education, mother's education, and family structure to the complete-case specification. Comparing the second and third models isolates the change in the IV estimate associated with adding family background controls while holding the sample constant.

One caution: This comparison separates the **sample-composition effect** from the **specification effect** in the estimates, but it does not establish that either difference is causal. Missing family data may not be random, and the IV assumptions still need to hold in the restricted sample.

**3. What I changed, and why**

I retained the code and changed the report prose to my own wording. The code was already a simple three-model comparison, and I did not know what meaningful code change would improve it. [Revise if any code was changed during execution.]

**4. What I verified, and against what**

Compare the three-model printout with the notebook output (full N=3,010, complete-case N=2,220; IV estimates 0.1315, 0.0799 and 0.0829). Check robust SE settings before treating 0.0540 and 0.0550 as interchangeable. Document what was actually checked.

---

## Entry 3: Section 3, exercise 3 — first request — Two instruments and over-identification (initial response)

### P.R.I.M.E. mapping of the original prompt

- **[Prep]** No specific role was assigned.
- **[Request]** Estimate the return to schooling using `nearc2` and `nearc4` as instruments for `educ`.
- **[Iterate]** Use linearmodels `IV2SLS`, existing controls, print coefficient, SE, sample size and Sargan results; compare with baseline.
- **[Mechanism Check]** Explain what an over-identification test does and why common-sample comparison matters.
- **[Evaluate]** Evaluate sensitivity of the Card result to the instrument set and whether instrument restrictions appear inconsistent with the data.

### Verification log

**1. Prompt, pasted in full**

I want to perform another robustness check using both nearc2 and nearc4 as instruments for educ.

Please provide simple Python code using linearmodels IV2SLS to:
1. Estimate the IV coefficient with both instruments.
2. Print its standard error and sample size.
3. Report an over-identification test such as the Sargan test.
4. Compare the result with my baseline IV estimate.

**2. AI-generated response, pasted in full**

## 1. Python code: IV with two instruments

This code uses both `nearc2` and `nearc4` as instruments for `educ`, keeping your original controls.

```python
import pandas as pd
import statsmodels.api as sm
from linearmodels.iv import IV2SLS

# Keep observations with complete data
variables = ["lwage", "educ", "nearc2", "nearc4"] + CONTROLS
df = card.dropna(subset=variables).copy()

# Define variables
y = df["lwage"]
X = sm.add_constant(df[CONTROLS])
endog = df["educ"]
instruments = df[["nearc2", "nearc4"]]

# Estimate IV with two instruments
model = IV2SLS(y, X, endog, instruments).fit(
    cov_type="robust"
)

# Print results
print("IV coefficient:", round(model.params["educ"], 4))
print("Standard error:", round(model.std_errors["educ"], 4))
print("Sample size:", model.nobs)

# Over-identification test
print("\nSargan test:")
print(model.sargan)

# Compare with baseline
print("\nBaseline IV: 0.1315 (SE = 0.0550)")
print("Two-instrument IV:", round(model.params["educ"], 4))
print("Difference:", round(model.params["educ"] - 0.1315, 4))
```

## 2. How to interpret the results

| Result | Interpretation |
|---|---|
| IV coefficient | Estimated return to schooling using both instruments |
| Standard error | Statistical uncertainty of the IV estimate |
| Sample size | Number of observations included |
| Sargan p-value > 0.05 | Fail to reject the over-identifying restrictions |
| Sargan p-value < 0.05 | Evidence against the joint validity of the instruments, assuming the test's conditions hold |

**Important:** The Sargan test is a conventional over-identification test that assumes homoskedastic errors. Because the model uses `cov_type="robust"`, you can also examine the heteroskedasticity-robust Wooldridge over-identification test:

```python
print(model.wooldridge_overid)
```

Neither test proves instrument validity when its p-value is large.

## 3. Comparison with the baseline

Your original estimates were:

- **Baseline IV (`nearc4`):** 0.1315 (SE = 0.0550)
- **Two-instrument IV (`nearc2`, `nearc4`):** Calculated by the code above

If the two-instrument estimate is similar to the baseline, that supports the conclusion that the estimated return to schooling is relatively insensitive to adding `nearc2` as an instrument.

If the estimate changes substantially, it suggests sensitivity to instrument choice, although this alone does not identify the cause.

For a strictly comparable analysis, re-estimate the baseline using the same complete-case sample if including `nearc2` reduces the number of observations.

**3. What I changed, and why**

I kept the basic code approach. I then made a follow-up request for more explicit output of the Sargan statistic and p-value. The written interpretation was revised into my own words.

**4. What I verified, and against what**

Use notebook output to check two-instrument coefficient, SE and N. See the next entry for the revised requested code and explicit test output. Do not represent the initial code as the final script if the follow-up version was actually used.

---

## Entry 4: Section 3, exercise 3 — follow-up — Two instruments and over-identification (revised response)

### P.R.I.M.E. mapping of the original prompt

- **[Prep]** No specific role was assigned.
- **[Request]** Two instruments for `educ`, `nearc2` and `nearc4`, with the existing Card replication controls.
- **[Iterate]** Use linearmodels, report the coefficient, robust SE, N, Sargan statistic and p-value; no invented results.
- **[Mechanism Check]** Explain test null and the limits of non-rejection.
- **[Evaluate]** Determine whether the larger instrument set changes the IV estimate and whether over-identifying restrictions are rejected.

### Verification log

**1. Prompt, pasted in full**

Could you help me write simple Python code using linearmodels IV2SLS to:
1. Estimate the IV model using both nearc2 and nearc4 as instruments for educ.
2. Print the IV coefficient, standard error, and sample size.
3. Perform an over-identification test, such as the Sargan test, and report its statistic and p-value.
4. Compare the new estimate with my baseline IV result.

Please also explain briefly how to interpret the over-identification test and why failing to reject its null hypothesis does not prove that the instruments are valid.

Keep the code simple and use my existing card and CONTROLS variables. Do not generate artificial data or invent results.

**2. AI-generated response, pasted in full**

## 1. Python code

This code uses your existing `card` DataFrame and `CONTROLS` list.

```python
import statsmodels.api as sm
from linearmodels.iv import IV2SLS

# Remove observations with missing values
variables = ["lwage", "educ", "nearc2", "nearc4"] + CONTROLS
df = card.dropna(subset=variables).copy()

# Define variables
y = df["lwage"]
X = sm.add_constant(df[CONTROLS])
endog = df["educ"]
Z = df[["nearc2", "nearc4"]]

# Estimate IV with two instruments
model = IV2SLS(y, X, endog, Z).fit(cov_type="robust")

# Print IV results
print("IV coefficient:", round(model.params["educ"], 4))
print("Standard error:", round(model.std_errors["educ"], 4))
print("Sample size:", model.nobs)

# Sargan over-identification test
print("\nSargan test:")
print("Statistic:", round(model.sargan.stat, 4))
print("P-value:", round(model.sargan.pval, 4))

# Compare with baseline IV
baseline_iv = 0.1315

print("\nBaseline IV:", baseline_iv)
print("Two-instrument IV:", round(model.params["educ"], 4))
print("Difference:", round(model.params["educ"] - baseline_iv, 4))

# Robust over-identification test
print("\nWooldridge over-identification test:")
print(model.wooldridge_overid)
```

## 2. Interpreting the over-identification test

The Sargan test examines whether the over-identifying restrictions are consistent with the data.

- **Null hypothesis (H₀):** The instruments satisfy the over-identifying restrictions, conditional on the model assumptions.
- **p-value < 0.05:** Reject H₀. This suggests that at least one instrument may be invalid or that the model is misspecified.
- **p-value ≥ 0.05:** Fail to reject H₀. There is insufficient evidence against the over-identifying restrictions.

**Why doesn't a large p-value prove instrument validity?**

The test examines whether the instruments produce moment conditions consistent with one another. It cannot establish that both instruments are exogenous. For example, if both `nearc2` and `nearc4` violate the exclusion restriction in similar ways, the test may fail to detect the problem.

Also, the Sargan test assumes homoskedasticity. The Wooldridge over-identification test is more appropriate when allowing for heteroskedasticity.

## 3. Comparing the estimates

Your baseline IV estimate is **0.1315 (SE = 0.0550, N = 3,010)**.

After running the code:

- If the two-instrument coefficient is similar to 0.1315, the estimated return to schooling appears relatively insensitive to adding `nearc2`.
- If the coefficient differs substantially, the estimate may be sensitive to instrument selection.
- Compare standard errors to determine whether adding the second instrument changes estimation precision.

For a fair comparison, both models should use the same observations. If the two-instrument model has fewer than 3,010 observations, re-estimate the baseline using the restricted sample.

**Suggested report interpretation:**

> I estimate an alternative IV specification using both proximity to a two-year college (`nearc2`) and proximity to a four-year college (`nearc4`) as instruments for education. I compare the resulting coefficient with the baseline IV estimate of 0.1315. I also conduct an over-identification test to assess whether the additional moment restriction is consistent with the data. Failure to reject the null hypothesis does not establish instrument validity, because the test may have limited power and cannot rule out violations shared by both instruments.

**3. What I changed, and why**

I asked a clearer follow-up question to make the code print the statistic and p-value separately and to include a robust test. I kept the resulting simple code and rewrote the report discussion in my own words.

**4. What I verified, and against what**

Compare own notebook output: IV 0.1571 (SE 0.0524), N=3,010; Sargan 1.2482 (p=0.2639), Wooldridge p=0.2600. Check the printed result; these values are student-provided and not independently verified here.

---

## Entry 5: Section 3, exercise 4 — Placebo / covariate balance test

### P.R.I.M.E. mapping of the original prompt

- **[Prep]** No specific role was assigned.
- **[Request]** Check association between `nearc4` and `momdad14` with Card data.
- **[Iterate]** Use statsmodels OLS, HC3 SEs, predetermined demographic/geographic controls; exclude education and experience; print estimates and confidence interval.
- **[Mechanism Check]** Explain control choice and why this is not a direct exclusion-restriction test.
- **[Evaluate]** Assess whether the instrument is correlated with an observed pre-college family characteristic.

### Verification log

**1. Prompt, pasted in full**

Could you help me write simple Python code using statsmodels OLS with HC3 robust standard errors?

Please:

1. Test whether `nearc4` is associated with `momdad14`, controlling for appropriate demographic and geographic variables.
2. Avoid including variables that could be affected by college proximity, such as education and experience, as controls.
3. Print the coefficient, standard error, p-value, 95% confidence interval, and sample size.
4. Briefly explain what the results imply about the plausibility of the IV exclusion restriction.

**2. AI-generated response, pasted in full**

## 1. Python code: OLS with HC3 robust standard errors

This regression tests whether proximity to a four-year college (`nearc4`) is associated with family structure at age 14 (`momdad14`), controlling for demographic and geographic characteristics.

```python
import statsmodels.api as sm

# Demographic and geographic controls
controls = [
    "black", "south", "smsa66",
    "reg662", "reg663", "reg664", "reg665",
    "reg666", "reg667", "reg668", "reg669"
]

# Remove missing observations
variables = ["momdad14", "nearc4"] + controls
df = card.dropna(subset=variables).copy()

# Define variables
y = df["momdad14"]
X = sm.add_constant(df[["nearc4"] + controls])

# OLS with HC3 robust standard errors
model = sm.OLS(y, X).fit(cov_type="HC3")

# Print results
print("Coefficient:", round(model.params["nearc4"], 4))
print("Standard error:", round(model.bse["nearc4"], 4))
print("P-value:", round(model.pvalues["nearc4"], 4))
print("95% CI:", model.conf_int().loc["nearc4"].round(4).tolist())
print("Sample size:", int(model.nobs))
```

**Why these controls?** The model includes race and geographic variables, including region and urban residence in 1966. It excludes `educ`, `exper`, and `expersq`, which could be influenced by college proximity. It also excludes current `smsa` to avoid conditioning on potentially subsequent residential outcomes.

## 2. How to interpret the results

| Result | Interpretation |
|---|---|
| Coefficient on `nearc4` | Conditional association between college proximity and family structure |
| Standard error | Uncertainty of the estimated coefficient |
| p-value < 0.05 | Statistically significant association at the 5% level |
| p-value ≥ 0.05 | Insufficient evidence of an association at the 5% level |
| 95% confidence interval | Range of parameter values consistent with the estimate under the model's assumptions |

If `momdad14` is a binary indicator, the coefficient can be interpreted as a difference in probability. For example, a coefficient of 0.05 would represent a 5-percentage-point difference, holding the included controls constant. This is an illustration, not an estimated result.

## 3. Implications for the IV exclusion restriction

The IV exclusion restriction requires `nearc4` to affect wages only through its effect on education.

- **Significant association:** If `nearc4` predicts `momdad14` after controlling for demographics and geography, this raises concerns about differences in family background between individuals living near and far from colleges. Such differences could undermine the plausibility of the IV identification assumptions.
- **No significant association:** This provides no statistical evidence of imbalance in this particular family-background characteristic, but it does not prove that `nearc4` is valid.

**Important distinction:** This is a covariate balance or falsification check, not a direct test of the exclusion restriction. An association with `momdad14` may suggest a violation of instrument exogeneity, but it does not establish a direct effect of college proximity on wages.

The interpretation also depends on whether the included geographic variables are appropriate pre-treatment controls.

## 4. Suggested report explanation

> I examine whether college proximity is associated with family background by regressing `momdad14` on `nearc4`, controlling for race and geographic characteristics. I use HC3 robust standard errors and exclude education and experience because these may be affected by college proximity. A statistically significant association would raise concerns that the instrument is correlated with predetermined family characteristics, potentially weakening the credibility of the IV identification assumptions. However, a statistically insignificant association would not establish instrument validity, because other unobserved confounding factors or violations of the exclusion restriction may remain.

**3. What I changed, and why**

I kept the short OLS code and rewrote the written explanation in my own language. I did not change the code because it already used the requested variables, estimator and outputs. [Confirm whether your actual notebook included the same control list.]

**4. What I verified, and against what**

Compare the own Colab regression output with coefficient −0.0198, SE 0.0180, p=0.2698, 95% CI [−0.0550, 0.0154], N=3,010. Check that no `educ`, `exper`, or `expersq` is included. Only claim checks you actually made.

---

## Entry 6: Section 3, exercises 1–4 — Writing the four robustness subsections

### P.R.I.M.E. mapping of the original prompt

- **[Prep]** No explicit role was assigned; the requested style was that of a graduate student report.
- **[Request]** Use the supplied Card replication outputs for four robustness exercises and write Section 3.1–3.4.
- **[Iterate]** Use only supplied estimates; exactly three headings per exercise; 100–140 words; no invented South-only regression; plain English.
- **[Mechanism Check]** Explain why each comparison is informative, including missing-data selection and the limits of diagnostic tests.
- **[Evaluate]** Evaluate the credibility and stability of Card’s main IV finding in each exercise.

### Verification log

**1. Prompt, pasted in full**

My baseline estimates are:
- OLS: 0.0747 (SE = 0.0035)
- IV: 0.1315 (SE = 0.0550)
- Sample size: 3,010
- Endogenous variable: educ
- Instrument: nearc4
- Controls: exper, expersq, black, south, smsa, smsa66, and reg662–reg669

My four robustness exercises are:

1. Regional Subsample Analysis
- Non-South IV coefficient: 0.0555
- Standard error: 0.0566
- First-stage F-statistic: 11.5
- Sample size: 1,795
- The full-sample baseline IV coefficient is 0.1315.
- I did not run a separate South-only regression in this exercise.

2. Family Background Controls
- Full-sample baseline IV: 0.1315 (robust SE = 0.0540), N = 3,010
- Complete-case baseline IV: 0.0799 (SE = 0.0714), N = 2,220
- Complete-case IV with family background controls: 0.0829 (SE = 0.0735), N = 2,220
- Family controls include fatheduc, motheduc, and momdad14.

3. Alternative Instruments and Over-identification
- IV with nearc2 and nearc4: 0.1571 (robust SE = 0.0524), N = 3,010
- Sargan statistic: 1.2482
- Sargan p-value: 0.2639
- Wooldridge over-identification p-value: 0.2600
- Baseline IV with nearc4 only: 0.1315

4. Placebo Test
- Outcome: momdad14
- Explanatory variable: nearc4
- Estimation: OLS with HC3 robust standard errors
- Coefficient: -0.0198
- Standard error: 0.0180
- p-value: 0.2698
- 95% confidence interval: [-0.0550, 0.0154]
- Sample size: 3,010
- The placebo regression controls for demographic and geographic characteristics but excludes education and experience.

Please write four subsections, numbered 3.1 to 3.4.

For each subsection, use exactly these three headings:
- What I did
- What I found
- What it implies

Please use simple, natural academic English, like a graduate student writing their own report. Keep each subsection around 100–140 words.

Important:
- Use only the results I provided.
- Do not invent numbers or analyses.
- Do not claim that the South-only regression was performed.
- Distinguish the effects of sample selection from adding family controls.
- Do not claim that an insignificant over-identification or placebo test proves instrument validity.
- Explain what each result means for the credibility of Card's main finding.
- Avoid overly complicated vocabulary.

**2. AI-generated response, pasted in full**

## 3.1 Regional Subsample Analysis

### What I did
I restricted the sample to individuals living outside the South to see whether the IV estimate changes across regions. I kept the same instrument, `nearc4`, and the original controls.

### What I found
The IV coefficient decreased from 0.1315 in the full sample to 0.0555 in the non-South sample. The standard error was 0.0566, and the sample size decreased from 3,010 to 1,795. The first-stage F-statistic was 11.5.

### What it implies
The smaller coefficient suggests that the estimated return to schooling is sensitive to the sample used. The non-South estimate is also less precise relative to its size. This raises some concerns about the stability of Card's main finding across regions. However, the change may also reflect differences in sample composition.

## 3.2 Family Background Controls

### What I did
I added three family background variables: father's education (`fatheduc`), mother's education (`motheduc`), and family structure (`momdad14`). I first estimated the baseline model using only observations with complete family data, then added the family controls to the same sample.

### What I found
The full-sample IV estimate was 0.1315 (SE = 0.0540), with 3,010 observations. After restricting the sample to 2,220 observations, the estimate decreased to 0.0799 (SE = 0.0714). Adding family controls slightly increased the coefficient to 0.0829 (SE = 0.0735).

### What it implies
Most of the change came from restricting the sample rather than adding family controls. This suggests that missing data and sample selection matter more for the estimate than these additional controls. Card's finding remains positive, but its size appears sensitive to the estimation sample.

## 3.3 Alternative Instruments and Over-identification

### What I did
I estimated another IV model using both `nearc2` and `nearc4` as instruments for education. I also performed Sargan and Wooldridge over-identification tests to examine whether the instruments are consistent with the model assumptions.

### What I found
The two-instrument IV coefficient was 0.1571 (SE = 0.0524), compared with the baseline estimate of 0.1315. The sample size remained 3,010. The Sargan statistic was 1.2482, with a p-value of 0.2639. The Wooldridge test gave a p-value of 0.2600.

### What it implies
The estimate remains positive and becomes slightly larger when both instruments are used. Neither test rejects the over-identifying restrictions at the 5% level. This is consistent with Card's main finding, but it does not prove that the instruments are valid.

## 3.4 Placebo Test

### What I did
I tested whether `nearc4` is associated with family structure at age 14 (`momdad14`). I used OLS with HC3 robust standard errors and controlled for demographic and geographic characteristics. I excluded education and experience because they could be affected by college proximity.

### What I found
The coefficient on `nearc4` was -0.0198, with a standard error of 0.0180 and a p-value of 0.2698. The 95% confidence interval was [-0.0550, 0.0154], and the sample size was 3,010.

### What it implies
The relationship was not statistically significant at the 5% level. This means I did not find clear evidence that college proximity is related to this family background variable. This result does not raise an additional concern about Card's instrument, but it cannot prove that the exclusion restriction holds.

**3. What I changed, and why**

I rewrote the AI-generated paragraphs using my own vocabulary and sentence structure before submission. I did not use its prose unchanged. [Add one exact before/after sentence from your submitted report to document this editing, since the final rewritten text was not provided in this conversation.]

**4. What I verified, and against what**

Cross-check every reported coefficient, SE, p-value and sample size against the Colab output. Check that the final report contains only the analyses actually executed. These are suggested verification steps, not claims that they have already been performed.

---

## Entry 7: Section 6 — Configuring the data downloader

### P.R.I.M.E. mapping (retrospective)

- **[Prep]** Use the instructor's existing replication template rather than write a new downloader.
- **[Request]** Identify the setting needed for Card (1995) and explain how to test it.
- **[Iterate]** Keep the template code and select `card`; troubleshoot the repository path when the file cannot be found.
- **[Mechanism Check]** Understand that `download_data.py` downloads `card.dta` into `data/`, while the notebook can also read the public URL directly.
- **[Evaluate]** Check that the download command completes and the file exists.

### Verification log

**1. Request summary (English paraphrase, not the original prompt)**

I asked whether setting `MY_PAPER = "card"` was sufficient to configure the instructor’s existing data download script.

**2. AI assistance (summary)**

The AI recommended retaining the instructor’s downloader and changing only `MY_PAPER` to `"card"`. It also explained how to test the download in Colab.

```python
MY_PAPER = "card"
```

**3. What I changed, and why**

I changed `MY_PAPER` from an empty string to `"card"` in the instructor's downloader. I retained the rest of the template code without rewriting it. The change selects the dataset for my paper. This was a repository configuration change, not a new estimation procedure.

**4. What I verified, and against what**

In the Colab test, `python download_data.py` printed the Card paper name and downloaded `data/card.dta`. The subsequent `ls -lh data/` output showed `card.dta` (459 KB). The screenshot confirms that the downloader ran successfully. The notebook's direct URL-loading method is separate; this test does not establish that the notebook uses the downloaded local copy.

---

## Entry 8: Section 6 — GitHub paths and download troubleshooting

### P.R.I.M.E. mapping (retrospective)

- **[Prep]** Use a separate Colab notebook to test the GitHub repository.
- **[Request]** Explain why the download script could not be found after cloning.
- **[Iterate]** Inspect the directory, compare the two repository names and clone the repository containing the modified script.
- **[Mechanism Check]** Understand the difference between the remote repository, the local checkout and the current working directory.
- **[Evaluate]** Verify the script can be run from the intended checkout.

### Verification log

**1. Request summary (English paraphrase, not the original prompt)**

I asked why the downloader could not be found in the cloned repository even after running `git pull`. I supplied the actual directory listing and later a screenshot of the repository containing the script.

**2. AI assistance (summary)**

The AI identified the repository mismatch and provided commands to clone the intended repository, move into it and run the downloader. The original commands and directory listing are retained in the transcript excerpts below.

**3. What I changed, and why**

I followed the suggested commands to test the repository that actually contained the downloader. I did not rewrite these commands or change the econometric analysis. I also used `%cd` to return to the repository when `git pull` failed outside a Git working directory.

**4. What I verified, and against what**

The first checkout did not contain `download_data.py`. The second checkout contained it and successfully downloaded the dataset. This established that the earlier failure was related to the repository location. The repository used for testing differs from the original Canvas submission link; acceptance of a changed link is not established by this test.

---

## Entry 9: Section 6 — Configuring the automatic notebook runner

### P.R.I.M.E. mapping (retrospective)

- **[Prep]** Keep the instructor's `reproduce.py` template.
- **[Request]** Adapt the runner to a project containing one notebook at the repository root.
- **[Iterate]** Change the notebook search path, install the listed dependencies and run the script.
- **[Mechanism Check]** Understand that the runner starts a fresh kernel and executes notebooks from top to bottom.
- **[Evaluate]** Check whether the notebook finishes and produces the expected output.

### Verification log

**1. Request summary (English paraphrase, not the original prompt)**

I asked how to adapt the instructor’s automatic notebook runner to my repository, where the analysis notebook is stored at the root rather than in the `notebooks/` directory.

**2. AI assistance (summary)**

The AI recommended changing the notebook search path from `ROOT / "notebooks"` to `ROOT` and testing the runner using the pinned dependencies.

```python
NOTEBOOKS = ROOT
```

**3. What I changed, and why**

I changed the template's notebook search directory to the repository root because my analysis is in one notebook located there. I did not independently rewrite the runner or alter its execution mechanism. I used the suggested commands to install dependencies and test it.

**4. What I verified, and against what**

The runner located my notebook and began executing it. Its first attempt stopped at a `NameError` for `MY_SPEC`; subsequent attempts using the corrected notebook completed. The existing runner does not itself explicitly call the downloader in the supplied script, so I do not claim that the path change alone connected the downloader to the runner.

---

## Entry 10: Sections 3 and 6 — Diagnosing a fresh-kernel error

### P.R.I.M.E. mapping (retrospective)

- **[Prep]** Diagnose an actual execution traceback rather than assume saved notebook outputs demonstrate reproducibility.
- **[Request]** Identify the cause of `NameError: name 'MY_SPEC' is not defined`.
- **[Iterate]** Check the notebook version and keep the intended final notebook in the repository.
- **[Mechanism Check]** Understand that variables from earlier interactive sessions may not exist in a fresh kernel.
- **[Evaluate]** Run the correct notebook from start to finish.

### Verification log

**1. Request summary (English paraphrase, not the original prompt)**

I requested help diagnosing a fresh-kernel `NameError` involving `MY_SPEC`. I then explained that I had uploaded the wrong notebook and asked how to remove the duplicate file.

**2. AI assistance (summary)**

The AI identified the missing `MY_SPEC` variable and proposed an alternative-instrument specification. After I explained that the wrong notebook had been uploaded, it advised removing the duplicate and rerunning the intended notebook. The proposed code is retained in the transcript excerpts; its use in the final notebook is not established here.

**3. What I changed, and why**

I identified that I had uploaded the wrong notebook, replaced it with the intended notebook and removed the duplicate. I did not need to claim that I independently rewrote the AI's proposed specification. Whether that particular proposed code was incorporated into the final notebook should be checked against the actual file; successful execution alone does not establish its use.

**4. What I verified, and against what**

After `git pull`, the file listing showed one root-level analysis notebook. The next `python reproduce.py` run reported `All 1 notebooks ran in 14 s.` It also reported that `output/headline_table.csv` was missing. This verified that the earlier execution blocker was no longer present in the tested version, but that CSV export still needed to be configured.

---

## Entry 11: Section 6 — Exporting the existing headline comparison table

### P.R.I.M.E. mapping (retrospective)

- **[Prep]** Reuse the existing `comparison` DataFrame; do not recalculate or replace the Section 2 estimates.
- **[Request]** Identify where to add the export code and whether to edit GitHub or Colab.
- **[Iterate]** Add a new output-saving cell to the analysis notebook and save the updated notebook to GitHub.
- **[Mechanism Check]** Understand that displaying a DataFrame and writing it to a CSV file are different operations.
- **[Evaluate]** Run the notebook through `reproduce.py` and check that the headline table is generated and printed.

### Verification log

**1. Request summary (English paraphrase, not the original prompt)**

I asked where to insert the CSV export code in my existing analysis notebook and whether the change should be made in Google Colab or directly on GitHub.

**2. AI assistance (summary)**

The AI supplied a separate output-saving cell that exports the existing comparison DataFrame without changing the regression estimates.

```python
from pathlib import Path

Path("output").mkdir(exist_ok=True)
comparison.to_csv("output/headline_table.csv", index=False)
print("Headline table saved.")
```

**3. What I changed, and why**

I used the suggested export code without rewriting it. I added it as a separate Section 6 cell to save the already calculated comparison table. The code creates an output directory when necessary and writes the existing DataFrame to a CSV file. It does not change the OLS or IV specification.

**4. What I verified, and against what**

The next test reported `All 1 notebooks ran in 18 s.` and printed the headline table. The screenshot showed the following rounded values:

| Estimate | Paper | Replication | Gap |
|---|---:|---:|---:|
| OLS coefficient | 0.073 | 0.074693 | 0.001693 |
| OLS SE | 0.004 | 0.003498 | -0.000502 |
| IV coefficient | 0.132 | 0.131504 | -0.000496 |
| IV SE | 0.049 | 0.054964 | 0.005964 |

These are results displayed by the actual test, not values generated for the appendix. This confirms the headline table was generated; it does not verify every robustness result in the report.

---

## Entry 12: Section 6 — Fresh checkout test and README documentation

### P.R.I.M.E. mapping (retrospective)

- **[Prep]** Test a separate checkout and document actual behavior rather than promise untested reproducibility.
- **[Request]** Update the README with the one reproduction command, observed runtime and correct repository map.
- **[Iterate]** Use a new clone, revise the Markdown formatting, remove optional redundant bullets and update file paths.
- **[Mechanism Check]** Distinguish a clean repository checkout from a completely new Python environment and distinguish generated outputs from tracked files.
- **[Evaluate]** Confirm the headline results in a separate checkout and check the README against the repository structure.

### Verification log

**1. Request summary (English paraphrase, not the original prompt)**

I asked how to preserve Markdown formatting in the README, remove optional redundant information, and update the repository map to reflect the actual file structure.

**2. AI assistance (summary)**

The AI suggested README wording documenting the reproduction command, observed runtime and repository structure. It explained Markdown syntax and removed optional duplicate information at my request.

**3. What I changed, and why**

I used AI assistance to update the README instructions and repository map rather than independently rewriting the supplied wording. I requested that the optional clean-checkout and duplicate coefficient bullets be removed because the instructor's template already had a headline comparison and required the runtime and reproduction command. I retained the observed runtime and actual execution instructions. This was documentation work, not a change in the statistical estimates.

**4. What I verified, and against what**

I cloned the repository into a separate folder named `final_test` and ran `python reproduce.py`. The screenshot showed `All 1 notebooks ran in 15 s.` and the same headline table as Entry 11. Dependencies had already been installed in the Colab environment, so this was a fresh repository checkout, not a fully independent dependency-installation test. The run also printed `MissingIDFieldWarning`, which did not prevent completion. I have not established from this test alone that all report numbers match, that Python 3.11 specifically was tested, or that the changed repository link will be accepted.

---


## Appendix A — Original conversation excerpts for Entries 7–12

The following records preserve the original Korean questions, selected AI response excerpts, code, and descriptions of screenshot-only messages from the earlier appendix. English request summaries above are paraphrases, not verbatim prompts. This section is not a full transcript.

### Entry 7: Section 6 — Configuring the data downloader

**1. Actual prompt (verbatim excerpt)**

> 나머지 코드는 전부 그대로 두면 돼. 이미 교수님 코드에 Card 데이터의 다운로드 주소와 저장 경로가 설정되어 있어.
>
> 1번만 해도 된다는거지?

The user selected a sentence from the preceding AI response and asked whether changing that one setting was sufficient.

**2. Actual AI response (verbatim excerpt)**

> 응. **`download_data.py`에서는 딱 한 줄만 수정하면 돼.**

```python
MY_PAPER = "card"
```

> 나머지 코드는 삭제하거나 수정하지 말고 그대로 유지해. 그 상태로 `Commit changes`를 누르면 돼.

---

### Entry 8: Section 6 — GitHub paths and download troubleshooting

**1. Actual prompt (verbatim)**

```text
Already up to date.
total 108
drwxr-xr-x 3 root root  4096 Nov 26 03:25 .
drwxr-xr-x 1 root root  4096 Nov 26 03:24 ..
-rw-r--r-- 1 root root 32659 Nov 26 03:25 AI_Appendix.md
drwxr-xr-x 8 root root  4096 Nov 26 03:25 .git
-rw-r--r-- 1 root root  2291 Nov 26 03:25 README.md
-rw-r--r-- 1 root root 54684 Nov 26 03:25 replication_starter_card1995.ipynb
-rw-r--r-- 1 root root    96 Nov 26 03:25 requirements.txt
```

A later screenshot showed that the modified downloader was in `econ5200-lab01-data-portfolio1`, not the originally cloned midterm repository.

**2. Actual AI response (verbatim excerpt from the later exchange)**

> 아, 원인을 찾았어. **서로 다른 GitHub repository를 사용하고 있었어.**

> 그래서 아무리 `git pull`을 해도 `download_data.py`가 나타나지 않았던 거야.

The AI provided the following commands in separate cells (shown together here):

```python
%cd /content
!git clone https://github.com/jioh-shin/econ5200-lab01-data-portfolio1.git
%cd /content/econ5200-lab01-data-portfolio1
!python download_data.py
!ls -lh data/
```

---

### Entry 9: Section 6 — Configuring the automatic notebook runner

**1. Actual prompt (description of attachment and verbatim text)**

I pasted the instructor's complete `reproduce.py` script. In a follow-up, I sent a screenshot of that file in GitHub and asked:

> 내꺼 이렇게 되어 있는데?

**2. Actual AI response (verbatim excerpts)**

> 현재 27번째 줄이 다음과 같이 되어 있어.

```python
NOTEBOOKS = ROOT / "notebooks"
```

The suggested replacement was:

```python
NOTEBOOKS = ROOT
```

> 왼쪽에 `notebooks` 폴더가 있기는 하지만, 네가 실제로 작성한 `replication_starter_card1995.ipynb`는 그 폴더 밖에 있기 때문에 이렇게 수정하는 거야.

The AI also supplied these commands in separate cells:

```python
!git pull
!pip install -r requirements.txt
!python reproduce.py
```

---

### Entry 10: Sections 3 and 6 — Diagnosing a fresh-kernel error

**1. Actual prompt (screenshot description)**

I sent a screenshot of the runner's traceback ending with `NameError: name 'MY_SPEC' is not defined`. The prompt was an image rather than a typed question. I subsequently explained that the wrong notebook had been uploaded:

> 내가 다른 파일을 잘못 올렸어서 밑에 (4)를 다시 올렸고 위에거를 삭제하려고 하는데 왜 저거 안지워질까

**2. Actual AI response (verbatim excerpts)**

> 오류 원인을 찾았어. `reproduce.py`는 정상적으로 작동하지만, **네 분석 Notebook 안에서 `MY_SPEC`이라는 변수가 정의되지 않아서 실행이 중단된 거야.**

The AI suggested the following Section 3 code as one possible fix:

```python
MY_SPEC = {
    "controls": CONTROLS,
    "instrument": "nearc2",
    "sample": None
}

MY_SPEC_WHY = (
    "I use proximity to a two-year college instead of a four-year college. "
    "This helps check whether the estimated return to schooling is sensitive to the choice of instrument."
)
```

After I explained that I had uploaded the wrong file, the AI advised deleting the duplicate notebook, committing the deletion and testing the intended notebook instead.

---

### Entry 11: Section 6 — Exporting the existing headline comparison table

**1. Actual prompt (verbatim)**

> 여기 중 어디? 그리고 깃허브에서 바꿔 아님 코랩에서 바꿔?

I attached my analysis notebook with this question.

**2. Actual AI response (verbatim excerpts)**

> 즉, 마지막 `Sections 4 to 6, in your repository`라는 제목 바로 위에 새로운 코드 셀을 추가하면 돼.

```python
# Section 6: Save the headline table

from pathlib import Path

Path("output").mkdir(exist_ok=True)

comparison.to_csv("output/headline_table.csv", index=False)

print("Headline table saved.")
```

> 첨부한 Notebook에 `comparison` 변수가 실제로 존재하는 것을 확인했어. 기존 분석 코드는 수정할 필요 없어.

The AI then instructed me to save the notebook to GitHub and run `git pull` followed by `python reproduce.py` in the test notebook.

---

### Entry 12: Section 6 — Fresh checkout test and README documentation

**1. Actual prompts (verbatim excerpts)**

> 이렇게 되어 있는데 왜 너가 준걸 복붙하면 저런 볼드체나 **이런게 사라지지?

> 엥 그럼 지울래

> 그럼 어디를 수정하면 돼?

These questions followed the fresh-checkout test and concerned Markdown formatting, removing optional README bullets and correcting the repository map. I also supplied screenshots of the successful run and the README editor.

**2. Actual AI response (verbatim excerpts)**

> 내가 준 글이 Markdown으로 렌더링되어서 복사할 때 `**`나 `##` 같은 서식 기호가 빠진 거야.

> GitHub의 README 편집창에는 **Markdown 원본 형식**으로 붙여넣어야 해.

The AI supplied the following README excerpt:

````markdown
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
````

This is an excerpt, omitting three optional bullets that the AI subsequently advised removing at my request. The AI also provided a repository map matching the root-level notebook and the added scripts.

## Transcript completeness and final verification

The instructor requires full transcripts. The original conversation excerpts above are incomplete; attach the full relevant conversation, including screenshot-only messages and follow-ups, before submission. The P.R.I.M.E. mappings are retrospective. Entries 1–6 remain as supplied, including their unresolved verification notes and numerical discrepancies that need to be checked against the final notebook and report. This appendix does not certify that Sections 1 and 2 were completed without AI. Report any AI assistance accurately; the course prohibits AI use in Sections 1 and 2. No complete Section 4 or Section 5 transcript is included here.
