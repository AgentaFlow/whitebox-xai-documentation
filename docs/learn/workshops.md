# Hands-On Workshops

This document outlines interactive workshop materials designed to provide practical experience with WhiteBoxXAI platform.

## Overview

These workshops combine theory with practice, allowing participants to learn by doing. Each workshop includes setup instructions, step-by-step exercises, challenge problems, and solutions.

---

## 🎯 Workshop Catalog

### Workshop 1: First Model Integration (90 minutes)
**Audience:** Data scientists, ML engineers
**Level:** Beginner
**Prerequisites:** Python knowledge, basic ML understanding

#### Learning Objectives
- Set up WhiteBoxXAI SDK in a Python environment
- Register and log predictions from a scikit-learn model
- View explanations in the dashboard
- Configure basic monitoring and alerts

#### Materials Needed
- Python 3.8+ environment
- Jupyter Notebook or Python IDE
- WhiteBoxXAI account credentials
- Sample dataset (provided: `credit_scoring.csv`)

#### Workshop Structure

**Part 1: Environment Setup (15 min)**
1. Install Python and create virtual environment
2. Install WhiteBoxXAI SDK: `pip install whitebox-xai-sdk`
3. Verify installation
4. Configure authentication with API key
5. Test connection to platform

**Part 2: Train a Simple Model (20 min)**
1. Load credit scoring dataset
2. Explore data (features, target distribution)
3. Split into train/test sets
4. Train logistic regression model
5. Evaluate baseline accuracy
6. Save model for later use

**Part 3: SDK Integration (25 min)**
1. Import WhiteBoxXAI client
2. Initialize with API credentials
3. Register the model with metadata
4. Wrap model with framework adapter
5. Log training metadata
6. Log first prediction with explanation
7. Verify data appears in dashboard

**Part 4: Batch Prediction Logging (15 min)**
1. Log test set predictions
2. Use batch logging for efficiency
3. Configure sampling for large datasets
4. Monitor logging progress
5. Handle errors gracefully

**Part 5: Dashboard Exploration (15 min)**
1. Navigate to model in dashboard
2. View prediction history
3. Explore SHAP explanations
4. Check feature importance
5. Set up performance monitoring
6. Configure drift detection alert

#### Hands-On Exercises

**Exercise 1: Log Your First Prediction**
```python
from whiteboxxai import WhiteBoxXAI
from sklearn.linear_model import LogisticRegression
import pandas as pd

# TODO: Initialize client with your API key
client = WhiteBoxXAI(api_key="YOUR_API_KEY")

# TODO: Load data and train model
data = pd.read_csv("credit_scoring.csv")
# ... train your model ...

# TODO: Log a prediction with explanation
# YOUR CODE HERE
```

**Exercise 2: Batch Logging**
```python
# TODO: Log predictions for entire test set
# Use batch logging for better performance
# YOUR CODE HERE
```

**Exercise 3: Configure Monitoring**
```python
# TODO: Set up drift detection
# Configure alert thresholds
# YOUR CODE HERE
```

#### Challenge Problems

**Challenge 1: Custom Sampling**
Implement custom sampling to log only high-confidence predictions (>0.8 or <0.2 probability).

**Challenge 2: Error Handling**
Add robust error handling with retries and fallback mechanisms.

**Challenge 3: Async Logging**
Refactor to use asynchronous logging for better performance.

#### Solutions
Full solutions provided in `workshops/01-first-model/solutions/` directory.

---

### Workshop 2: Advanced Explainability (120 minutes)
**Audience:** Data scientists, ML researchers
**Level:** Intermediate
**Prerequisites:** Workshop 1 or equivalent experience

#### Learning Objectives
- Generate and interpret SHAP explanations
- Use LIME for text and image models
- Understand global vs local explanations
- Analyze feature interactions
- Debug model decisions using explanations

#### Workshop Structure

