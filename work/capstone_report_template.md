# Capstone Report — Refresh / Content Opportunity Scoring

* **Author:** Kashaf Fayyaz
* **Lane:** Refresh / Content Opportunity Scoring
* **Repo:** https://github.com/Kashaf537/flyrank-ml-internship
* **Date:** August 24, 2026

> This report is a living document. Results marked `TBD` will be replaced after the final
> capstone pipeline has been executed and freshly validated.

## 1. Problem framing

### Decision

This project supports the decision:

> **Which content pages should a FlyRank editor review first for potential refresh or further investigation?**

The goal is to prioritize a large set of content pages so that limited editorial resources can be directed toward pages with stronger evidence of future content-review opportunity.

### Unit of analysis

The primary unit of analysis is a **content page at a defined prediction/decision date**.

Historical observations for each page are aggregated into a feature vector using information that would have been available at the prediction date.

### Input

The analysis uses historical content, search-performance, and engagement signals available in the FlyRank warehouse.

Potential feature groups include:

* Search impressions
* Search clicks
* Click-through rate
* Average search position
* Engagement signals where available
* Content age
* Days since last update
* Historical performance/momentum
* Content characteristics such as word count where appropriate

The final feature list will be recorded after the data contract and leakage audit are completed.

### Output

The system produces:

1. A score representing the estimated content-review opportunity.
2. A ranked list of pages.
3. Human-readable reason codes.
4. A recommended action such as:

   * **REFRESH**
   * **REVIEW**
   * **MONITOR**
   * **PROTECT**, where supported by the evidence.

### Human action

A FlyRank editor can use the ranked queue to decide which pages deserve investigation first. The system is a **decision-support tool**, not an automatic content-editing system.

An editor should review the underlying page and business context before taking action.

### Cost of a wrong call

A false positive can waste editorial time by prioritizing a page that does not require attention.

A false negative can cause a potentially valuable content opportunity to be missed.

Because editorial resources are limited, the project emphasizes **ranking quality at the top of the queue**, particularly Precision@20 and Precision@50.

### Why data/ML helps

A manual rule can identify obvious cases such as stale pages with substantial search visibility. However, a large content inventory may contain more complex combinations of signals.

The project therefore compares:

* a transparent, human-designed baseline, and
* an ML-based scoring model.

The purpose is not to claim that ML is automatically better, but to test whether a learned model can produce a more useful ranked queue than a simple rule under honest validation.

---

## 2. Data safety

### Data used

The project uses the **FlyRank ML Internship dataset/warehouse** made available for the internship.

The primary data source for the final analysis is the FlyRank content-performance warehouse queried through DuckDB rather than downloading the entire warehouse into the repository.

The final analysis will document:

* Dataset release used
* Tables used
* Date range
* Aggregation grain
* Inclusion criteria
* Exclusion criteria
* Data availability constraints

### Data tables

The analysis is expected to use the content and daily performance data required to construct historical page-level features and future outcomes.

The final notebook will record the exact tables used after the data contract is implemented.

### Deliberately excluded information

The following types of information will not be used as predictive features:

* `trend_direction`
* `trend_pct`
* Any field directly derived from the target
* Any future-window information
* Product decision flags or fields that already encode the decision being predicted
* Client-identifying information
* Domains, URLs, private queries, or other identifying information
* Pseudonymous IDs as numerical predictive features

### Pseudonymous IDs

Client and content identifiers may be used for:

* grouping,
* joins,
* aggregation,
* train/test grouping,
* traceability during private analysis.

They will **not** be used as predictive features.

No client-identifying details will appear in the public `work/` materials or deployed paper.

### Leakage risks

The most important leakage risk is allowing information from the outcome window to enter the feature window.

The project therefore follows a prediction-time design:

```text
Historical feature window
        ↓
    Decision date
        ↓
Future outcome window
```

Features must only use information available before the decision date.

In particular:

* `trend_direction` is excluded because it represents an outcome-derived state.
* `trend_pct` is excluded because the starter analysis demonstrates that it directly encodes the decline label.
* Future performance metrics are excluded from features.
* Target-derived product flags are excluded.
* Pseudonymous identifiers are grouping variables only.

### Public safety

No client names, domains, URLs, private search queries, credentials, raw warehouse exports, or other client-identifying information will be included in the public repository or paper.

---

## 3. Baseline

### Baseline approach

The first benchmark is a transparent content-review prioritization rule based on observable historical signals.

The baseline is designed around the intuition that a page may deserve review when it is:

* relatively stale, and
* still sufficiently visible/search-demanded.

A baseline score can therefore prioritize pages using combinations of:

* freshness/update recency,
* search visibility,
* historical performance.

The exact final rule and thresholds will be recorded in the baseline notebook.

### Why this is a fair comparison

The baseline represents a realistic human-designed prioritization heuristic.

It uses the same eligible information and is evaluated on the **same validation data and metrics** as the ML model.

The purpose is to answer:

