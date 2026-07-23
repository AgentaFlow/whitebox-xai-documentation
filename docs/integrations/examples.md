# WhiteBoxXAI Integration Examples

Comprehensive examples for integrating WhiteBoxXAI into your ML applications.

---

## Table of Contents

1. [Quick Start Examples](#quick-start-examples)
2. [Framework Integrations](#framework-integrations)
3. [Use Case Examples](#use-case-examples)
4. [Advanced Patterns](#advanced-patterns)
5. [Production Scenarios](#production-scenarios)

---

## Quick Start Examples

### Example 1: Basic Prediction Logging

```python
"""
basic_logging.py - Simplest WhiteBoxXAI integration
"""
from whiteboxxai import WhiteBoxXAI
import numpy as np

# Initialize client
client = WhiteBoxXAI(api_key="your-api-key")

# Register model
model_id = client.models.register(
    name="simple_classifier",
    version="1.0.0",
    model_type="classification"
)

print(f"Model registered: {model_id}")

# Simulate predictions
for i in range(10):
    # Your model prediction logic
    features = {
        "feature_1": np.random.rand(),
        "feature_2": np.random.rand(),
        "feature_3": np.random.randint(0, 10)
    }

    prediction = np.random.choice([0, 1])
    probability = np.random.rand()

    # Log to WhiteBoxXAI
    prediction_id = client.predictions.log(
        model_id=model_id,
        inputs=features,
        output={
            "prediction": int(prediction),
            "probability": float(probability)
        }
    )

    print(f"Logged prediction {i+1}: {prediction_id}")

print("✓ All predictions logged successfully!")
```

### Example 2: Batch Logging

```python
"""
batch_logging.py - Efficient batch prediction logging
"""
from whiteboxxai import WhiteBoxXAI
import pandas as pd
import numpy as np

client = WhiteBoxXAI(api_key="your-api-key")
model_id = "your-model-id"

# Simulate batch predictions
batch_size = 100
predictions = []

for i in range(batch_size):
    predictions.append({
        "inputs": {
            "age": np.random.randint(18, 80),
            "income": np.random.randint(20000, 200000),
            "credit_score": np.random.randint(300, 850)
        },
        "output": {
            "prediction": np.random.choice(["approved", "denied"]),
            "confidence": np.random.rand()
        },
        "metadata": {
            "application_id": f"app_{i:05d}"
        }
    })

# Log entire batch at once
print(f"Logging {len(predictions)} predictions...")
prediction_ids = client.predictions.log_batch(
    model_id=model_id,
    predictions=predictions
)

print(f"✓ Logged {len(prediction_ids)} predictions")
print(f"First prediction ID: {prediction_ids[0]}")
```

---

## Framework Integrations

### Scikit-learn

#### Example 3: Random Forest Classification

```python
"""
sklearn_random_forest.py - Monitor Random Forest classifier
"""
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, precision_score, recall_score
from whiteboxxai import WhiteBoxXAI
from whiteboxxai.integrations.sklearn import SklearnMonitor

# Generate dataset
X, y = make_classification(
    n_samples=1000,
    n_features=20,
    n_informative=15,
    n_redundant=5,
    random_state=42
)

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# Train model
print("Training Random Forest...")
model = RandomForestClassifier(
    n_estimators=100,
    max_depth=10,
    random_state=42
)
model.fit(X_train, y_train)

# Calculate baseline metrics
y_pred = model.predict(X_test)
baseline_metrics = {
    "accuracy": accuracy_score(y_test, y_pred),
    "precision": precision_score(y_test, y_pred),
    "recall": recall_score(y_test, y_pred)
}

print(f"Baseline Accuracy: {baseline_metrics['accuracy']:.3f}")

# Setup monitoring
client = WhiteBoxXAI(api_key="your-api-key")
monitor = SklearnMonitor(client, model=model)

# Register model with baseline
feature_names = [f"feature_{i}" for i in range(X.shape[1])]
model_id = monitor.register_from_model(
    name="random_forest_classifier",
    model_type="classification",
    X_train=X_train,
    y_train=y_train,
    feature_names=feature_names,
    baseline_metrics=baseline_metrics
)

print(f"✓ Model registered: {model_id}")

# Wrap model for automatic monitoring
monitored_model = monitor.wrap_model(model)

# Make predictions - automatically logged
print("Making monitored predictions...")
predictions = monitored_model.predict(X_test[:10])
probabilities = monitored_model.predict_proba(X_test[:10])

print(f"✓ Logged {len(predictions)} predictions automatically")
```

#### Example 4: Logistic Regression with Pipeline

```python
"""
sklearn_pipeline.py - Monitor sklearn pipeline
"""
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split
from whiteboxxai import WhiteBoxXAI
from whiteboxxai.integrations.sklearn import SklearnMonitor

# Load data
data = load_breast_cancer()
X_train, X_test, y_train, y_test = train_test_split(
    data.data, data.target, test_size=0.2, random_state=42
)

# Create pipeline
pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('classifier', LogisticRegression(random_state=42))
])

print("Training pipeline...")
pipeline.fit(X_train, y_train)

# Setup monitoring
client = WhiteBoxXAI(api_key="your-api-key")
monitor = SklearnMonitor(client, model=pipeline)

# Register - monitor automatically handles pipeline
model_id = monitor.register_from_model(
    name="logistic_regression_pipeline",
    model_type="classification",
    X_train=X_train,
    y_train=y_train,
    feature_names=data.feature_names.tolist()
)

print(f"✓ Pipeline registered: {model_id}")

# Monitor predictions
monitored_pipeline = monitor.wrap_model(pipeline)
predictions = monitored_pipeline.predict(X_test[:20])

print(f"✓ Logged {len(predictions)} predictions from pipeline")
```

### PyTorch

#### Example 5: Neural Network Classification

```python
"""
pytorch_nn.py - Monitor PyTorch neural network
"""
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader, TensorDataset
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from whiteboxxai import WhiteBoxXAI
from whiteboxxai.integrations.pytorch import TorchMonitor

# Generate dataset
X, y = make_classification(n_samples=1000, n_features=20, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# Convert to tensors
X_train_tensor = torch.FloatTensor(X_train)
y_train_tensor = torch.LongTensor(y_train)
X_test_tensor = torch.FloatTensor(X_test)

# Define model
class SimpleNN(nn.Module):
    def __init__(self, input_dim):
        super(SimpleNN, self).__init__()
        self.fc1 = nn.Linear(input_dim, 64)
        self.fc2 = nn.Linear(64, 32)
        self.fc3 = nn.Linear(32, 2)
        self.relu = nn.ReLU()
        self.softmax = nn.Softmax(dim=1)

    def forward(self, x):
        x = self.relu(self.fc1(x))
        x = self.relu(self.fc2(x))
        x = self.fc3(x)
        return x

# Train model
print("Training neural network...")
model = SimpleNN(input_dim=20)
criterion = nn.CrossEntropyLoss()
optimizer = optim.Adam(model.parameters(), lr=0.001)

# Training loop (simplified)
dataset = TensorDataset(X_train_tensor, y_train_tensor)
dataloader = DataLoader(dataset, batch_size=32, shuffle=True)

model.train()
for epoch in range(10):
    for batch_X, batch_y in dataloader:
        optimizer.zero_grad()
        outputs = model(batch_X)
        loss = criterion(outputs, batch_y)
        loss.backward()
        optimizer.step()

print("✓ Training complete")

# Setup monitoring
client = WhiteBoxXAI(api_key="your-api-key")
monitor = TorchMonitor(client, model=model, device="cpu")

# Register model
model_id = monitor.register_from_model(
    name="pytorch_classifier",
    model_type="classification",
    input_shape=(20,),
    output_classes=2
)

print(f"✓ Model registered: {model_id}")

# Wrap for automatic monitoring
monitored_model = monitor.wrap_model(model)

# Make predictions
model.eval()
with torch.no_grad():
    outputs = monitored_model(X_test_tensor[:10])
    predictions = torch.argmax(outputs, dim=1)

print(f"✓ Logged {len(predictions)} predictions")
```

#### Example 6: CNN Image Classification

```python
"""
pytorch_cnn.py - Monitor CNN for image classification
"""
import torch
import torch.nn as nn
from torchvision import datasets, transforms
from torch.utils.data import DataLoader
from whiteboxxai import WhiteBoxXAI
from whiteboxxai.integrations.pytorch import TorchMonitor

# Define CNN
class SimpleCNN(nn.Module):
    def __init__(self):
        super(SimpleCNN, self).__init__()
        self.conv1 = nn.Conv2d(1, 32, kernel_size=3)
        self.conv2 = nn.Conv2d(32, 64, kernel_size=3)
        self.fc1 = nn.Linear(64 * 5 * 5, 128)
        self.fc2 = nn.Linear(128, 10)
        self.pool = nn.MaxPool2d(2, 2)
        self.relu = nn.ReLU()

    def forward(self, x):
        x = self.pool(self.relu(self.conv1(x)))
        x = self.pool(self.relu(self.conv2(x)))
        x = x.view(-1, 64 * 5 * 5)
        x = self.relu(self.fc1(x))
        x = self.fc2(x)
        return x

# Load MNIST (example)
transform = transforms.Compose([
    transforms.ToTensor(),
    transforms.Normalize((0.1307,), (0.3081,))
])

# Assume model is trained
model = SimpleCNN()
model.load_state_dict(torch.load("mnist_cnn.pth", weights_only=True))
model.eval()

# Setup monitoring
client = WhiteBoxXAI(api_key="your-api-key")
monitor = TorchMonitor(client, model=model, device="cpu")

# Register
model_id = monitor.register_from_model(
    name="mnist_cnn",
    model_type="classification",
    input_shape=(1, 28, 28),
    output_classes=10,
    metadata={
        "dataset": "MNIST",
        "architecture": "CNN",
        "layers": 4
    }
)

print(f"✓ CNN registered: {model_id}")

# Monitor predictions
test_data = datasets.MNIST("./data", train=False, download=True, transform=transform)
test_loader = DataLoader(test_data, batch_size=32)

monitored_model = monitor.wrap_model(model)

with torch.no_grad():
    for images, labels in test_loader:
        outputs = monitored_model(images)
        break  # Just first batch for demo

print("✓ Logged batch predictions")
```

---

## Use Case Examples

### Example 7: Fraud Detection

```python
"""
fraud_detection.py - Real-world fraud detection monitoring
"""
from whiteboxxai import WhiteBoxXAI, ModelMonitor
import pandas as pd
import joblib

# Load trained model
model = joblib.load("fraud_model.pkl")

# Initialize monitoring
client = WhiteBoxXAI(api_key="your-api-key")
monitor = ModelMonitor(client)

# Register with detailed metadata
model_id = monitor.register_model(
    name="fraud_detection_v2",
    version="2.1.0",
    model_type="classification",
    framework="xgboost",
    description="Detects fraudulent credit card transactions",
    features=[
        "transaction_amount",
        "merchant_category",
        "distance_from_home",
        "time_since_last_transaction",
        "num_transactions_24h",
        "avg_transaction_amount",
        "is_international"
    ],
    target="is_fraud",
    baseline_metrics={
        "accuracy": 0.94,
        "precision": 0.91,
        "recall": 0.89,
        "f1_score": 0.90
    },
    tags=["production", "fraud", "real-time", "high-stakes"],
    metadata={
        "business_impact": "high",
        "sla_latency_ms": 100,
        "owner": "fraud-team@company.com",
        "model_path": "s3://models/fraud-v2.1.0.pkl"
    }
)

# Production prediction function
def predict_fraud(transaction_data):
    """
    Make fraud prediction with monitoring.

    Args:
        transaction_data: Dict with transaction features

    Returns:
        Dict with prediction and metadata
    """
    # Extract features
    features = {
        "transaction_amount": transaction_data["amount"],
        "merchant_category": transaction_data["merchant_category"],
        "distance_from_home": transaction_data["distance"],
        "time_since_last_transaction": transaction_data["time_since_last"],
        "num_transactions_24h": transaction_data["num_recent"],
        "avg_transaction_amount": transaction_data["avg_amount"],
        "is_international": transaction_data["is_international"]
    }

    # Make prediction
    prediction = model.predict([list(features.values())])[0]
    probability = model.predict_proba([list(features.values())])[0][1]

    # Log to WhiteBoxXAI
    prediction_id = monitor.log_prediction(
        inputs=features,
        output={
            "prediction": "fraud" if prediction == 1 else "legitimate",
            "fraud_probability": float(probability),
            "risk_level": "high" if probability > 0.8 else "medium" if probability > 0.5 else "low"
        },
        metadata={
            "transaction_id": transaction_data["transaction_id"],
            "customer_id": transaction_data["customer_id"],
            "merchant_id": transaction_data["merchant_id"],
            "timestamp": transaction_data["timestamp"],
            "channel": transaction_data["channel"]
        }
    )

    return {
        "is_fraud": bool(prediction),
        "probability": probability,
        "prediction_id": prediction_id
    }

# Example usage
transaction = {
    "transaction_id": "txn_123456",
    "customer_id": "cust_789",
    "merchant_id": "merch_456",
    "amount": 1250.00,
    "merchant_category": "electronics",
    "distance": 150.5,
    "time_since_last": 2.5,
    "num_recent": 3,
    "avg_amount": 85.50,
    "is_international": 0,
    "timestamp": "2025-12-05T14:30:00Z",
    "channel": "online"
}

result = predict_fraud(transaction)
print(f"Fraud Probability: {result['probability']:.2%}")
print(f"Prediction logged: {result['prediction_id']}")
```

### Example 8: Credit Scoring

```python
"""
credit_scoring.py - Credit approval with fairness monitoring
"""
from whiteboxxai import WhiteBoxXAI, ModelMonitor
import pandas as pd

client = WhiteBoxXAI(api_key="your-api-key")
monitor = ModelMonitor(client)

# Register model with fairness focus
model_id = monitor.register_model(
    name="credit_approval_model",
    version="3.0.0",
    model_type="classification",
    features=[
        "income",
        "employment_length",
        "credit_score",
        "debt_to_income_ratio",
        "loan_amount",
        "loan_purpose"
    ],
    target="approved",
    baseline_metrics={
        "accuracy": 0.87,
        "precision": 0.84,
        "recall": 0.81
    },
    tags=["production", "credit", "regulated"],
    metadata={
        "regulatory_compliance": ["ECOA", "FCRA"],
        "fairness_audited": True,
        "last_audit_date": "2025-11-15"
    }
)

# Prediction with explanation generation
def approve_credit(application):
    """
    Credit approval decision with explainability.
    """
    features = {
        "income": application["annual_income"],
        "employment_length": application["years_employed"],
        "credit_score": application["credit_score"],
        "debt_to_income_ratio": application["debt_ratio"],
        "loan_amount": application["requested_amount"],
        "loan_purpose": application["purpose"]
    }

    # Make prediction (using your model)
    prediction = model.predict([list(features.values())])[0]
    probability = model.predict_proba([list(features.values())])[0][1]

    # Log prediction
    prediction_id = monitor.log_prediction(
        inputs=features,
        output={
            "decision": "approved" if prediction == 1 else "denied",
            "approval_probability": float(probability)
        },
        metadata={
            "application_id": application["id"],
            "applicant_id": application["applicant_id"]
        }
    )

    # Generate explanation for denied applications
    explanation = None
    if prediction == 0:
        explanation = client.explanations.generate(
            prediction_id=prediction_id,
            method="shap"
        )

    return {
        "approved": bool(prediction),
        "probability": probability,
        "explanation": explanation,
        "prediction_id": prediction_id
    }

# Example
application = {
    "id": "app_12345",
    "applicant_id": "applicant_789",
    "annual_income": 65000,
    "years_employed": 5,
    "credit_score": 680,
    "debt_ratio": 0.35,
    "requested_amount": 25000,
    "purpose": "home_improvement"
}

result = approve_credit(application)
print(f"Decision: {'Approved' if result['approved'] else 'Denied'}")
if result['explanation']:
    print(f"Top factors: {list(result['explanation']['feature_contributions'].keys())[:3]}")
```

### Example 9: Customer Churn Prediction

```python
"""
churn_prediction.py - Customer churn with drift monitoring
"""
from whiteboxxai import WhiteBoxXAI, ModelMonitor
import pandas as pd
import numpy as np

client = WhiteBoxXAI(api_key="your-api-key")
monitor = ModelMonitor(client)

# Register model
model_id = monitor.register_model(
    name="churn_predictor",
    version="1.5.0",
    model_type="classification",
    features=[
        "account_age_days",
        "total_purchases",
        "avg_purchase_value",
        "days_since_last_purchase",
        "customer_service_calls",
        "satisfaction_score",
        "subscription_tier"
    ],
    target="will_churn"
)

# Enable drift detection
monitor.enable_drift_detection(
    threshold=0.15,
    window_size=1000
)

# Prediction function with drift check
def predict_churn(customer_data):
    """
    Predict customer churn with drift monitoring.
    """
    features = {
        "account_age_days": customer_data["account_age"],
        "total_purchases": customer_data["total_purchases"],
        "avg_purchase_value": customer_data["avg_value"],
        "days_since_last_purchase": customer_data["days_since_last"],
        "customer_service_calls": customer_data["support_calls"],
        "satisfaction_score": customer_data["satisfaction"],
        "subscription_tier": customer_data["tier"]
    }

    # Make prediction
    prediction = model.predict([list(features.values())])[0]
    churn_probability = model.predict_proba([list(features.values())])[0][1]

    # Log prediction
    prediction_id = monitor.log_prediction(
        inputs=features,
        output={
            "will_churn": bool(prediction),
            "churn_probability": float(churn_probability),
            "risk_level": "high" if churn_probability > 0.7 else "medium" if churn_probability > 0.4 else "low"
        },
        metadata={
            "customer_id": customer_data["customer_id"],
            "cohort": customer_data["cohort"],
            "region": customer_data["region"]
        }
    )

    # Check for drift periodically
    if np.random.random() < 0.01:  # 1% of predictions
        recent_data = get_recent_predictions()  # Your function
        drift_result = client.drift.detect(
            model_id=model_id,
            data=recent_data
        )

        if drift_result["severity"] == "high":
            print(f"⚠️  High drift detected! Score: {drift_result['overall_score']:.3f}")

    return {
        "will_churn": bool(prediction),
        "probability": churn_probability,
        "prediction_id": prediction_id
    }
```

---

## Advanced Patterns

### Example 10: A/B Testing with Monitoring

```python
"""
ab_testing.py - Monitor A/B test variants
"""
from whiteboxxai import WhiteBoxXAI, ModelMonitor
import random

client = WhiteBoxXAI(api_key="your-api-key")

# Register both model variants
monitor_a = ModelMonitor(client)
model_a_id = monitor_a.register_model(
    name="recommendation_model_a",
    version="1.0.0",
    model_type="classification",
    tags=["ab-test", "variant-a"]
)

monitor_b = ModelMonitor(client)
model_b_id = monitor_b.register_model(
    name="recommendation_model_b",
    version="1.0.0",
    model_type="classification",
    tags=["ab-test", "variant-b"]
)

def predict_with_ab_test(user_features, user_id):
    """
    Make prediction using A/B test assignment.
    """
    # Assign to variant (50/50 split)
    variant = "A" if hash(user_id) % 2 == 0 else "B"

    # Select model and monitor
    if variant == "A":
        model = model_a
        monitor = monitor_a
    else:
        model = model_b
        monitor = monitor_b

    # Make prediction
    prediction = model.predict([user_features])

    # Log with variant metadata
    prediction_id = monitor.log_prediction(
        inputs={"features": user_features},
        output={"recommendation": prediction[0]},
        metadata={
            "user_id": user_id,
            "variant": variant,
            "ab_test_id": "recommendation_v1_v2"
        }
    )

    return {
        "recommendation": prediction[0],
        "variant": variant,
        "prediction_id": prediction_id
    }

# Analyze results
def compare_variants():
    """
    Compare A/B test performance.
    """
    metrics_a = client.metrics.get(
        model_id=model_a_id,
        metric_names=["accuracy", "precision"],
        date_range="last_7_days"
    )

    metrics_b = client.metrics.get(
        model_id=model_b_id,
        metric_names=["accuracy", "precision"],
        date_range="last_7_days"
    )

    print("Variant A:")
    print(f"  Accuracy: {metrics_a['accuracy']:.3f}")
    print(f"  Precision: {metrics_a['precision']:.3f}")

    print("Variant B:")
    print(f"  Accuracy: {metrics_b['accuracy']:.3f}")
    print(f"  Precision: {metrics_b['precision']:.3f}")

    # Determine winner
    if metrics_b['accuracy'] > metrics_a['accuracy'] * 1.02:  # 2% lift
        print("✓ Variant B wins!")
    else:
        print("✓ Variant A wins (control)")
```

### Example 11: Multi-Model Ensemble

```python
"""
ensemble_monitoring.py - Monitor ensemble of models
"""
from whiteboxxai import WhiteBoxXAI, ModelMonitor
import numpy as np

client = WhiteBoxXAI(api_key="your-api-key")

# Register ensemble components
monitors = {}
for model_name in ["random_forest", "xgboost", "neural_net"]:
    monitor = ModelMonitor(client)
    model_id = monitor.register_model(
        name=f"{model_name}_component",
        version="1.0.0",
        model_type="classification",
        tags=["ensemble", "component"]
    )
    monitors[model_name] = monitor

# Register ensemble itself
ensemble_monitor = ModelMonitor(client)
ensemble_id = ensemble_monitor.register_model(
    name="ensemble_model",
    version="1.0.0",
    model_type="classification",
    tags=["ensemble", "production"],
    metadata={
        "components": list(monitors.keys()),
        "aggregation": "weighted_average"
    }
)

def ensemble_predict(features):
    """
    Make ensemble prediction and log all components.
    """
    # Get predictions from all models
    predictions = {}
    for name, monitor in monitors.items():
        pred = models[name].predict_proba([features])[0][1]
        predictions[name] = pred

        # Log component prediction
        monitor.log_prediction(
            inputs={"features": features},
            output={"probability": float(pred)}
        )

    # Weighted ensemble
    weights = {"random_forest": 0.4, "xgboost": 0.4, "neural_net": 0.2}
    ensemble_pred = sum(predictions[name] * weights[name] for name in predictions)

    # Log ensemble prediction
    prediction_id = ensemble_monitor.log_prediction(
        inputs={"features": features},
        output={
            "probability": float(ensemble_pred),
            "prediction": int(ensemble_pred > 0.5),
            "component_predictions": predictions
        }
    )

    return {
        "probability": ensemble_pred,
        "components": predictions,
        "prediction_id": prediction_id
    }
```

---

## Production Scenarios

### Example 12: High-Throughput Service

```python
"""
high_throughput.py - Production service with streaming
"""
from whiteboxxai import WhiteBoxXAI, ModelMonitor
from queue import Queue
import threading
import time

class ProductionMLService:
    def __init__(self, model, api_key, model_id):
        self.model = model
        self.client = WhiteBoxXAI(api_key=api_key)
        self.monitor = ModelMonitor(self.client, model_id=model_id)

        # Buffering for batch logging
        self.buffer = Queue(maxsize=1000)
        self.flush_interval = 10  # seconds
        self.batch_size = 100

        # Start background flush thread
        self._start_flush_thread()

    def predict(self, features):
        """
        Make prediction with async logging.
        """
        # Make prediction (fast)
        prediction = self.model.predict([features])[0]
        probability = self.model.predict_proba([features])[0][1]

        # Add to buffer (non-blocking)
        try:
            self.buffer.put_nowait({
                "inputs": {"features": features},
                "output": {
                    "prediction": int(prediction),
                    "probability": float(probability)
                }
            })
        except:
            # Buffer full, skip logging (graceful degradation)
            pass

        return {
            "prediction": int(prediction),
            "probability": float(probability)
        }

    def _start_flush_thread(self):
        """
        Background thread to flush buffer periodically.
        """
        def flush_worker():
            while True:
                time.sleep(self.flush_interval)
                self._flush_buffer()

        thread = threading.Thread(target=flush_worker, daemon=True)
        thread.start()

    def _flush_buffer(self):
        """
        Flush buffered predictions to API.
        """
        predictions = []
        while not self.buffer.empty() and len(predictions) < self.batch_size:
            predictions.append(self.buffer.get())

        if predictions:
            try:
                self.monitor.log_batch(predictions)
                print(f"✓ Flushed {len(predictions)} predictions")
            except Exception as e:
                print(f"⚠️  Failed to flush: {e}")

# Usage
service = ProductionMLService(
    model=trained_model,
    api_key="your-api-key",
    model_id="your-model-id"
)

# Handle high-throughput requests
for i in range(10000):
    features = generate_features()  # Your function
    result = service.predict(features)
    # Prediction returns immediately, logging happens in background
```

### Example 13: Microservice with Health Checks

```python
"""
microservice.py - Flask microservice with monitoring
"""
from flask import Flask, request, jsonify
from whiteboxxai import WhiteBoxXAI, ModelMonitor
import joblib

app = Flask(__name__)

# Load model
model = joblib.load("model.pkl")

# Setup monitoring
client = WhiteBoxXAI(api_key="your-api-key")
monitor = ModelMonitor(client, model_id="your-model-id")

@app.route("/predict", methods=["POST"])
def predict():
    """
    Prediction endpoint with monitoring.
    """
    try:
        data = request.get_json()
        features = data["features"]

        # Make prediction
        prediction = model.predict([list(features.values())])[0]
        probability = model.predict_proba([list(features.values())])[0][1]

        # Log to WhiteBoxXAI (async)
        try:
            prediction_id = monitor.log_prediction(
                inputs=features,
                output={
                    "prediction": int(prediction),
                    "probability": float(probability)
                },
                metadata={
                    "request_id": request.headers.get("X-Request-ID"),
                    "client_ip": request.remote_addr
                }
            )
        except Exception as e:
            # Don't fail request if logging fails
            print(f"Logging failed: {e}")
            prediction_id = None

        return jsonify({
            "prediction": int(prediction),
            "probability": float(probability),
            "prediction_id": prediction_id
        })

    except Exception as e:
        return jsonify({"error": str(e)}), 400

@app.route("/health", methods=["GET"])
def health():
    """
    Health check including WhiteBoxXAI connectivity.
    """
    health_status = {
        "service": "ok",
        "model": "ok",
        "whiteboxxai": "unknown"
    }

    # Check WhiteBoxXAI connectivity
    try:
        client.models.get(monitor.model_id)
        health_status["whiteboxxai"] = "ok"
    except Exception as e:
        health_status["whiteboxxai"] = f"error: {str(e)}"

    status_code = 200 if all(v == "ok" for v in health_status.values()) else 503
    return jsonify(health_status), status_code

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

---

## Additional Resources

- **SDK Documentation:** `/docs/SDK_DOCUMENTATION.md`
- **API Reference:** `/docs/API_REFERENCE.md`
- **Best Practices:** `/docs/BEST_PRACTICES.md`
- **Example Code:** `/examples` directory

---

*Last Updated: December 5, 2025*