**Part 1: SHAP Deep Dive (30 min)**
1. Review SHAP theory (Shapley values)
2. Generate waterfall plots
3. Create force plots
4. Build summary plots
5. Analyze beeswarm plots
6. Interpret dependence plots

**Part 2: LIME for Different Data Types (30 min)**
1. LIME for tabular data
2. LIME for text classification
3. LIME for image classification
4. Compare LIME vs SHAP results
5. When to use each method

**Part 3: Global Explanations (25 min)**
1. Feature importance across dataset
2. Partial dependence plots
3. ICE (Individual Conditional Expectation) plots
4. Global feature interactions
5. Model behavior analysis

**Part 4: Debugging with Explanations (35 min)**
1. Identify suspicious predictions
2. Analyze misclassifications
3. Detect data leakage
4. Find spurious correlations
5. Validate domain knowledge
6. Document findings

#### Hands-On Exercises

**Exercise 1: Create Comprehensive Explanation**
```python
import shap
import lime

# TODO: Generate SHAP explanation
# TODO: Generate LIME explanation
# TODO: Compare and interpret both
# YOUR CODE HERE
```

**Exercise 2: Analyze Feature Interactions**
```python
# TODO: Create interaction plots
# TODO: Identify top interactions
# TODO: Interpret business meaning
# YOUR CODE HERE
```

**Exercise 3: Debug a Prediction**
```python
# TODO: Find a misclassified example
# TODO: Generate explanation
# TODO: Identify root cause
# TODO: Propose fix
# YOUR CODE HERE
```

#### Challenge Problems

**Challenge 1: Custom Explainer**
Implement a custom explainer for a specific model type.

**Challenge 2: Explanation Consistency**
Analyze consistency of explanations across similar predictions.

**Challenge 3: Counterfactual Analysis**
Generate counterfactual explanations ("what if" scenarios).

---

### Workshop 3: Bias Detection & Fairness (120 minutes)
**Audience:** Data scientists, compliance officers, ethicists
**Level:** Advanced
**Prerequisites:** Workshop 1 or 2

#### Learning Objectives
- Understand fairness metrics
- Detect bias in model predictions
- Analyze protected attributes
- Implement bias mitigation strategies
- Document fairness analysis

#### Workshop Structure

**Part 1: Understanding Bias (25 min)**
1. Types of bias in ML
2. Legal and ethical considerations
3. Protected attributes
4. Fairness metrics overview
5. Trade-offs in fairness

**Part 2: Running Bias Analysis (30 min)**
1. Configure bias detection
2. Specify protected attributes
3. Run fairness analysis
4. Interpret bias metrics
5. Visualize disparities
6. Statistical significance testing

**Part 3: Mitigation Strategies (35 min)**
1. Pre-processing: reweighting data
2. In-processing: fairness constraints
3. Post-processing: threshold adjustment
4. Implement mitigation in code
5. Evaluate effectiveness
6. Document approach

**Part 4: Compliance Documentation (30 min)**
1. Generate fairness report
2. Document methodology
3. Create audit trail
4. Prepare stakeholder summary
5. Establish monitoring plan

#### Hands-On Exercises

**Exercise 1: Detect Bias**
```python
from whiteboxxai.bias import BiasDetector

# TODO: Initialize bias detector
# TODO: Configure protected attributes
# TODO: Run analysis
# TODO: Interpret results
# YOUR CODE HERE
```

**Exercise 2: Implement Mitigation**
```python
# TODO: Choose mitigation strategy
# TODO: Apply to your model
# TODO: Measure improvement
# YOUR CODE HERE
```

**Exercise 3: Create Fairness Report**
```python
# TODO: Generate comprehensive fairness report
# TODO: Include visualizations
# TODO: Add interpretations
# YOUR CODE HERE
```

#### Challenge Problems

**Challenge 1: Multiple Protected Attributes**
Analyze intersectional fairness across multiple protected attributes.

**Challenge 2: Fairness-Accuracy Trade-off**
Find optimal balance between fairness and accuracy.

**Challenge 3: Temporal Fairness**
Monitor fairness over time and detect fairness drift.