> Does the ML model provide additional ranking value beyond a simple transparent rule?

### Baseline result

Final baseline performance:

* **Precision@20:** `TBD`
* **Precision@50:** `TBD`
* **Average Precision:** `TBD`
* **ROC-AUC:** `TBD`
* **Task base rate:** `TBD`

These values will be replaced with results from the final fresh run.

---

## 4. Model / analysis

### Method

The primary modeling approach will be a supervised ML scoring model trained to estimate the likelihood that a page becomes a content-review opportunity in a future observation window.

Candidate models may include:

* Logistic Regression as an interpretable linear baseline/model
* Decision Tree
* Random Forest or another suitable tree-based model

The final model will be selected based on validation performance, stability, interpretability, and usefulness for ranking rather than complexity alone.

### Target definition

The intended target is:

> **Whether a page experiences a predefined meaningful performance decline or content-review opportunity during a future window after the prediction date.**

The exact threshold and future window will be defined in the data contract before model training and will not use future information in the feature construction process.

### Feature list

The final feature list will be documented after the warehouse analysis and leakage audit.

Candidate feature groups include:

#### Search visibility

* Historical impressions
* Historical clicks
* CTR
* Average position
* Days with search activity

#### Content lifecycle

* Content age
* Days since last update

#### Engagement

* Engagement rate
* Other eligible engagement aggregates

#### Momentum

* Recent-vs-prior-window performance changes
* Historical volatility
* Recent performance trend calculated only from pre-decision observations

#### Content characteristics

* Word count
* Other non-identifying content characteristics available before the prediction date

### Features deliberately excluded

The following will not be used:

* `trend_direction`
* `trend_pct`
* Future-window metrics
* Target-derived fields
* Product decision flags
* Client IDs as predictive variables
* Content IDs as predictive variables
* Any client-identifying information

### Starter-model evidence

The Week 2 educational experiment showed that a depth-2 decision tree could provide a readable learned rule, while the train/test experiment produced:

* Precision@20: **0.650**
* Precision@50: **0.620**

These numbers are **not final capstone results**. They came from the starter dataset and are retained only as development context.

The final capstone results will be produced using the defined full-warehouse methodology and honest validation.

---

## 5. Evaluation

### Validation design

The final evaluation will use a validation strategy that respects the temporal nature of the problem.

The key principle is:

> **Training information must come from the past, while evaluation represents future/unseen outcomes.**

The final split design will be documented before the final model is selected.

Where appropriate, client grouping will also be considered so that repeated observations from the same client do not create an unrealistically easy evaluation.

### Primary metrics

The primary ranking metrics are:

* **Precision@20**
* **Precision@50**

These directly measure whether the highest-priority recommendations contain a high proportion of actual opportunities.

Secondary metrics may include:

* Average Precision
* ROC-AUC
* Lift over baseline
* Calibration or probability-quality diagnostics where useful

### Base rate

The target base rate will always be reported alongside precision metrics.

* **Positive-class base rate:** `TBD`

This prevents a high Precision@K value from being interpreted without knowing how common the positive class is.

### Model vs baseline

Final comparison:

| Metric            | Baseline | ML Model | Difference |
| ----------------- | -------: | -------: | ---------: |
| Precision@20      |      TBD |      TBD |        TBD |
| Precision@50      |      TBD |      TBD |        TBD |
| Average Precision |      TBD |      TBD |        TBD |
| ROC-AUC           |      TBD |      TBD |        TBD |
| Base rate         |      TBD |      TBD |          — |

The baseline and model will be evaluated on the **same held-out data**.

### Error analysis

The final analysis will inspect:

* False positives among highly ranked pages
* False negatives
* Pages that the baseline ranks highly but the model does not
* Pages the model identifies that the baseline misses
* Whether errors concentrate in particular performance ranges or data-availability conditions

A short qualitative error analysis will accompany the final metrics.

---

## 6. Interpretation

The interpretation will focus on what the model can actually establish from the observed data.

The final analysis will examine:

* Feature importance
* Model coefficients where applicable
* Decision-tree rules where applicable
* Ranking behavior
* Differences between the baseline and ML model

### Expected interpretation

Potential findings may involve combinations of:

* search visibility,
* historical momentum,
* content freshness,
* search position,
* engagement,
* content characteristics.

However, these are hypotheses until confirmed by the final analysis.

### Surprises and negative results

The project will explicitly report unexpected or negative findings.

For example:

* A feature may contribute little predictive value.
* The ML model may fail to outperform the baseline.
* Performance may decline substantially on the held-out period.
* A seemingly intuitive signal may not generalize.
* Additional model complexity may provide little ranking benefit.

A result showing that the ML model does not materially improve the baseline will still be considered a valid research finding if it is supported by honest evaluation.

### Interpretation language

The project will use language such as:

* "associated with"
* "observed"
* "measured"
* "ranked"
* "directional"
* "supports prioritization"

