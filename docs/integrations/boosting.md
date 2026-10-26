---
description: Monitor XGBoost and LightGBM models with WhiteBoxXAI — register the model, wrap predict and predict_proba so every prediction is logged, and track feature importance.
---

# XGBoost & LightGBM

`XGBoostMonitor` and `LightGBMMonitor` log your gradient-boosted model's predictions to WhiteBoxXAI,
so drift, explanations (SHAP's fast tree explainer) and fairness audits can be computed from real
traffic. Feature importance is recorded alongside.

## Install

```bash
pip install "whitebox-xai-sdk[xgboost]"     # or [lightgbm]
```

## Register, then wrap

Register the model with the client, then attach the monitor to it by id and wrap the model:

```python
import os

import xgboost as xgb
from sklearn.metrics import accuracy_score
from whiteboxxai import WhiteBoxXAI
from whiteboxxai.integrations.boosting import XGBoostMonitor, wrap_xgboost_model

client = WhiteBoxXAI(api_key=os.environ["WHITEBOX_API_KEY"])

model = xgb.XGBClassifier(n_estimators=200, max_depth=6).fit(X_train, y_train)

registered = client.models.register(
    name="fraud_detection",
    model_type="xgboost",
    framework="xgboost",
    version="3.1.0",
    features=list(X_train.columns),
    baseline_metrics={"accuracy": accuracy_score(y_test, model.predict(X_test))},
)

monitor = XGBoostMonitor(client, model_name="fraud_detection", model_id=registered["id"])
model = wrap_xgboost_model(model, monitor)

model.predict(X_live)          # logged
model.predict_proba(X_live)    # logged
```

LightGBM works the same way with `LightGBMMonitor` and `wrap_lightgbm_model`. Use `model_type="custom"`
and `framework="lightgbm"` when registering.

!!! warning "Register explicitly"
    Pass `model_id` from `client.models.register(...)` to the monitor, as above. Don't rely on
    `register_from_model()` or the wrapper's automatic registration in SDK 1.3.0. They don't send the
    `version`, `features` and `baseline_metrics` fields the API requires, so registration is refused.

## Log without wrapping

If you'd rather not replace the model's methods, call the monitor's `predict` yourself:

```python
predictions = monitor.predict(model, X_live, metadata={"batch": "2026-10-27"})
```

Pass `y_true=` when you already know the outcomes, for example when backfilling a labelled batch.

## Feature importance

The monitor records the model's feature importance with its predictions. Choose the measure with
`importance_type`: `weight`, `gain` (the default), `cover`, `total_gain` or `total_cover`. Turn it off
with `track_feature_importance=False`.

```python
monitor = XGBoostMonitor(client, model_name="fraud_detection", model_id=registered["id"],
                         importance_type="total_gain")
```

Feature importance describes the model as a whole. For why a single prediction came out the way it
did, request an explanation: SHAP's tree method is exact and fast for boosted trees.

```python
client.explanations.generate(registered["id"], instance=X_live.iloc[0].to_dict(), method="shap")
```

## Next steps

- [Assurance](../user-guide/assurance.md): drift, explanations and fairness for this model in the dashboard
- [GitHub Diff Reviews](github-diff.md): compare model versions
- [Python SDK](../sdk/index.md)
- Example: [`boosting_example.py`](https://github.com/AgentaFlow/whitebox-python-sdk/blob/main/examples/boosting_example.py) in the public SDK repository