---

### Workshop 4: Production Monitoring (120 minutes)
**Audience:** ML engineers, DevOps, data scientists
**Level:** Intermediate
**Prerequisites:** Workshop 1, basic production ML experience

#### Learning Objectives
- Set up real-time monitoring
- Configure drift detection
- Create custom alerts
- Implement automated responses
- Build monitoring dashboards

#### Workshop Structure

**Part 1: Monitoring Setup (25 min)**
1. Initialize monitoring configuration
2. Select metrics to track
3. Set baseline performance
4. Configure data collection
5. Verify monitoring is active

**Part 2: Drift Detection (30 min)**
1. Types of drift (data, concept, prediction)
2. Statistical tests for drift
3. Configure drift thresholds
4. Analyze drift alerts
5. Investigate drift causes
6. Document findings

**Part 3: Custom Alerts (30 min)**
1. Define alerting rules
2. Set up notification channels
3. Configure alert severity levels
4. Test alert delivery
5. Create runbooks for responses
6. Set up alert escalation

**Part 4: Automated Responses (35 min)**
1. Implement health checks
2. Graceful degradation patterns
3. Automated retraining triggers
4. Rollback procedures
5. Circuit breaker pattern
6. Load shedding strategies

#### Hands-On Exercises

**Exercise 1: Configure Drift Detection**
```python
from whiteboxxai.monitoring import DriftDetector

# TODO: Initialize drift detector
# TODO: Set baseline distribution
# TODO: Configure alert thresholds
# YOUR CODE HERE
```

**Exercise 2: Create Custom Alert**
```python
# TODO: Define custom metric
# TODO: Set alerting rule
# TODO: Configure notification
# YOUR CODE HERE
```

**Exercise 3: Implement Health Check**
```python
# TODO: Create health check endpoint
# TODO: Monitor model health
# TODO: Return status and metrics
# YOUR CODE HERE
```

---

### Workshop 5: PyTorch Integration (90 minutes)
**Audience:** Deep learning engineers, ML researchers
**Level:** Intermediate
**Prerequisites:** PyTorch knowledge, Workshop 1

#### Learning Objectives
- Integrate WhiteBoxXAI with PyTorch models
- Log predictions from neural networks
- Generate explanations for deep learning
- Monitor training progress
- Handle tensor inputs efficiently

#### Workshop Structure

**Part 1: PyTorch Setup (20 min)**
1. PyTorch adapter overview
2. Model registration
3. Handling tensor inputs
4. Custom preprocessing hooks
5. Batch processing with DataLoader

**Part 2: Training Integration (25 min)**
1. Log training metrics
2. Track validation performance
3. Monitor learning curves
4. Save checkpoints with metadata
5. Version model iterations

**Part 3: Inference Monitoring (25 min)**
1. Log inference predictions
2. Generate explanations for predictions
3. Handle batch inference
4. Optimize for latency
5. Monitor inference performance

**Part 4: Advanced Techniques (20 min)**
1. Gradient-based explanations
2. Attention visualization
3. Layer activation analysis
4. Feature map exploration

#### Hands-On Exercises

**Exercise 1: Register PyTorch Model**
```python
import torch
from whiteboxxai import WhiteBoxXAI

# TODO: Define PyTorch model
# TODO: Register with WhiteBoxXAI
# YOUR CODE HERE
```

**Exercise 2: Training Loop Integration**
```python
# TODO: Add logging to training loop
# TODO: Track metrics per epoch
# YOUR CODE HERE
```

**Exercise 3: Inference Monitoring**
```python
# TODO: Log inference predictions
# TODO: Generate explanations
# YOUR CODE HERE
```

---


## 🛠️ Workshop Materials

### Required Files

For each workshop, provide:
- `README.md` - Workshop overview and instructions
- `setup.sh` / `setup.ps1` - Environment setup scripts
- `requirements.txt` - Python dependencies
- `data/` - Sample datasets
- `notebooks/` - Jupyter notebooks with exercises
- `solutions/` - Complete solutions
- `slides.pdf` - Presentation slides
- `cheatsheet.pdf` - Quick reference guide

