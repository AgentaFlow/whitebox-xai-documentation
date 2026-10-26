---
description: WhiteBoxXAI Assurance — explainability (SHAP, LIME and DiCE counterfactuals), fairness audits and drift detection for a system, each computed from logged data and shown with a plain-language verdict.
---

# Assurance

An auditor's next questions after *what AI do you run* are *why did it decide that*, *is it fair*, and
*is it still behaving the way it did when you approved it*. **Assurance** answers those three for
one system, or for all of them, *"in the language your auditor will accept"*.

Every result is computed from the predictions you logged, not from a template. Each tab leads with
a plain-language **verdict**, has a **How was this computed?** note, and lets analysts and
administrators **Mark reviewed**. With too little data, the tab says the result can't be computed
yet rather than estimating one.

Use **Choose a system** (or **All systems**) at the top. The tabs are **Explainability**,
**Fairness** and **Drift**.

## Explainability

!!! note "Explanations run your model"
    SHAP, LIME and DiCE explanations are computed by running the model itself, so WhiteBoxXAI needs
    the model file. Register the model through [GitHub](../integrations/github.md#configuration-file)
    with `storage_mode: full`; scikit-learn-style pickle and joblib files are supported. For a model
    registered with metadata only, the explanation is recorded as **failed**, with a message saying no
    stored model artifact is available. Drift, fairness audits and reports don't need the model file.

### Request an explanation

1. Choose a system, then a logged prediction to explain.
2. Pick a **Method**:

    | Method | Use it for |
    | --- | --- |
    | **Model default** | The method configured for this system |
    | **SHAP (kernel)** | Any model; model-agnostic Shapley values |
    | **SHAP (tree models)** | Tree ensembles such as XGBoost, LightGBM and random forests; exact and fast |
    | **LIME (tabular)** | A local linear approximation around the prediction |
    | **DiCE (counterfactuals)** | *What would have to change for a different outcome?* |

3. Request the explanation. Results appear when the computation finishes.

### Read the result

- **Contribution waterfall**: how each feature moved the prediction away from the base value
- **Force plot**: the features pushing the prediction up and down, side by side
- **Feature weights**: the ranked contributions
- **What this explanation shows**: a plain-language reading of the result

**Counterfactuals (DiCE)** answer the question an applicant or customer actually asks: *what would
have needed to be different?* Each counterfactual is an alternative input, close to the original,
that the model scores differently. From the SDK you can choose how many to generate (`total_cfs`,
default 3), the target outcome (`desired_class`, default `"opposite"`) and which features may change
(`features_to_vary`, default `"all"`). Restrict `features_to_vary` to features a person could
realistically change. Counterfactuals that vary age or protected attributes aren't useful
recourse.

```python
cf = client.explanations.generate(
    model_id,
    instance={"income": 52000, "utilisation": 0.31, "tenure_months": 18},
    method="dice",
    total_cfs=3,
    features_to_vary=["income", "utilisation"],
)
```

For how SHAP and LIME values are computed, and global feature importance, see
[Explainability Engine](features.md#explainability-engine).

## Fairness

Choose a **Protected attribute** (**All**, **Gender**, **Race/Ethnicity**, **Age**, **Disability**),
then read the sub-tabs:

| Sub-tab | Shows |
| --- | --- |
| **Fairness Metrics** | Demographic parity, equalized odds, disparate impact and the other metrics computed by the audit |
| **Group Comparison** | Outcomes and error rates per group, side by side |
| **Audit Results** | Each recorded audit, its severity, and whether it passed |
| **Recommendations** | The mitigations the audit recorded for this result |

### Where audit data comes from

A fairness audit needs true outcomes, predicted outcomes and group membership. WhiteBoxXAI doesn't
infer protected attributes from other columns. You supply group data you're entitled to use when
you run the audit:

```python
audit = client.fairness.audit(
    model_id,
    sensitive_attributes=["gender"],
    y_true=y_true,
    y_pred=y_pred,
    group_data={"gender": genders},
)
```

If you don't have that data, the audit can't be run, and the tab says so. A failed audit with
high severity can draft an entry in the [AI Risk Register](risk-register.md) automatically.

For the metric definitions and thresholds, see [Bias Detection](features.md#bias-detection).

## Drift

Choose a **drift type** and a **time range**, then read the sub-tabs:

| Sub-tab | Shows |
| --- | --- |
| **Overview** | The verdict and overall drift score for the period |
| **Feature Analysis** | Per-feature test results and which features drifted |
| **Timeline** | How drift has changed over time |
| **Details** | The test statistics behind each result |

**Refresh** re-reads the latest results. **Export Report** downloads the drift results as CSV, and
every chart can also be exported as PNG or CSV.

### How drift is measured

Each feature in a recent window of predictions is compared with the model's baseline:

- **Continuous features**: Kolmogorov–Smirnov test and Population Stability Index (PSI)
- **Categorical features**: chi-squared test and Jensen–Shannon divergence
- **Concept drift**: performance degradation against the baseline, and distribution-based methods
  including ADWIN

Sensitivity presets (`low`, `medium`, `high`) set all the thresholds together. See
[Drift Detection](features.md#drift-detection) for configuration.

### Keep a record

A quick check (`client.drift.detect`) answers *has it drifted?* without storing anything. A **drift
report** (`client.drift.create_report`) is stored with an id you can cite, appears in the
[Model Register](model-register.md) dossier, and can be turned into a **Drift Report** document in
[Reports](reports.md). Use reports for anything you'll show someone else.

High-severity drift can draft an entry in the [AI Risk Register](risk-register.md) automatically.

## Who can do what

| Action | Who |
| --- | --- |
| View results and verdicts | Everyone in the organization |
| **Mark reviewed** on a verdict | Analysts and administrators |

Requesting an explanation, running a fairness audit or creating a drift report from the SDK, REST
API or MCP server records one [Governance Event](../get-started/governance-events.md). Doing it from
the dashboard is free.