It will not claim that a feature **causes** search-performance changes.

---

## 7. Recommendation

### Ranked action queue

The final system will produce a ranked content-review queue containing, at minimum:

| Rank | Content           | Score | Action | Reason code | Confidence |
| ---: | ----------------- | ----: | ------ | ----------- | ---------- |
|  TBD | Anonymous page ID |   TBD | TBD    | TBD         | TBD        |

Client-identifying information will not be exposed in the public report.

### Recommended actions

The scoring system will support actions such as:

#### REFRESH

Prioritize a page for content review when the evidence indicates a meaningful review opportunity and the page has sufficient historical visibility/value.

#### REVIEW

Investigate the page further when the model identifies an opportunity but the evidence is less decisive.

#### MONITOR

Continue observing the page when signals suggest possible movement but the evidence is insufficient for immediate editorial action.

#### PROTECT

Avoid unnecessary changes when a page is performing strongly or showing stable/recovering behavior, subject to editorial review.

### How an editor could use the system

A FlyRank editor could begin each review cycle with the highest-ranked pages instead of manually scanning the entire content inventory.

For each recommended page, the system provides:

1. Priority score
2. Rank
3. Reason code
4. Suggested action
5. Confidence/strength of evidence

The editor then verifies the recommendation against the actual content and broader business context.

### Confidence and limitations

The ranking should be treated as **decision support**, not an automatic decision.

A high model score means:

> The page resembles pages associated with the defined future opportunity under the training data and validation design.

It does **not** mean:

> Refreshing this page will definitely improve its search performance.

The model does not establish causality, guarantee traffic growth, or predict Google's ranking algorithm.

---

## 8. Reproducibility

### Repository

The complete project is maintained in:

`https://github.com/Kashaf537/flyrank-ml-internship`

The repository contains the weekly work, capstone notebook, supporting scripts, and final submission materials.

### Expected capstone files

```text
work/
├── capstone.ipynb
└── capstone_report.md

submission/
└── paper_url.txt
```

The exact repository structure will follow the existing project conventions.

### Environment

Python environment and dependencies are defined through the repository's dependency files.

The final environment will be recorded using the repository's `requirements.txt` and/or other environment configuration where applicable.

### Random seed

Where randomized ML procedures are used:

```text
random_state = 42
```

will be used unless a documented reason requires another setting.

### Data access

The full warehouse is accessed through the approved FlyRank/Hugging Face release using DuckDB.

The full raw warehouse will not be committed to GitHub.

### Re-running the analysis

From a fresh clone, the intended workflow is:

```bash
git clone https://github.com/Kashaf537/flyrank-ml-internship.git
cd flyrank-ml-internship

python -m venv .venv
```

Activate the environment and install dependencies:

```bash
pip install -r requirements.txt
```

The capstone notebook can then be executed in the configured environment after the required FlyRank dataset access has been established.

The exact final execution commands will be updated here after the capstone pipeline is complete.

### Reproducibility checks

Before submission:

* [ ] Fresh environment successfully runs the capstone.
* [ ] All notebook cells execute without manual hidden state.
* [ ] Random seeds are fixed.
* [ ] Final metrics are reproduced.
* [ ] Baseline and model use the same validation split.
* [ ] Feature definitions match the data contract.
* [ ] No leakage features are present.
* [ ] No client-identifying information is exposed.
* [ ] Final paper numbers match a fresh run.
* [ ] `submission/paper_url.txt` contains exactly one deployed paper URL.

---

# Claims checklist before submitting

### Claim language

The final paper will use:

* **Observed** for directly measured results.
* **Measured** for calculated metrics.
* **Directional** for associations or patterns that should not be interpreted causally.
* **Decision-support** for recommendations produced by the ranking system.

### Claims we will NOT make

We will not claim:

* The model predicts Google's algorithm.
* The model proves why Google rankings change.
* Refreshing a page causes improved traffic or rankings.
* The model guarantees future performance.
* The model identifies universally correct editorial actions.

### Data safety

* [ ] No client names.
* [ ] No client domains.
* [ ] No URLs from client data.
* [ ] No private search queries.
* [ ] No credentials or tokens.
* [ ] No raw warehouse exports.
* [ ] No client-identifying information in `work/`.
* [ ] Pseudonymous IDs are never used as predictive features.

### Evaluation integrity

* [ ] Base rate reported.
* [ ] Baseline reported.
* [ ] Model reported.
* [ ] Both evaluated on the same held-out data.
* [ ] Precision@20 reported.
* [ ] Precision@50 reported.
* [ ] Average Precision and/or AUC reported where appropriate.
* [ ] Error analysis completed.
* [ ] Leakage audit completed.
* [ ] Final numbers reproduced from a fresh run.

### Final status

**Current status:** Capstone planning and methodology definition.

**Final model results:** `TBD`

**Final ranked recommendations:** `TBD`

**Final paper URL:** `TBD`

**Submission URL file:** `submission/paper_url.txt`
