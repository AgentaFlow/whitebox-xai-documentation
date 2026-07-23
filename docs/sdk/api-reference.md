# WhiteBoxXAI API Reference

Complete API documentation for the WhiteBoxXAI platform.

**Base URL:** `https://api.whiteboxxai.com/api/v1`

---

## Table of Contents

1. [Authentication](#authentication)
2. [Models](#models)
3. [Predictions](#predictions)
4. [Metrics](#metrics)
5. [Drift Detection](#drift-detection)
6. [Explainability (XAI)](#explainability-xai)
7. [Bias & Fairness](#bias--fairness)
8. [LLM Monitoring](#llm-monitoring)
9. [Alerts](#alerts)
10. [Reports](#reports)
11. [Users & RBAC](#users--rbac)
12. [Dashboard](#dashboard)
13. [Error Handling](#error-handling)

---

## Authentication

### Overview
WhiteBoxXAI uses JWT (JSON Web Token) based authentication. All API requests (except login/register) require a valid JWT token in the `Authorization` header.

### Login
```http
POST /auth/login
Content-Type: application/json

{
  "username": "user@example.com",
  "password": "your_password"
}
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "bearer",
  "expires_in": 3600,
  "user": {
    "id": "uuid",
    "username": "user@example.com",
    "role": "admin"
  }
}
```

### Register
```http
POST /auth/register
Content-Type: application/json

{
  "username": "user@example.com",
  "email": "user@example.com",
  "password": "secure_password",
  "full_name": "John Doe"
}
```

### Using Authentication
Include the JWT token in all subsequent requests:

```http
GET /models
Authorization: Bearer ...
```

---

## Models

### Register Model
Register a new ML model for monitoring.

```http
POST /models
Authorization: Bearer {token}
Content-Type: application/json

{
  "name": "fraud_detection_v1",
  "description": "Credit card fraud detection model",
  "model_type": "classification",
  "framework": "scikit-learn",
  "version": "1.0.0",
  "metadata": {
    "algorithm": "random_forest",
    "features": ["amount", "merchant_category", "time_of_day"],
    "target": "is_fraud",
    "training_samples": 100000
  },
  "baseline_metrics": {
    "accuracy": 0.95,
    "precision": 0.92,
    "recall": 0.88,
    "f1_score": 0.90
  }
}
```

**Response (201 Created):**
```json
{
  "id": "model_uuid",
  "name": "fraud_detection_v1",
  "model_type": "classification",
  "framework": "scikit-learn",
  "version": "1.0.0",
  "status": "active",
  "created_at": "2025-12-05T10:00:00Z",
  "monitoring_enabled": true
}
```

### List Models
```http
GET /models?status=active&model_type=classification&page=1&page_size=20
Authorization: Bearer {token}
```

**Query Parameters:**
- `status` (optional): Filter by status (active, inactive, archived)
- `model_type` (optional): Filter by type (classification, regression, clustering, llm)
- `framework` (optional): Filter by framework
- `page` (optional): Page number (default: 1)
- `page_size` (optional): Items per page (default: 20, max: 100)

**Response (200 OK):**
```json
{
  "total": 45,
  "page": 1,
  "page_size": 20,
  "models": [
    {
      "id": "model_uuid",
      "name": "fraud_detection_v1",
      "model_type": "classification",
      "framework": "scikit-learn",
      "version": "1.0.0",
      "status": "active",
      "created_at": "2025-12-05T10:00:00Z",
      "last_prediction": "2025-12-05T15:30:00Z",
      "prediction_count": 15420
    }
  ]
}
```

### Get Model Details
```http
GET /models/{model_id}
Authorization: Bearer {token}
```

**Response (200 OK):**
```json
{
  "id": "model_uuid",
  "name": "fraud_detection_v1",
  "description": "Credit card fraud detection model",
  "model_type": "classification",
  "framework": "scikit-learn",
  "version": "1.0.0",
  "status": "active",
  "metadata": {
    "algorithm": "random_forest",
    "features": ["amount", "merchant_category", "time_of_day"]
  },
  "baseline_metrics": {
    "accuracy": 0.95,
    "precision": 0.92
  },
  "monitoring_config": {
    "sampling_rate": 0.1,
    "drift_threshold": 0.05
  },
  "created_at": "2025-12-05T10:00:00Z",
  "updated_at": "2025-12-05T15:30:00Z"
}
```

### Update Model
```http
PUT /models/{model_id}
Authorization: Bearer {token}
Content-Type: application/json

{
  "description": "Updated description",
  "monitoring_config": {
    "sampling_rate": 0.2
  }
}
```

### Archive Model
```http
POST /models/{model_id}/archive
Authorization: Bearer {token}
```

### Delete Model
```http
DELETE /models/{model_id}
Authorization: Bearer {token}
```

---

## Predictions

### Log Prediction
Log a single prediction for monitoring.

```http
POST /predictions/log
Authorization: Bearer {token}
Content-Type: application/json

{
  "model_id": "model_uuid",
  "prediction_id": "pred_12345",
  "inputs": {
    "amount": 150.50,
    "merchant_category": "grocery",
    "time_of_day": "afternoon"
  },
  "output": {
    "prediction": 0,
    "probability": 0.92,
    "class": "not_fraud"
  },
  "metadata": {
    "user_id": "user_789",
    "request_id": "req_xyz"
  },
  "timestamp": "2025-12-05T15:30:00Z"
}
```

**Response (201 Created):**
```json
{
  "id": "prediction_uuid",
  "model_id": "model_uuid",
  "prediction_id": "pred_12345",
  "status": "logged",
  "timestamp": "2025-12-05T15:30:00Z"
}
```

### Batch Log Predictions
```http
POST /predictions/log/batch
Authorization: Bearer {token}
Content-Type: application/json

{
  "model_id": "model_uuid",
  "predictions": [
    {
      "prediction_id": "pred_001",
      "inputs": {...},
      "output": {...}
    },
    {
      "prediction_id": "pred_002",
      "inputs": {...},
      "output": {...}
    }
  ]
}
```

### Get Predictions
```http
GET /predictions?model_id={model_id}&start_date=2025-12-01&end_date=2025-12-05&page=1&page_size=50
Authorization: Bearer {token}
```

**Query Parameters:**
- `model_id` (required): Model UUID
- `start_date` (optional): Start date (ISO 8601)
- `end_date` (optional): End date (ISO 8601)
- `prediction_id` (optional): Filter by prediction ID
- `page` (optional): Page number
- `page_size` (optional): Items per page

---

## Metrics

### Log Metrics
```http
POST /metrics
Authorization: Bearer {token}
Content-Type: application/json

{
  "model_id": "model_uuid",
  "metric_type": "accuracy",
  "value": 0.94,
  "metadata": {
    "dataset_size": 1000,
    "evaluation_date": "2025-12-05"
  }
}
```

### Batch Log Metrics
```http
POST /metrics/batch
Authorization: Bearer {token}
Content-Type: application/json

{
  "model_id": "model_uuid",
  "metrics": [
    {"metric_type": "accuracy", "value": 0.94},
    {"metric_type": "precision", "value": 0.91},
    {"metric_type": "recall", "value": 0.89},
    {"metric_type": "f1_score", "value": 0.90}
  ]
}
```

### Calculate Metrics
Calculate metrics from predictions.

```http
POST /metrics/calculate
Authorization: Bearer {token}
Content-Type: application/json

{
  "model_id": "model_uuid",
  "y_true": [0, 1, 1, 0, 1],
  "y_pred": [0, 1, 0, 0, 1],
  "metric_types": ["accuracy", "precision", "recall", "f1_score"]
}
```

**Response (200 OK):**
```json
{
  "model_id": "model_uuid",
  "metrics": {
    "accuracy": 0.8,
    "precision": 0.67,
    "recall": 0.67,
    "f1_score": 0.67
  },
  "calculated_at": "2025-12-05T15:30:00Z"
}
```

### Get Aggregated Metrics
```http
GET /metrics/aggregate?model_id={model_id}&metric_type=accuracy&aggregation=avg&time_window=7d
Authorization: Bearer {token}
```

**Query Parameters:**
- `model_id` (required): Model UUID
- `metric_type` (required): Metric type
- `aggregation` (required): Aggregation function (avg, min, max, sum, count)
- `time_window` (required): Time window (1h, 24h, 7d, 30d)
- `start_date` (optional): Start date
- `end_date` (optional): End date

**Response (200 OK):**
```json
{
  "model_id": "model_uuid",
  "metric_type": "accuracy",
  "aggregation": "avg",
  "time_window": "7d",
  "value": 0.93,
  "data_points": 168,
  "period": {
    "start": "2025-11-28T00:00:00Z",
    "end": "2025-12-05T00:00:00Z"
  }
}
```

### Get Metrics Trend
```http
GET /metrics/trend?model_id={model_id}&metric_type=accuracy&interval=1d&days=30
Authorization: Bearer {token}
```

**Response (200 OK):**
```json
{
  "model_id": "model_uuid",
  "metric_type": "accuracy",
  "interval": "1d",
  "data": [
    {"timestamp": "2025-11-05T00:00:00Z", "value": 0.95},
    {"timestamp": "2025-11-06T00:00:00Z", "value": 0.94},
    {"timestamp": "2025-11-07T00:00:00Z", "value": 0.93}
  ]
}
```

---

## Drift Detection

### Detect Data Drift
```http
POST /drift/detect
Authorization: Bearer {token}
Content-Type: application/json

{
  "model_id": "model_uuid",
  "reference_data": {
    "feature1": [1.2, 3.4, 2.1, ...],
    "feature2": ["A", "B", "A", ...]
  },
  "current_data": {
    "feature1": [1.5, 3.8, 2.5, ...],
    "feature2": ["A", "C", "B", ...]
  },
  "config": {
    "threshold": 0.05,
    "method": "ks_test"
  }
}
```

**Response (200 OK):**
```json
{
  "model_id": "model_uuid",
  "drift_detected": true,
  "overall_drift_score": 0.12,
  "features": {
    "feature1": {
      "drift_detected": true,
      "drift_score": 0.08,
      "p_value": 0.001,
      "method": "ks_test",
      "severity": "medium"
    },
    "feature2": {
      "drift_detected": true,
      "drift_score": 0.15,
      "p_value": 0.0001,
      "method": "chi_squared",
      "severity": "high"
    }
  },
  "timestamp": "2025-12-05T15:30:00Z"
}
```

### Get Drift Reports
```http
GET /drift/reports?model_id={model_id}&drift_type=data&start_date=2025-11-01
Authorization: Bearer {token}
```

**Query Parameters:**
- `model_id` (required): Model UUID
- `drift_type` (optional): Filter by type (data, concept)
- `severity` (optional): Filter by severity (low, medium, high)
- `start_date` (optional): Start date
- `end_date` (optional): End date

### Get Drift Trend
```http
GET /drift/trend?model_id={model_id}&feature=feature1&days=30
Authorization: Bearer {token}
```

---

## Explainability (XAI)

### Request Explanation
Generate an explanation for a prediction.

```http
POST /explanations/generate
Authorization: Bearer {token}
Content-Type: application/json

{
  "model_id": "model_uuid",
  "prediction_id": "pred_12345",
  "method": "shap",
  "config": {
    "background_samples": 100,
    "max_features": 10
  }
}
```

**Response (202 Accepted):**
```json
{
  "explanation_id": "exp_uuid",
  "status": "processing",
  "estimated_time": 5
}
```

### Get Explanation
```http
GET /explanations/{explanation_id}
Authorization: Bearer {token}
```

**Response (200 OK):**
```json
{
  "id": "exp_uuid",
  "model_id": "model_uuid",
  "prediction_id": "pred_12345",
  "method": "shap",
  "status": "completed",
  "result": {
    "base_value": 0.2,
    "predicted_value": 0.92,
    "feature_contributions": {
      "amount": 0.15,
      "merchant_category": -0.05,
      "time_of_day": 0.62
    },
    "top_features": [
      {"feature": "time_of_day", "value": 0.62, "impact": "positive"},
      {"feature": "amount", "value": 0.15, "impact": "positive"}
    ]
  },
  "created_at": "2025-12-05T15:30:00Z",
  "completed_at": "2025-12-05T15:30:05Z"
}
```

### Get Global Feature Importance
```http
GET /explanations/global?model_id={model_id}&method=shap
Authorization: Bearer {token}
```

**Response (200 OK):**
```json
{
  "model_id": "model_uuid",
  "method": "shap",
  "feature_importance": {
    "time_of_day": 0.45,
    "amount": 0.30,
    "merchant_category": 0.15,
    "location": 0.10
  },
  "calculated_at": "2025-12-05T15:30:00Z"
}
```

---

## Bias & Fairness

### Run Fairness Audit
```http
POST /bias/audit
Authorization: Bearer {token}
Content-Type: application/json

{
  "model_id": "model_uuid",
  "protected_attributes": ["gender", "age_group"],
  "reference_group": {"gender": "male", "age_group": "30-50"},
  "metrics": ["demographic_parity", "equal_opportunity", "equalized_odds"],
  "threshold": 0.8
}
```

**Response (202 Accepted):**
```json
{
  "audit_id": "audit_uuid",
  "status": "processing"
}
```

### Get Audit Results
```http
GET /bias/audits/{audit_id}
Authorization: Bearer {token}
```

**Response (200 OK):**
```json
{
  "id": "audit_uuid",
  "model_id": "model_uuid",
  "status": "completed",
  "overall_fairness_score": 0.85,
  "bias_detected": true,
  "protected_attributes": ["gender", "age_group"],
  "results": {
    "demographic_parity": {
      "passed": false,
      "score": 0.72,
      "threshold": 0.8,
      "groups": {
        "male_30-50": {"rate": 0.45},
        "female_30-50": {"rate": 0.62},
        "male_50+": {"rate": 0.40}
      }
    },
    "equal_opportunity": {
      "passed": true,
      "score": 0.88
    }
  },
  "recommendations": [
    "Consider rebalancing training data for gender groups",
    "Review feature importance for age-related features"
  ],
  "created_at": "2025-12-05T15:30:00Z"
}
```

### List Audits
```http
GET /bias/audits?model_id={model_id}&status=completed
Authorization: Bearer {token}
```

---

## LLM Monitoring

### Log LLM Completion
```http
POST /llm/completions
Authorization: Bearer {token}
Content-Type: application/json

{
  "model_id": "model_uuid",
  "provider": "openai",
  "model_name": "gpt-4",
  "prompt": "Explain quantum computing",
  "completion": "Quantum computing is...",
  "tokens": {
    "prompt_tokens": 15,
    "completion_tokens": 120,
    "total_tokens": 135
  },
  "latency_ms": 1500,
  "cost": 0.0027,
  "metadata": {
    "temperature": 0.7,
    "max_tokens": 200
  }
}
```

### Get LLM Metrics
```http
GET /llm/metrics?model_id={model_id}&start_date=2025-12-01&end_date=2025-12-05
Authorization: Bearer {token}
```

**Response (200 OK):**
```json
{
  "model_id": "model_uuid",
  "period": {
    "start": "2025-12-01T00:00:00Z",
    "end": "2025-12-05T23:59:59Z"
  },
  "totals": {
    "completions": 15420,
    "total_tokens": 2345678,
    "prompt_tokens": 1234567,
    "completion_tokens": 1111111,
    "total_cost": 47.89,
    "avg_latency_ms": 1245
  },
  "by_model": {
    "gpt-4": {
      "completions": 10000,
      "cost": 35.00
    },
    "gpt-3.5-turbo": {
      "completions": 5420,
      "cost": 12.89
    }
  }
}
```

### Analyze Toxicity
```http
POST /llm/toxicity
Authorization: Bearer {token}
Content-Type: application/json

{
  "text": "Example text to analyze",
  "completion_id": "completion_uuid"
}
```

**Response (200 OK):**
```json
{
  "completion_id": "completion_uuid",
  "toxicity_detected": false,
  "scores": {
    "toxicity": 0.02,
    "severe_toxicity": 0.001,
    "obscene": 0.01,
    "threat": 0.002,
    "insult": 0.015,
    "identity_attack": 0.003
  },
  "flagged": false
}
```

---

## Alerts

### Create Alert Rule
```http
POST /alerts/rules
Authorization: Bearer {token}
Content-Type: application/json

{
  "name": "High Error Rate Alert",
  "description": "Trigger when error rate exceeds 5%",
  "model_id": "model_uuid",
  "conditions": [
    {
      "metric": "error_rate",
      "operator": "greater_than",
      "threshold": 0.05,
      "aggregation": "avg",
      "time_window": "5m"
    }
  ],
  "severity": "high",
  "channels": ["slack", "email"],
  "enabled": true
}
```

**Response (201 Created):**
```json
{
  "id": "rule_uuid",
  "name": "High Error Rate Alert",
  "status": "active",
  "created_at": "2025-12-05T15:30:00Z"
}
```

### List Alert Rules
```http
GET /alerts/rules?model_id={model_id}&enabled=true
Authorization: Bearer {token}
```

### Get Active Alerts
```http
GET /alerts?status=active&severity=high
Authorization: Bearer {token}
```

**Response (200 OK):**
```json
{
  "total": 3,
  "alerts": [
    {
      "id": "alert_uuid",
      "rule_id": "rule_uuid",
      "model_id": "model_uuid",
      "severity": "high",
      "status": "active",
      "message": "Error rate (7.2%) exceeds threshold (5.0%)",
      "triggered_at": "2025-12-05T15:25:00Z",
      "acknowledged": false
    }
  ]
}
```

### Acknowledge Alert
```http
POST /alerts/{alert_id}/acknowledge
Authorization: Bearer {token}
Content-Type: application/json

{
  "notes": "Investigating the issue"
}
```

---

## Reports

### Generate Report
```http
POST /reports/generate
Authorization: Bearer {token}
Content-Type: application/json

{
  "template": "model_performance",
  "model_id": "model_uuid",
  "config": {
    "start_date": "2025-11-01",
    "end_date": "2025-12-05",
    "include_sections": ["metrics", "drift", "explanations"],
    "format": "pdf"
  }
}
```

**Response (202 Accepted):**
```json
{
  "report_id": "report_uuid",
  "status": "generating",
  "estimated_time": 30
}
```

### Get Report Status
```http
GET /reports/{report_id}
Authorization: Bearer {token}
```

**Response (200 OK):**
```json
{
  "id": "report_uuid",
  "status": "completed",
  "template": "model_performance",
  "format": "pdf",
  "download_url": "/reports/report_uuid/download",
  "created_at": "2025-12-05T15:30:00Z",
  "completed_at": "2025-12-05T15:30:30Z"
}
```

### Download Report
```http
GET /reports/{report_id}/download
Authorization: Bearer {token}
```

---

## Users & RBAC

### List Users
```http
GET /users?role=admin&status=active
Authorization: Bearer {token}
```

### Create User
```http
POST /users
Authorization: Bearer {token}
Content-Type: application/json

{
  "username": "newuser@example.com",
  "email": "newuser@example.com",
  "password": "secure_password",
  "full_name": "New User",
  "role": "viewer"
}
```

### Update User Role
```http
PUT /users/{user_id}/role
Authorization: Bearer {token}
Content-Type: application/json

{
  "role": "admin"
}
```

### List Roles
```http
GET /roles
Authorization: Bearer {token}
```

### Create Role
```http
POST /roles
Authorization: Bearer {token}
Content-Type: application/json

{
  "name": "data_scientist",
  "description": "Data scientist with model access",
  "permissions": [
    "models:read",
    "models:create",
    "predictions:read",
    "explanations:read"
  ]
}
```

---

## Dashboard

### Get Dashboard Stats
```http
GET /dashboard/stats?time_range=7d
Authorization: Bearer {token}
```

**Response (200 OK):**
```json
{
  "models": {
    "total": 45,
    "active": 38,
    "inactive": 7
  },
  "predictions": {
    "total": 1542000,
    "today": 45000,
    "avg_per_day": 220285
  },
  "alerts": {
    "active": 3,
    "acknowledged": 12,
    "resolved": 45
  },
  "drift": {
    "models_with_drift": 5,
    "high_severity": 2
  }
}
```

### WebSocket Connection
```javascript
// Connect to real-time dashboard updates
const ws = new WebSocket('wss://api.whiteboxxai.com/dashboard/ws?token={token}');

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log('Update:', data);
};
```

---

## Error Handling

### Error Response Format
All errors follow a consistent format:

```json
{
  "error": {
    "code": "MODEL_NOT_FOUND",
    "message": "Model with ID 'model_uuid' not found",
    "details": {
      "model_id": "model_uuid"
    },
    "timestamp": "2025-12-05T15:30:00Z",
    "request_id": "req_xyz"
  }
}
```

### HTTP Status Codes

| Status Code | Meaning |
|-------------|---------|
| 200 | OK - Request successful |
| 201 | Created - Resource created successfully |
| 202 | Accepted - Request accepted for processing |
| 204 | No Content - Successful deletion |
| 400 | Bad Request - Invalid request data |
| 401 | Unauthorized - Missing or invalid token |
| 403 | Forbidden - Insufficient permissions |
| 404 | Not Found - Resource not found |
| 409 | Conflict - Resource already exists |
| 422 | Unprocessable Entity - Validation error |
| 429 | Too Many Requests - Rate limit exceeded |
| 500 | Internal Server Error - Server error |
| 503 | Service Unavailable - Service temporarily unavailable |

### Common Error Codes

| Code | Description |
|------|-------------|
| `AUTHENTICATION_FAILED` | Invalid credentials |
| `TOKEN_EXPIRED` | JWT token has expired |
| `INSUFFICIENT_PERMISSIONS` | User lacks required permissions |
| `MODEL_NOT_FOUND` | Model does not exist |
| `VALIDATION_ERROR` | Request data validation failed |
| `RATE_LIMIT_EXCEEDED` | Too many requests |
| `RESOURCE_CONFLICT` | Resource already exists |
| `PROCESSING_ERROR` | Error processing request |

---

## Rate Limits

- **Standard:** 1000 requests per hour
- **Prediction Logging:** 10,000 requests per hour
- **Batch Operations:** 100 requests per hour
- **Report Generation:** 10 requests per hour

Rate limit headers:
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 950
X-RateLimit-Reset: 1638720000
```

---

## Pagination

List endpoints support pagination:

**Request:**
```http
GET /models?page=2&page_size=20
```

**Response:**
```json
{
  "total": 145,
  "page": 2,
  "page_size": 20,
  "total_pages": 8,
  "has_next": true,
  "has_prev": true,
  "next_page": 3,
  "prev_page": 1,
  "models": [...]
}
```

---

## Versioning

The API uses URL versioning. Current version: **v1**

Future versions will be available at:
- `/api/v2/...`
- `/api/v3/...`

---

## SDK Integration

For easier integration, use the Python SDK:

```python
from whiteboxxai import WhiteBoxXAI

client = WhiteBoxXAI(api_key="your_api_key")

# Register model
model = client.models.register(
    name="my_model",
    model_type="classification"
)

# Log prediction
client.predictions.log(
    model_id=model.id,
    inputs={"feature": 1.5},
    output={"prediction": 0}
)
```

See [SDK Documentation](index.md) for details.

---

## Support

- **Documentation:** https://docs.whiteboxxai.com
- **API Status:** https://status.whiteboxxai.com
- **Support:** support@whiteboxxai.com

---

*Last Updated: December 5, 2025*