### Sample Datasets

**Credit Scoring Dataset** (Workshop 1)
- 10,000 samples
- 20 features (numeric and categorical)
- Binary target (default/no default)
- Includes protected attributes (age, gender)

**Fraud Detection Dataset** (Workshop 3)
- 50,000 transactions
- Imbalanced (1% fraud rate)
- 30 features
- Temporal component

**Image Classification Dataset** (Workshop 5)
- MNIST or Fashion-MNIST
- 10,000 images
- 10 classes
- Preprocessed for PyTorch

---

## 📋 Facilitator Guide

### Preparation Checklist

**One Week Before:**
- [ ] Review workshop materials
- [ ] Test all code examples
- [ ] Verify dataset availability
- [ ] Set up demo environment
- [ ] Prepare participant accounts
- [ ] Send pre-workshop email with setup instructions

**One Day Before:**
- [ ] Test screen sharing and audio
- [ ] Verify all links work
- [ ] Review common troubleshooting issues
- [ ] Prepare backup plans
- [ ] Check timezone for participants

**Day Of:**
- [ ] Join 15 minutes early
- [ ] Test screen share
- [ ] Have troubleshooting guide ready
- [ ] Monitor chat for questions
- [ ] Have phone number for technical support

### Facilitation Tips

**Ice Breaker (5 min)**
- Have participants introduce themselves
- Ask about their experience level
- Learn about their use cases
- Set expectations for the workshop

**Setup Time (10-15 min)**
- Walk through setup together
- Have TAs help with issues
- Use breakout rooms if needed
- Don't rush this step

**Live Coding**
- Type slowly and deliberately
- Explain what you're doing
- Make intentional mistakes to show debugging
- Encourage participants to code along
- Check in frequently ("Everyone with me?")

**Exercise Time**
- Give clear time limits
- Circulate to help (or use breakout rooms)
- Share solutions after time is up
- Discuss different approaches

**Breaks**
- Take 10-minute break every 60 minutes
- Announce break timing upfront
- Use breaks to catch up with strugglers

### Troubleshooting Common Issues

**Installation Problems**
- Have a prebuilt environment ready as backup
- Provide cloud notebook environment (Google Colab)
- Pair struggling participants with helpers

**API Connection Issues**
- Verify API key format
- Check firewall/proxy settings
- Use different network if needed
- Have shared demo account as fallback

**Code Not Working**
- Check Python version
- Verify all dependencies installed
- Compare with working solution
- Use screen sharing to debug

---

## 🎓 Self-Paced Options

Each workshop can be adapted for self-paced learning:

### Self-Paced Format
- **Video recordings** of each section (10-15 min each)
- **Interactive notebooks** with explanations
- **Auto-graded exercises** where possible
- **Discussion forum** for questions
- **Office hours** for live help (weekly)

### Assessment
- **Knowledge checks** after each section (3-5 questions)
- **Hands-on projects** to validate learning
- **Peer review** of challenge solutions
- **Certificates** upon completion

---

## 📊 Workshop Metrics

Track these metrics to improve workshops:

### Participant Metrics
- Registration vs attendance rate
- Completion rate
- Exercise completion time
- Quiz scores
- Satisfaction ratings (1-5)
- Net Promoter Score

### Content Metrics
- Which sections take longest
- Most common errors
- Most asked questions
- Challenge completion rate
- Follow-up question themes

### Improvements
- Update based on feedback quarterly
- Add FAQs based on questions
- Improve sections with low scores
- Update datasets and examples
- Refresh screenshots and demos

---

## 📞 Support

For workshop questions:
- **Email:** workshops@whiteboxxai.example.com
- **Slack:** #workshops
- **Office Hours:** Tuesdays 2-3 PM EST

---

*Last Updated: December 2024*
*Version: 1.0*
