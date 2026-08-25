# Capstone Report — Refresh / Content Opportunity Scoring

**Author:** Kashaf Fayyaz
**Lane:** Refresh / Content Opportunity Scoring
**Repo:** https://github.com/Kashaf537/flyrank-ml-internship
**Date:** August 25, 2026

## 1. Problem framing

### Decision

This project supports the following decision:

> Which content pages should an editor review first for potential refresh or further investigation?

The goal is to prioritize content pages using historical search-performance signals rather than requiring an editor to manually inspect the entire content inventory.

The system is intended to support editorial prioritization, not automatic content changes.

### Unit of analysis

The unit of analysis is a content page represented by an anonymized `content_hash_id`.

Each page is represented using historical search-performance information.

### Input

The analysis uses anonymized content/query performance data from the FlyRank internship warehouse.

The main signals used in the modeling workflow include:

- Historical impressions
- Historical clicks
- Recent impressions
- Previous-period impressions
- Recent clicks
- Previous-period clicks
- Average search position
- Recent versus previous performance
- Query visibility and distribution signals

The final modeling dataset contained:

- 1,380 content records
- 14 predictive features
- 1 binary target variable

The pseudonymous client and content IDs were retained for grouping and traceability but were not used as predictive features.

### Output

The model produces a score for each content page.

Pages can then be ranked from highest to lowest score to create a content-review queue.

The resulting output supports actions such as:

- REFRESH
- REVIEW
- MONITOR
- PROTECT

These actions are editorial recommendations rather than automated decisions.

### Human action

An editor can start with the highest-ranked pages and investigate the supporting performance signals before deciding whether a page should actually be refreshed.

The model therefore reduces the amount of manual screening required to identify potentially important pages.

### Cost of a wrong call

A false positive can cause an editor to spend time reviewing a page that does not require intervention.

A false negative can cause a potentially useful refresh opportunity to be missed.

Because editorial resources are limited, the project focuses particularly on the quality of the highest-ranked pages.

### Why ML helps

A simple rule can identify obvious declining pages.

However, content performance can involve several signals at the same time. A machine-learning model can learn combinations of historical signals and produce a ranking that can be compared against a transparent baseline.

The purpose of the experiment is therefore:

> To determine whether a learned scoring model provides useful prioritization beyond a simple baseline rule.

## 2. Data safety

### Data source

The project uses the FlyRank ML Internship dataset/warehouse.

The warehouse was accessed through the Hugging Face dataset release using DuckDB.

The analysis did not commit the full warehouse or raw warehouse exports to the GitHub repository.

The dataset contains anonymized identifiers and search/content performance measurements.

### Data used

The capstone analysis used two primary analytical datasets.

#### Content-level performance data

The content-level dataset contained:

- 1,380 rows
- 16 columns

The columns included:

- `client_hash_id`
- `content_hash_id`
- `impressions_90d`
- `clicks_90d`
- `impressions_last30`
- `clicks_last30`
- `impressions_prev30`
- `clicks_prev30`
- `avg_position_90d`
- `avg_position_last30`
- `avg_position_prev30`
- `content_total_impressions_90d`
- `content_visible_query_count`
- `rare_query_count`
- `rare_impressions_share`
- `anonymized_impressions_share`

#### Query-level data

The query-level dataset contained:

- 10,000 rows
- 21 columns

The query-level data included anonymized query identifiers and aggregated search-performance information.

It was used during analysis and aggregation but raw query identifiers were not treated as predictive model features.

### Client coverage

An important limitation of the available analytical sample is that the modeling data contains:

- 1 unique client

Therefore, client-level generalization could not be evaluated.

This is explicitly treated as a limitation rather than hidden from the evaluation.

### Deliberately excluded information

The following information was not used as predictive features:

- `client_hash_id`
- `content_hash_id`
- `query_hash_id`
- `trend_direction`
- `trend_pct`
- Any target-derived field
- Future outcome information
- Client-identifying information
- Domains
- URLs
- Private queries
- Credentials or access tokens

Pseudonymous IDs were used only for:

- joining datasets
- grouping
- identifying individual content records
- constructing the analytical dataset

They were not included in X as predictive features.

### Leakage risks

The main leakage risk was allowing information that directly describes the target to become a model feature.

In particular:

- `trend_direction`
- `trend_pct`

were excluded because they are derived from performance movement and could directly reveal the target.

The modeling design therefore separates:

```
Historical information
        ↓
Feature construction
        ↓
Target definition
        ↓
Train / test evaluation
```

Future outcome information was not used as a predictive feature.

### Public safety

The public project does not expose:

- Client names
- Client domains
- Client URLs
- Private search queries
- Credentials
- API tokens
- Raw warehouse exports

The analysis uses anonymized identifiers supplied by the internship dataset.

## 3. Baseline

### Baseline approach

