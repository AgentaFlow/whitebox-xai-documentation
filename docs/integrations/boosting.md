# XGBoost & LightGBM - Quick Reference

Quick reference for monitoring gradient boosting models (XGBoost and LightGBM) with WhiteBoxXAI.

## Installation

```bash
# XGBoost
pip install whitebox-xai-sdk xgboost

# LightGBM
pip install whitebox-xai-sdk lightgbm

# Both
pip install whitebox-xai-sdk xgboost lightgbm
```

## Basic Setup

### XGBoost

```python
import xgboost as xgb
from whiteboxxai import WhiteBoxXAI
from whiteboxxai.integrations.boosting import XGBoostMonitor

client = WhiteBoxXAI(api_key="your-api-key")
monitor = XGBoostMonitor(
    client=client,
    model_name="fraud_detector",
    track_feature_importance=True,
    importance_type="gain"
)
```

### LightGBM

```python
import lightgbm as lgb
from whiteboxxai import WhiteBoxXAI
from whiteboxxai.integrations.boosting import LightGBMMonitor

client = WhiteBoxXAI(api_key="your-api-key")
monitor = LightGBMMonitor(
    client=client,
    model_name="churn_predictor",
    track_feature_importance=True,
    importance_type="gain"
)
```

## Quick Patterns

### Register and Monitor

```python
# Train model
model = xgb.XGBClassifier(n_estimators=100, max_depth=5)
model.fit(X_train, y_train)

# Register with WhiteBoxXAI
monitor.register_from_model(model, X_train, y_train)

# Make monitored predictions
predictions = monitor.predict(model, X_test, y_test)
```

### Wrapped Model (Recommended)

```python
from whiteboxxai.integrations.boosting import wrap_xgboost_model

# Wrap for automatic monitoring
wrapped = wrap_xgboost_model(model, monitor, auto_register=True)

# Predictions automatically logged
predictions = wrapped.predict(X_test)
```

## Common Use Cases

### Binary Classification (XGBoost)

```python
import xgboost as xgb

# Train model
model = xgb.XGBClassifier(
    n_estimators=100,
    max_depth=5,
    learning_rate=0.1,
    random_state=42
)
model.fit(X_train, y_train)

# Register and monitor
monitor = XGBoostMonitor(client=client, model_name="fraud_classifier")
monitor.register_from_model(model, X_train, y_train, metadata={
    'description': 'Fraud detection model',
    'dataset': 'transactions_2024'
})

# Predict with monitoring
predictions = monitor.predict(model, X_test, y_test)
```

### Regression (LightGBM)

```python
import lightgbm as lgb

# Train model
model = lgb.LGBMRegressor(
    n_estimators=100,
    max_depth=5,
    learning_rate=0.1,
    random_state=42
)
model.fit(X_train, y_train)

# Register and monitor
monitor = LightGBMMonitor(client=client, model_name="price_predictor")
monitor.register_from_model(model, X_train, y_train)

# Predict
predictions = monitor.predict(model, X_test, y_test)
```

### Feature Importance Tracking

```python
# XGBoost - different importance types
for imp_type in ['weight', 'gain', 'cover', 'total_gain', 'total_cover']:
    monitor = XGBoostMonitor(
        client=client,
        model_name=f"model_{imp_type}",
        importance_type=imp_type
    )
    monitor.register_from_model(model, X_train, y_train)

# LightGBM - different importance types
for imp_type in ['split', 'gain']:
    monitor = LightGBMMonitor(
        client=client,
        model_name=f"model_{imp_type}",
        importance_type=imp_type
    )
    monitor.register_from_model(model, X_train, y_train)
```

### Named Features

```python
import pandas as pd

# Use DataFrame for automatic feature name tracking
X_train_df = pd.DataFrame(X_train, columns=['age', 'income', 'score'])
y_train = np.array([0, 1, 1, 0])

model = xgb.XGBClassifier()
model.fit(X_train_df, y_train)

# Feature names automatically tracked
monitor = XGBoostMonitor(client=client, model_name="named_features")
monitor.register_from_model(model, X_train_df, y_train)

# Feature importance uses real names
importance = monitor._get_feature_importance(model)
# {'age': 0.45, 'income': 0.35, 'score': 0.20}
```

### Model Comparison

```python
# Train both frameworks
xgb_model = xgb.XGBClassifier(n_estimators=100)
xgb_model.fit(X_train, y_train)

lgb_model = lgb.LGBMClassifier(n_estimators=100)
lgb_model.fit(X_train, y_train)

# Monitor both
xgb_monitor = XGBoostMonitor(client=client, model_name="xgb_model")
xgb_monitor.register_from_model(xgb_model, X_train, y_train)
xgb_preds = xgb_monitor.predict(xgb_model, X_test, y_test)

lgb_monitor = LightGBMMonitor(client=client, model_name="lgb_model")
lgb_monitor.register_from_model(lgb_model, X_train, y_train)
lgb_preds = lgb_monitor.predict(lgb_model, X_test, y_test)

# Compare in WhiteBoxXAI dashboard
```

## Configuration Options

### XGBoostMonitor

```python
monitor = XGBoostMonitor(
    client=client,                        # WhiteBoxXAI client
    model_name="my_model",                # Model identifier
    track_feature_importance=True,        # Track importance
    importance_type="gain"                # Type: weight, gain, cover, total_gain, total_cover
)
```

### LightGBMMonitor

