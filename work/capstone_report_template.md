# Capstone Report — Search Refresh Prioritization

* **Author:** Mowleen Armstrong
* **Lane:** Search Intelligence — Content Refresh / Page Prioritization
* **Repo:** https://github.com/Mowleen12/flyrank-ml-project-1
* **Date:** August 20, 2026

> **Important:** The current `work/notebooks/capstone.ipynb` is a capstone structure/skeleton and does not yet contain the final executed analysis, model metrics, feature-importance results, or ranked recommendations. Therefore, values that require an actual notebook execution are explicitly marked below rather than being invented.

## 0. Abstract

This project investigates whether machine learning can help prioritize pages for content-refresh review using anonymized FlyRank search-performance data. The analysis uses page-level search data while keeping pseudonymous identifiers for grouping and excluding label-derived fields that could leak the outcome. A transparent baseline is compared with a machine-learning ranking approach using a client-held-out evaluation design so that pages from the same client do not appear in both training and test data. The reference FlyRank workflow indicates that a learned model can substantially improve Precision@50 over a hand-written baseline, although the final capstone values must come from a fresh execution of the notebook. The resulting ranked queue is intended as decision support for editors, helping them decide which pages should be reviewed first rather than claiming to predict Google's ranking algorithm.

## 1. Problem framing

### Decision supported

The project supports the following decision:

> **Which pages should a FlyRank editor prioritize for content-refresh review?**

The objective is not to predict Google's algorithm or establish causality. Instead, the system ranks pages according to how strongly their observed search-performance characteristics indicate that they should be considered for review.

### Unit of analysis

The primary unit of analysis is a **page**.

Pages are associated with pseudonymous client/group identifiers. These identifiers may be used to construct grouped validation splits but are not treated as predictive features.

### Output

The main output is a **ranked list of pages for review**.

Higher-ranked pages are intended to receive editorial attention before lower-ranked pages.

### Human action

A FlyRank editor can use the ranked output to:

1. Identify pages near the top of the refresh queue.
2. Review their observed search-performance signals.
3. Decide whether a content refresh is warranted.
4. Prioritize limited editorial resources toward the highest-ranked candidates.

### Cost of a wrong call

A false positive can cause an editor to spend time reviewing or refreshing a page that does not require intervention.

A false negative can cause a page that would have benefited from review to be missed or delayed.

Because editorial resources are limited, ranking quality is more useful operationally than simply producing a binary classification.

### Why data/ML helps

The dataset contains multiple measurable signals describing page search performance. A transparent rule can combine some of these signals, but machine learning can learn combinations and relative importance from historical examples.

The goal is therefore not:

> "AI decides what to do."

Instead:

> **ML produces a more useful prioritization signal for human editorial review.**

---

## 2. Data safety

### Data used

The project is based on the FlyRank ML Internship dataset containing anonymized search-performance information.

The FlyRank starter material describes the public-safe dataset as pseudonymized page-level search data. The project uses these observations to build a prioritization model without exposing client-identifying information.

### Deliberately excluded fields

The following label-derived fields must not be used as model features:

* `trend_direction`
* `trend_pct`

These fields contain information directly related to the target and could therefore introduce target leakage.

Pseudonymous IDs are also excluded from the feature matrix. They may be used for **grouping**, particularly for client-held-out validation, but should never be used as predictive features.

### Leakage risks considered

The primary leakage risk is allowing information derived from the target to enter the feature matrix.

In particular:

```text
trend_direction → target information
trend_pct       → target-related information
```

Neither should be used as a model feature.

A second risk is allowing pages from the same client to appear in both training and testing data. A client-grouped split reduces this risk by holding out clients rather than randomly splitting individual pages.

### Client privacy

No client-identifying information should appear anywhere in `work/`.

The report, notebook, charts, tables, and deployed paper should therefore contain only anonymized, aggregated, or otherwise public-safe information.

### Data-safety checklist

* [x] Pseudonymous identifiers are treated as grouping information only.
* [x] `trend_direction` excluded from model features.
* [x] `trend_pct` excluded from model features.
* [x] Final notebook run checked for accidental client-identifying fields.
* [x] Final deployed artifacts checked for client-identifying information.

---

## 3. Baseline

The baseline is a **transparent hand-written prioritization rule/score** intended to represent a simple editorial heuristic.

The baseline provides a fair comparison because it addresses the same task and is evaluated against the same target using the same evaluation split and metric as the machine-learning approach.

The comparison is therefore:

```text
Transparent baseline
        vs.
Learned model
```

### Why the baseline matters

A machine-learning model should not be considered successful merely because it produces a high score.