The baseline represents a simple, transparent prioritization strategy.

It uses historical performance signals to identify pages that show evidence of performance decline.

The baseline provides a human-readable benchmark against which the Decision Tree can be compared.

The important principle is that the baseline and ML model are evaluated using the same test data.

### Why the baseline matters

A machine-learning model should not be considered useful simply because it produces a high metric.

The important question is:

> Does the model improve prioritization compared with a simple rule?

The baseline therefore provides the reference point for evaluating whether the learned model adds value.

### Evaluation metric

The primary ranking metrics used in the capstone are:

- Precision@20
- Precision@50

The target base rate is also reported so that Precision@K is interpreted in context.

## 4. Model / analysis

### Model

The final capstone model is a:

**Decision Tree Classifier**

A Decision Tree was selected because it is:

- Easy to interpret
- Suitable for tabular data
- Able to learn non-linear relationships
- Useful for producing understandable decision rules
- Appropriate for a transparent editorial-prioritization experiment

The model produces a score that is used to rank content pages.

### Target

The target is a binary indicator:

`is_declining`

where:

- 0 = not classified as declining
- 1 = classified as declining

The final modeling sample contained:

| | Count |
|---|---|
| Total records | 1,380 |
| Not declining | 826 |
| Declining | 554 |

Therefore, the positive-class rate was approximately:

**40.1%**

This base rate is important when interpreting Precision@K.

### Features

The final model used:

**14 numerical features**

constructed from the anonymized content-performance data.

The feature set was designed around historical search-performance information, including:

- Impressions
- Clicks
- Recent-period performance
- Previous-period performance
- Average search position
- Content/query visibility characteristics
- Search distribution characteristics

Identifier fields were deliberately excluded from the model.

### Feature shape

The final modeling matrices were:

```
X shape: (1380, 14)
y shape: (1380,)
```

This confirms that the model was trained on 14 predictive features and one binary target.

### Features deliberately excluded

The following were intentionally excluded:

- `client_hash_id`
- `content_hash_id`
- `query_hash_id`
- `trend_direction`
- `trend_pct`

along with any future-window or target-derived information.

### Validation design

The available modeling sample contains only:

- 1 unique client

Therefore, a client-grouped train/test split was not possible.

The model was consequently evaluated using a stratified content-level train/test split so that the positive and negative classes were represented in both partitions.

The target distribution was:

- Class 0: 826
- Class 1: 554

A fixed random state was used to make the experiment reproducible.

### Leakage check

The modeling dataset was checked to ensure that:

- Target-derived fields were not included.
- Identifier fields were not used as model features.
- Future outcome variables were not directly included.
- The target was represented separately from the feature matrix.

This is particularly important because fields such as `trend_direction` and `trend_pct` can encode the same information as the decline target.

## 5. Evaluation

### Primary evaluation

The model was evaluated on a held-out test set.

The main ranking metrics are:

- Precision@20
- Precision@50

These metrics answer a practical question:

> If an editor starts with the highest-ranked pages, how many of those pages are actually positive cases?

### Model performance

The current Decision Tree experiment produced:

| Metric | Decision Tree |
|---|---|
| Precision@20 | 0.650 |
| Precision@50 | 0.620 |
| Positive-class base rate | 0.401 |

The model's Precision@20 of 65.0% is substantially above the approximately 40.1% positive-class base rate.

Precision@50 of 62.0% is also above the base rate.

This indicates that the model's ranking concentrated more positive cases near the top of the recommendation list than would be expected from the overall class distribution.

### Important interpretation

These results should be described as measured ranking performance, not as proof that refreshing these pages will improve search performance.

The model identifies pages that resemble historically declining pages according to the defined target.

It does not establish causality.

### Model vs baseline

The final `results_df` generated by the notebook contains the model/baseline comparison.

The final paper should use the values from that table rather than manually entering numbers.

The comparison follows:

| Metric | Baseline | Decision Tree |
|---|---|---|
| Precision@20 | From results_df | 0.650 |
| Precision@50 | From results_df | 0.620 |
| Base rate | 0.401 | 0.401 |

Both approaches are evaluated on the same held-out data.

### Error analysis

The ranking output was inspected to identify:

- Highly ranked declining pages
- Highly ranked non-declining pages
- Pages missed by the model
- Differences between the model ranking and baseline ranking

The model's highest-ranked pages should therefore be treated as review candidates, rather than guaranteed refresh opportunities.

## 6. Interpretation

### What the model found

The Decision Tree learned relationships between historical search-performance characteristics and the binary decline target.

The feature-importance analysis provides a way to identify which input signals contributed most strongly to the learned decision structure.

The resulting feature-importance chart is generated directly from the trained model.

### Feature importance

The capstone notebook generates a feature-importance table and visualization from:

`feature_importance`

The chart should be included in the final research paper under the Results/Interpretation section.