```python
monitor = LightGBMMonitor(
    client=client,                        # WhiteBoxXAI client
    model_name="my_model",                # Model identifier
    track_feature_importance=True,        # Track importance
    importance_type="gain"                # Type: split, gain
)
```

## Monitoring Methods

| Method | Purpose | Example |
|--------|---------|---------|
| `register_from_model()` | Register model | `monitor.register_from_model(model, X_train, y_train)` |
| `predict()` | Make predictions & log | `predictions = monitor.predict(model, X_test, y_test)` |
| `log_predictions()` | Log existing predictions | `monitor.log_predictions(X, predictions, actuals)` |
| `set_baseline()` | Update baseline data | `monitor.set_baseline(X_train, y_train)` |

## Wrapper Functions

| Function | Purpose | Example |
|----------|---------|---------|
| `wrap_xgboost_model()` | Wrap XGBoost for auto-logging | `wrapped = wrap_xgboost_model(model, monitor)` |
| `wrap_lightgbm_model()` | Wrap LightGBM for auto-logging | `wrapped = wrap_lightgbm_model(model, monitor)` |

## Feature Importance Types

### XGBoost

- **weight**: Number of times feature is used to split
- **gain**: Average gain when feature is used
- **cover**: Average coverage when feature is used
- **total_gain**: Total gain when feature is used
- **total_cover**: Total coverage when feature is used

### LightGBM

- **split**: Number of times feature is used to split
- **gain**: Total gain when feature is used

## Tracked Metadata

Automatically extracted and logged:

- **Framework**: xgboost or lightgbm
- **Version**: Framework version
- **Features**: Feature names (if available)
- **Num Features**: Number of features
- **Num Trees**: Number of trees/estimators
- **Parameters**: Model hyperparameters
- **Feature Importance**: Importance scores
- **Model Type**: classification, regression, or ranking

## Supported Model Types

### XGBoost

- `xgb.XGBClassifier` - Binary and multiclass classification
- `xgb.XGBRegressor` - Regression
- `xgb.XGBRanker` - Learning to rank
- `xgb.Booster` - Native XGBoost booster

### LightGBM

- `lgb.LGBMClassifier` - Binary and multiclass classification
- `lgb.LGBMRegressor` - Regression
- `lgb.LGBMRanker` - Learning to rank
- `lgb.Booster` - Native LightGBM booster

## Integration Patterns

### Pattern 1: Explicit Monitoring

```python
monitor = XGBoostMonitor(client=client, model_name="my_model")
monitor.register_from_model(model, X_train, y_train)
predictions = monitor.predict(model, X_test, y_test)
```

### Pattern 2: Wrapped Model

```python
monitor = XGBoostMonitor(client=client, model_name="my_model")
wrapped = wrap_xgboost_model(model, monitor, auto_register=True)
predictions = wrapped.predict(X_test)  # Auto-logged
```

### Pattern 3: Batch Prediction Logging

```python
# Make predictions separately
predictions = model.predict(X_test)
probabilities = model.predict_proba(X_test)

# Log later
monitor.log_predictions(
    inputs=X_test,
    predictions=predictions,
    actuals=y_test,
    probabilities=probabilities,
    metadata={'batch_id': '2024-01'}
)
```

## Best Practices

1. **Use Feature Names**
   ```python
   # ✅ Use DataFrame with named columns
   X_train_df = pd.DataFrame(X_train, columns=feature_names)
   model.fit(X_train_df, y_train)
   ```

2. **Track Feature Importance**
   ```python
   # ✅ Enable for interpretability
   monitor = XGBoostMonitor(
       client=client,
       track_feature_importance=True
   )
   ```

3. **Set Appropriate Importance Type**
   ```python
   # ✅ Use 'gain' for model quality insights
   monitor = XGBoostMonitor(importance_type="gain")
   ```

4. **Include Metadata**
   ```python
   # ✅ Add context to registrations
   monitor.register_from_model(
       model, X_train, y_train,
       metadata={
           'dataset': 'training_v2',
           'date': '2024-01-15',
           'engineer': 'data-team'
       }
   )
   ```

5. **Use Wrappers for Production**
   ```python
   # ✅ Simplifies deployment
   wrapped_model = wrap_xgboost_model(model, monitor)
   # All predictions automatically logged
   ```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Import error | `pip install xgboost` or `pip install lightgbm` |
| Feature names missing | Use pandas DataFrame with named columns |
| Importance extraction fails | Check model is trained and has feature_importances_ |
| High overhead | Set `track_feature_importance=False` |
| Version conflicts | Check XGBoost/LightGBM versions match |

## Performance Tips

1. **Disable Feature Importance for Speed**
   ```python
   monitor = XGBoostMonitor(track_feature_importance=False)
   ```

2. **Use Sampling for Large Datasets**
   ```python
   monitor = XGBoostMonitor(sampling_rate=0.1)  # Log 10% of predictions
   ```

3. **Batch Predictions**
   ```python
   # More efficient than individual predictions
   predictions = monitor.predict(model, X_test_batch, y_test_batch)
   ```

## Examples

Full examples in:
- `sdk/examples/boosting_example.py`
- `sdk/guides/BOOSTING_INTEGRATION.md`

## Resources

- XGBoost Docs: https://xgboost.readthedocs.io/
- LightGBM Docs: https://lightgbm.readthedocs.io/
- WhiteBoxXAI Docs: https://docs.whiteboxxai.com
