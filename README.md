#Credit Card Fraud Detection (MVP)

This project is an MVP fraud detection service that scores credit card transactions and triggers alerts when the risk is above a configurable threshold.

Overview

Online payment fraud and scams can cause significant financial losses. To mitigate this risk, I built a minimal end-to-end pipeline that:

serves a fraud scoring model via a FastAPI inference API on AWS EC2
monitors API health and model behavior via Amazon CloudWatch
sends email notifications when abnormal patterns or thresholds are triggered
Architecture

Client -> FastAPI (EC2) -> Model Inference -> CloudWatch (Logs/Metrics/Dashboard) -> Alarms -> Email Notification

Monitoring (MVP)

API / SRE

Requests, 5xx error rate, p95 latency
Model Output

Risk score distribution
High-risk rate (score >= threshold)
Decision rate (approve / reject)
Data Quality

Missing feature rate
Unknown category rate (unseen categorical values)
Out-of-range feature count
Alerting

API 5xx rate > 1% for 5 minutes
Average latency > 500ms for 10 minutes
High-risk rate spike (threshold or anomaly detection)
Missing/unknown rates exceed thresholds
Note: Threshold values in this MVP are for demonstration purposes. In a real production setup, they should be calibrated using business requirements and validated with domain experts.

Model Information

I evaluated multiple baseline models for fraud detection (e.g., Logistic Regression+5f, Random Forest, IsolationForest) and selected the final approach based on the precision–recall trade-off on a validation set.

Final model: Random Forest classifier
Imbalance handling: oversampling on the training set (e.g., RandomOverSampler / SMOTE)
Decision threshold: 0.20 (tuned on the validation set to balance precision and recall for the target operating point)

Validation results (example):

Precision: 0.87
Recall: 0.80
F1-score: 0.84
PR-AUC: 0.92
Demo

Demo A: Latency Alert

Enable slow mode: set env `DEMO_SLOW_MS=20
Send requests for 1 minutes
Observe Average LatencyMs increase and alarm triggers
Threshold: LatencyMs > 15 for 1 datapoints within 1 minute
Demo B: Model Output Spike

Enable demo override: DEMO_SCORE_OVERRIDE=0.99
Observe HighRiskRate and ScoreBin_8 spike on dashboard
Demo C: Data Quality Alert

Send requests with missing/unknown fields
Observe MissingRate / UnknownCategoryRate increase and alarm triggers
Privacy/Security

Designed with privacy-by-design principles

The API never stores raw cardholder data (e.g., PAN/card number, CVV) or bank credentials.
Application logs are sanitized: only request_id, latency, model_version, decision, and risk_score are logged.
Any sensitive identifiers are masked or hashed before logging (e.g., last-4 only / irreversible hash).
Transport security: HTTPS is recommended (TLS termination via ALB/Nginx) and security groups restrict inbound traffic.
AWS IAM follows least-privilege: EC2 instance role is limited to CloudWatch logging/metrics (no long-lived access keys in code).
Alert notifications contain aggregated metrics only (no PII in emails).