Feature importance should be interpreted as:

> Contribution to the model's decision-making process

rather than:

> Proof that the feature causes performance decline.

### Negative results

The Decision Tree should not automatically be considered superior simply because it is an ML model.

If its improvement over the baseline is small, the appropriate conclusion is that a simple rule may already capture much of the useful signal.

That is still a valid result.

### Interpretation language

The project uses careful language such as:

- observed
- measured
- associated with
- directional
- ranked
- decision-support

The project does not claim:

- causal impact
- guaranteed traffic improvement
- Google's ranking algorithm
- guaranteed results from refreshing content

## 7. Recommendation

### Ranked action queue

The trained model generates a ranked recommendation list.

The current notebook creates the top-ranked pages using:

`recommendations`

and:

`top_recommendations`

The ranking is based on:

`model_score`

The public paper should not expose client-identifying information.

### Editorial playbook

#### 1. REFRESH

Prioritize a page for editorial review when:

- The model gives it a high opportunity score.
- Historical search visibility is meaningful.
- The page shows evidence consistent with the defined decline condition.

The editor should then inspect the actual content before making changes.

#### 2. REVIEW

Use this category for pages with elevated model scores where the evidence is less decisive.

These pages should receive additional investigation.

#### 3. MONITOR

Pages with weaker evidence should remain under observation rather than receiving immediate editorial changes.

#### 4. PROTECT

Strong or stable pages should not automatically be changed simply because the model was applied to them.

Editorial review remains necessary.

### How an editor could use the system

A practical workflow is:

```
All content pages
       ↓
Model scoring
       ↓
Rank by opportunity score
       ↓
Top 20 / Top 50
       ↓
Editor investigation
       ↓
Refresh / Review / Monitor / Protect
```

This turns the model into a prioritized editorial queue.

### Confidence and limitations

A high model score means:

> The page resembles examples associated with the defined decline target in the available training data.

It does not mean:

> Refreshing the page will definitely improve rankings, clicks, or traffic.

The model is therefore a decision-support system, not an autonomous content strategy.

## 8. Reproducibility

### Repository

The complete project is available at:

https://github.com/Kashaf537/flyrank-ml-internship

### Capstone notebook

The main capstone analysis is located under:

`work/notebooks/capstone.ipynb`

The notebook contains:

- Question
- Data
- Methodology
- Results
- Limitations
- Ranked recommendations
- Artifacts
- Report

This document is stored at:

`work/capstone_report.md`

### Submission

The final deployed research paper URL will be stored at:

`submission/paper_url.txt`

That file must contain exactly one line containing the deployed paper URL.

### Data access

The warehouse was accessed through the approved FlyRank dataset using DuckDB.

The full warehouse is not committed to the repository.

### Reproducibility

The experiment uses a fixed random state for the train/test procedure.

The final notebook should be run from top to bottom using:

**Runtime → Run all**

The notebook should complete without relying on hidden state from previous executions.

### Reproducibility checklist

- [ ] Data loaded from the approved FlyRank dataset
- [ ] Data exploration completed
- [ ] Leakage fields excluded
- [ ] Identifier fields excluded from model features
- [ ] Modeling dataset constructed
- [ ] Decision Tree trained
- [ ] Test-set evaluation completed
- [ ] Precision@20 calculated
- [ ] Precision@50 calculated
- [ ] Base rate calculated
- [ ] Ranked recommendations generated
- [ ] Feature importance generated
- [ ] Final research paper deployed
- [ ] submission/paper_url.txt created
- [ ] Final paper URL added
- [ ] Final fresh notebook run completed

## Claims checklist before submitting

### Claim language

The project describes results as:

- **Observed** — directly visible in the data.
- **Measured** — calculated using defined metrics.
- **Directional** — useful for identifying patterns without claiming causality.
- **Decision-support** — intended to help prioritize editorial work.

### Claims we do NOT make

This project does not claim that:

- The model predicts Google's algorithm.
- The model explains Google's ranking system.
- Content refreshes cause ranking improvements.
- The model guarantees future traffic growth.
- The model automatically determines the correct editorial action.

### Data safety

- [ ] No client names.
- [ ] No client domains.
- [ ] No client URLs.
- [ ] No private search queries.
- [ ] No credentials.
- [ ] No API tokens.
- [ ] No raw warehouse export committed to the repository.
- [ ] Pseudonymous IDs are not predictive features.

### Evaluation integrity

- [ ] Base rate reported.
- [ ] Model evaluated on held-out data.
- [ ] Precision@20 reported.
- [ ] Precision@50 reported.
- [ ] Feature importance generated.
- [ ] Ranked recommendations generated.
- [ ] Leakage fields excluded.
- [ ] Model uses the same target as the evaluation.
- [ ] Final baseline comparison numbers copied from results_df.
- [ ] Final fresh run reproduced.