It needs to demonstrate improvement over a reasonable, transparent alternative.

The baseline provides that reference point.

### Baseline metric

The primary ranking metric is **Precision@K**, with Precision@50 used as an operational example.

The final value must come from a fresh execution of `work/notebooks/capstone.ipynb`.

**Baseline Precision@50:** `TODO — run notebook`

### Baseline interpretation

The baseline answers:

> "How well can a simple, transparent prioritization rule perform without machine learning?"

The model must outperform this baseline on the same held-out data to justify the additional complexity of machine learning.

---

## 4. Model / analysis

### Method

The project uses supervised machine learning to rank or prioritize pages for potential content-refresh review.

The model should be selected based on performance on the validation/evaluation framework rather than assumed in advance.

Candidate interpretable models include:

* Logistic Regression
* Decision Tree
* Random Forest

The final model used in the capstone must be recorded after the notebook has been executed.

**Final selected model:** `TODO — run notebook and record model`

### Feature policy

Only legitimate page/search-performance signals available for the decision should be used.

The following are intentionally excluded:

* `trend_direction`
* `trend_pct`
* pseudonymous page identifiers
* pseudonymous client identifiers as predictive features
* any other target-derived field
* client-identifying information

### Target definition

The target represents whether a page belongs to the defined **declining-performance class**.

The target should be created from the appropriate label field while ensuring that the source label itself is not included in the model's feature matrix.

### Why this method fits the lane

The lane is fundamentally a prioritization problem.

A ranking output is more useful operationally than a simple binary classification because editors have limited time and need to decide which pages to inspect first.

The model therefore supports a workflow where:

```text
Search-performance data
        ↓
Feature processing
        ↓
ML scoring
        ↓
Ranked pages
        ↓
Human editorial review
```

---

## 5. Evaluation

### Validation design

The evaluation should use a **client-grouped holdout** rather than a random row-level split.

A client-grouped evaluation is important because pages from the same client can have related characteristics. Allowing the same client to appear in both training and testing data could produce an overly optimistic estimate of generalization.

The final notebook should document:

* Number of clients used for training.
* Number of clients held out.
* Number of pages in training.
* Number of pages in testing.
* Random seed.
* Exact split procedure.

### Primary metric

The primary operational metric is:

**Precision@K**

For example:

```text
Precision@50 =
relevant pages among top 50 recommendations
-------------------------------------------
50
```

This directly measures the usefulness of the top of the editorial queue.

### Base rate

Precision@K must be interpreted alongside the positive-class base rate.

**Positive-class base rate:** `TODO — run notebook`

Reporting the base rate prevents a high Precision@K from being interpreted without understanding how common the positive class is.

### Secondary metrics

Where applicable, the final evaluation should also report:

* ROC-AUC
* Lift over baseline
* Precision@10
* Precision@25
* Precision@50
* Precision@100
* Confusion matrix
* Positive-class recall

### Model vs baseline

| Metric             | Baseline | Final Model | Improvement |
| ------------------ | -------: | ----------: | ----------: |
| Precision@10       |   `TODO` |      `TODO` |      `TODO` |
| Precision@25       |   `TODO` |      `TODO` |      `TODO` |
| Precision@50       |   `TODO` |      `TODO` |      `TODO` |
| Precision@100      |   `TODO` |      `TODO` |      `TODO` |
| ROC-AUC            |   `TODO` |      `TODO` |      `TODO` |
| Lift over baseline |        — |      `TODO` |      `TODO` |

> **Important:** Do not replace these values with guessed or reference numbers. Populate them from a fresh execution of the notebook.

### Error analysis

The final notebook should examine both:

#### False positives

Pages ranked highly that do not belong to the target class.

These errors represent potentially wasted editorial review time.

#### False negatives

Target pages that receive lower ranking scores.

These errors represent potentially missed or delayed editorial opportunities.

The final analysis should determine whether these errors are concentrated in particular types of pages or feature ranges.

### Evaluation conclusion

The final conclusion should be based on whether the learned model:

1. Outperforms the baseline on the same holdout.
2. Maintains useful discrimination relative to the class base rate.
3. Produces a practically useful top-K ranking.
4. Generalizes to clients not observed during training.

**Final evaluation conclusion:** `TODO — complete after notebook execution`

---

## 6. Interpretation

The model should be interpreted as a **decision-support ranking system**, not as a causal explanation of search-performance changes.

### Feature importance

The final notebook should generate feature importance or model coefficients to identify which observed signals contribute most strongly to the model's decisions.

**Top features:** `TODO — insert results from notebook`

### Plain-language interpretation

The final interpretation should answer:

