# TensorFlow/Keras Integration - Quick Reference

## Installation

```bash
pip install whitebox-xai-sdk[tensorflow]
```

## Basic Setup

```python
from tensorflow import keras
from whiteboxxai import WhiteBoxXAI
from whiteboxxai.integrations.tensorflow import KerasMonitor, WhiteBoxXAICallback

# Initialize client
client = WhiteBoxXAI(api_key="your-api-key")

# Create monitor
monitor = KerasMonitor(
    client=client,
    model=your_model,
    model_name="my_keras_model",
    model_type="classification"  # or "regression"
)

# Register model
model_id = monitor.register_from_model(version="1.0.0")
```

## Training with Monitoring

```python
# Create callback
callback = WhiteBoxXAICallback(
    monitor=monitor,
    log_frequency=1,  # Log every epoch
    log_validation=True
)

# Train
model.fit(
    X_train, y_train,
    validation_split=0.2,
    callbacks=[callback],
    epochs=50
)
```

## Making Predictions

```python
# With automatic logging
predictions = monitor.predict(X_test, log=True)

# With actual values
predictions = monitor.predict(X_test, log=True, actuals=y_test)

# Without logging
predictions = monitor.predict(X_test, log=False)
```

## Baseline for Drift Detection

```python
monitor.set_baseline(X_train, y_train)
```

## Wrapping Existing Models

```python
from whiteboxxai.integrations.tensorflow import wrap_keras_model

# Wrap model for automatic logging
wrapped_model = wrap_keras_model(model, monitor)

# All predictions now automatically logged
predictions = wrapped_model.predict(X_test)
```

## Logging Metrics

```python
# Log epoch metrics
monitor.log_epoch(
    epoch=10,
    train_loss=0.5,
    val_loss=0.6,
    accuracy=0.85
)

# Log checkpoint
monitor.log_checkpoint(
    epoch=10,
    checkpoint_path="checkpoints/model_10.h5",
    metrics={'accuracy': 0.9}
)
```

## SavedModel Support

```python
# Save model
model.save('saved_models/my_model')

# Register SavedModel
monitor.register_saved_model(
    model_path='saved_models/my_model',
    metadata={'version': '1.0', 'format': 'SavedModel'}
)
```

## Complete Example

```python
import numpy as np
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from tensorflow import keras
from whiteboxxai import WhiteBoxXAI
from whiteboxxai.integrations.tensorflow import KerasMonitor, WhiteBoxXAICallback

# Generate data
X, y = make_classification(n_samples=1000, n_features=20, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)

# Standardize
scaler = StandardScaler()
X_train = scaler.fit_transform(X_train)
X_test = scaler.transform(X_test)

# Build model
model = keras.Sequential([
    keras.layers.Dense(64, activation='relu', input_shape=(20,)),
    keras.layers.Dropout(0.3),
    keras.layers.Dense(32, activation='relu'),
    keras.layers.Dense(1, activation='sigmoid')
])

model.compile(optimizer='adam', loss='binary_crossentropy', metrics=['accuracy'])

# Setup monitoring
client = WhiteBoxXAI(api_key='your-api-key')
monitor = KerasMonitor(client, model=model, model_name="demo_model")
monitor.register_from_model(model_type="classification")
monitor.set_baseline(X_train, y_train)

# Train with monitoring
callback = WhiteBoxXAICallback(monitor)
model.fit(X_train, y_train, validation_split=0.2,
          callbacks=[callback], epochs=50)

# Predict and log
predictions = monitor.predict(X_test, log=True, actuals=y_test)

print(f"Model registered with ID: {monitor.model_id}")
```

## Common Patterns

### Classification Model
```python
model = keras.Sequential([
    keras.layers.Dense(64, activation='relu'),
    keras.layers.Dense(num_classes, activation='softmax')
])
model.compile(optimizer='adam', loss='categorical_crossentropy')

monitor = KerasMonitor(client, model=model, model_type="classification")
```

### Regression Model
```python
model = keras.Sequential([
    keras.layers.Dense(64, activation='relu'),
    keras.layers.Dense(1)
])
model.compile(optimizer='adam', loss='mse')

monitor = KerasMonitor(client, model=model, model_type="regression")
```

### CNN for Images
```python
model = keras.Sequential([
    keras.layers.Conv2D(32, 3, activation='relu', input_shape=(28, 28, 1)),
    keras.layers.MaxPooling2D(),
    keras.layers.Flatten(),
    keras.layers.Dense(10, activation='softmax')
])

monitor = KerasMonitor(client, model=model, model_type="classification")
```

### LSTM for Time Series
```python
model = keras.Sequential([
    keras.layers.LSTM(64, return_sequences=True, input_shape=(timesteps, features)),
    keras.layers.LSTM(32),
    keras.layers.Dense(1)
])

monitor = KerasMonitor(client, model=model, model_type="regression")
```

## Advanced Features

### Multi-GPU Training
```python
strategy = tf.distribute.MirroredStrategy()

with strategy.scope():
    model = create_model()
    monitor = KerasMonitor(client, model=model)

model.fit(..., callbacks=[WhiteBoxXAICallback(monitor)])
```

### Custom Metrics Callback
```python
class MetricsCallback(keras.callbacks.Callback):
    def __init__(self, monitor):
        self.monitor = monitor

    def on_epoch_end(self, epoch, logs=None):
        lr = float(keras.backend.get_value(self.model.optimizer.lr))
        self.monitor.log_epoch(epoch=epoch, learning_rate=lr, **logs)
```

### Mixed Precision Training
```python
from tensorflow.keras import mixed_precision

mixed_precision.set_global_policy('mixed_float16')
model = create_model()
monitor = KerasMonitor(client, model=model)
```

## Troubleshooting

### Issue: TensorFlow not found
```bash
pip install tensorflow>=2.10.0
```

### Issue: Model not registered automatically
```python
# Explicitly register before predictions
monitor.register_from_model()
predictions = monitor.predict(X_test)
```

### Issue: Callback not logging
```python
# Ensure monitor is registered
callback = WhiteBoxXAICallback(monitor)
# Check log_frequency setting
callback = WhiteBoxXAICallback(monitor, log_frequency=1)
```

### Issue: Memory issues
```python
# Reduce batch size or enable mixed precision
mixed_precision.set_global_policy('mixed_float16')
```

## Best Practices

1. **Always set baseline** before production deployment
2. **Use callbacks** during training for automatic logging
3. **Log validation data** to track model performance
4. **Save checkpoints** at regular intervals
5. **Version your models** using the version parameter
6. **Use descriptive names** for easy identification
7. **Monitor GPU memory** in production
8. **Enable async logging** for high-throughput scenarios

## Resources

- [WhiteBoxXAI SDK Documentation](../sdk/index.md)
- [API Reference](../sdk/api-reference.md)
- [TensorFlow Documentation](https://www.tensorflow.org/api_docs)
