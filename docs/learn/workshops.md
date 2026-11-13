---
description: Hands-on tutorial — train a credit model on a public dataset, register and monitor it in WhiteBoxXAI, create a drift report, run a fairness audit by age group, take it through a governance review, and generate an auditor-ready evidence pack.
---

# Tutorial: from model to evidence pack

You'll take a credit-approval model from training to the evidence an auditor asks for: a registered
system, logged predictions, a stored drift report, a fairness audit, a recorded review decision, and an
evidence pack. It takes about 45 minutes and uses the public German Credit dataset, so you don't need
data of your own.

**You need:** Python 3.10 or later, a WhiteBoxXAI account (the Free plan is enough), and an
[API key](../account/api-keys.md) in `WHITEBOX_API_KEY`.

```bash
pip install "whitebox-xai-sdk[sklearn]" pandas
```

This tutorial records roughly 15 [Governance Events](../get-started/governance-events.md) and one
Governed AI System.

## 1. Train a model

```python
import pandas as pd
from sklearn.datasets import fetch_openml
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score
from sklearn.model_selection import train_test_split

data = fetch_openml(name="credit-g", version=1, as_frame=True)
X = pd.get_dummies(data.data, drop_first=True).astype(float)
y = (data.target == "good").astype(int)
age = data.data["age"]

X_train, X_test, y_train, y_test, age_train, age_test = train_test_split(
    X, y, age, test_size=0.3, random_state=7
)
clf = RandomForestClassifier(n_estimators=200, max_depth=8, random_state=7).fit(X_train, y_train)
accuracy = accuracy_score(y_test, clf.predict(X_test))
print(f"Test accuracy: {accuracy:.3f}")
```

## 2. Register it

```python
import os

from whiteboxxai import WhiteBoxXAI

client = WhiteBoxXAI(api_key=os.environ["WHITEBOX_API_KEY"])

model = client.models.register(
    name="credit_approval_tutorial",
    model_type="random_forest",
    framework="sklearn",
    version="1.0.0",
    features=list(X.columns),
    target_variable="credit_good",
    baseline_metrics={"accuracy": round(accuracy, 4)},   # measured above, not made up
    description="Tutorial: German Credit approval model",
    tags=["tutorial"],
)
model_id = model["id"]
```

Open **Model Register** in the dashboard: the system is there with the accuracy you measured as its
baseline.

## 3. Log predictions

Send the test-set predictions as if they were production traffic. Wrapping them in `governance_run()`
groups every 100 records into one Governance Event:

```python
from whiteboxxai import governance_run

proba = clf.predict_proba(X_test)[:, 1]
records = [
    {
        "input_data": row.to_dict(),
        "output_data": {"prediction": int(p >= 0.5), "probability": float(p)},
    }
    for (_, row), p in zip(X_test.iterrows(), proba)
]

with governance_run():
    client.predictions.log_batch(model_id, predictions=records)

print(client.predictions.get_stats(model_id))
```

## 4. Check for drift and keep a record

```python
report = client.drift.create_report(model_id, window_size=200)
print(report["id"], report.get("drift_detected"), report.get("drift_score"))
```

A drift report is stored evidence with an id you can cite. With a short history, the report may say
there isn't enough data to judge drift yet. That's an honest answer, not a failure. Open
**Assurance → Drift** for the verdict in plain language and **How was this computed?**.

## 5. Audit fairness by age group

A fairness audit needs true outcomes, predicted outcomes and the group each person belongs to. Here the
group is age under 25 versus 25 and over:

```python
y_pred = (proba >= 0.5).astype(int)
groups = ["under_25" if a < 25 else "25_and_over" for a in age_test]

audit = client.fairness.audit(
    model_id,
    sensitive_attributes=["age_group"],
    y_true=y_test.tolist(),
    y_pred=y_pred.tolist(),
    group_data={"age_group": groups},
)
print(audit.get("bias_detected"), audit.get("overall_fairness_score"))
```

Open **Assurance → Fairness**. If bias was detected with high severity, a draft entry appears in the
[AI Risk Register](../user-guide/risk-register.md) for someone to own.

## 6. Take it through a review

In the dashboard:

1. **Reviews & Decisions → Boards → New board** (administrators): create a board, for example "Model
   Risk Committee", and add at least one other member.
2. **Requests → Raise a review request**: type **Model deployment**, for `credit_approval_tutorial`.
   Use **Generate AI Template** for a first draft if you like, then **Save and submit**.
3. Another board member opens the request, reads the evidence, and votes with their authenticator
   code. You can't vote on a request you raised.
4. Once quorum and the approval threshold are met, an administrator or the chair chooses **Finalize
   the decision**.

The decision goes into the immutable archive under **Decisions**. See
[Reviews & Decisions](../user-guide/governance.md).

## 7. Generate the evidence pack

```python
types = {t["id"]: t for t in client.reports.list()}
print(types["evidence_pack"])           # check it's ready

export = client.reports.generate(report_type_id="evidence_pack", format="pdf")
print(export["id"])
```

Or use **Compliance & Evidence → Evidence packs**. Then, in **Reports → History**:

- download the PDF
- record sign-off as **Preparer**, and ask a colleague to sign as **Reviewer**
- **Share with an auditor** to send a verified, expiring link

## 8. Clean up

Archive the tutorial system so it doesn't count against your plan: **Model Register →** the system **→
System settings → Archive**, or:

```python
client.models.archive(model_id)
```

## What you've built

| Evidence | Where an auditor finds it |
| --- | --- |
| A registered AI system with a measured baseline | Model Register |
| Production predictions | The system's dossier |
| A stored drift report | Assurance → Drift; the dossier's Drift tab |
| A fairness audit by age group | Assurance → Fairness |
| A board decision with votes and justification | Reviews & Decisions → Decisions |
| A signed, shareable evidence pack | Compliance & Evidence → Reports → History |

Next: [Govern an LLM application](tutorial-llm-app.md).