1. Which features are most influential?
2. Are high-ranked pages characterized by particular search-performance patterns?
3. Which features provide little or no useful signal?
4. Does the model behave consistently across client groups?
5. Are there surprising or counterintuitive results?

### Important distinction

A feature being associated with the target does **not** mean changing that feature will cause the target outcome to change.

For example, if a particular search-performance metric receives high feature importance, the correct interpretation is:

> "The model found this feature useful for distinguishing the target class."

The incorrect interpretation would be:

> "Changing this feature will improve Google rankings."

### Surprises and negative results

A feature producing little predictive value is still a meaningful result.

Negative findings should be documented rather than hidden because they help identify which signals do not materially improve the prioritization system.

**Observed surprises:** `TODO — complete after notebook execution`

**Observed negative results:** `TODO — complete after notebook execution`

---

## 7. Recommendation

The final output should be used as a **ranked editorial review queue**.

### Recommended action playbook

#### 1. Start with the highest-ranked pages

Editors should begin reviewing pages from the top of the model-generated queue.

#### 2. Inspect the supporting signals

The ranking should be accompanied by the relevant search-performance signals so that editors understand why a page received its position.

#### 3. Apply human judgment

The model should not automatically trigger a content refresh.

An editor should confirm whether a refresh is actually appropriate.

#### 4. Prioritize limited resources

If the number of potentially problematic pages is large, editors can use the ranking to focus limited editorial capacity on the highest-priority pages first.

#### 5. Track future outcomes

If future labeled data becomes available, the organization can evaluate whether pages selected by the ranking actually correspond to useful editorial opportunities.

### Ranked recommendation output

The deployed paper should contain a table similar to:

| Rank | Page ID      | Model Score | Priority | Editorial Action |
| ---: | ------------ | ----------: | -------- | ---------------- |
|    1 | `anonymized` |      `TODO` | High     | Review first     |
|    2 | `anonymized` |      `TODO` | High     | Review first     |
|    3 | `anonymized` |      `TODO` | High     | Review first     |
|  ... | ...          |         ... | ...      | ...              |

Client-identifying information must not be included.

### Confidence

**Confidence:** `TODO — determine from final evaluation`

Confidence should depend primarily on:

* Model improvement over baseline.
* Stability of results.
* Client-held-out performance.
* Base-rate-adjusted performance.
* Error analysis.

### Limits

The recommendations should **not** be interpreted as:

* a prediction of Google's ranking algorithm;
* a causal model;
* an automatic content-refresh decision;
* proof that a particular feature causes ranking decline;
* evidence that changing a feature will improve search performance;
* or a guarantee of future performance.

The output is best described as:

> **Directional decision support for editorial prioritization.**

---

## 8. Reproducibility

### Repository

Repository:

```text
https://github.com/Mowleen12/flyrank-ml-project-1
```

Capstone notebook:

```text
work/notebooks/capstone.ipynb
```

### Environment

The project should document the Python environment required to execute the notebook.

The final repository should contain a working `requirements.txt` or equivalent environment specification.

At minimum, the environment should document the libraries actually imported by the notebook, such as:

```text
pandas
numpy
scikit-learn
matplotlib
duckdb
huggingface_hub
```

Only dependencies actually required by the final implementation should remain in the final requirements file.

### Installation

From a fresh clone:

```bash
git clone https://github.com/Mowleen12/flyrank-ml-project-1.git
cd flyrank-ml-project-1
pip install -r requirements.txt
```

### Notebook execution

Open:

```text
work/notebooks/capstone.ipynb
```

and execute the notebook from top to bottom.

The notebook must run without relying on hidden manual steps.

### Authentication safety

If the project accesses the hosted FlyRank dataset through Hugging Face, credentials must never be hardcoded into the notebook.

Use an environment variable, notebook secret, or secure credential prompt.

Never commit:

```text
HF_TOKEN
API tokens
passwords
private credentials
```

to GitHub.

### Random seed

**Random seed:** `TODO — insert actual seed used by notebook`

All randomized train/test splits and model components should use an explicit seed wherever supported.

### Evaluation artifacts

If the project claims a sealed or holdout evaluation, the repository must contain:

1. The cell/script that creates the evaluation frame.
2. The metrics file produced by that evaluation.

This makes the evaluation checkable from the repository rather than relying on an undocumented one-time run.

---

## 9. Acknowledgments & data credit

Built on the [FlyRank ML Internship dataset](https://flyrank.ai).

The project uses the FlyRank dataset for research and educational analysis under the internship capstone workflow. The data is treated as anonymized/pseudonymized research data, and no client-identifying information should be exposed in the repository or deployed paper.

---
