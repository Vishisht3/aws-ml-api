# AWS ML API - Technical Specification

## 1. Problem Statement

Build a real-time REST API for sentiment analysis that:  
- Accepts a short text input in JSON format  
- Returns the predicted sentiment label (`positive`, `neutral`, or `negative`)  

Use case: Support customer feedback analysis for product teams.

---

## 2. Functional Requirements

- **Endpoint:** `/predict` (POST request)  
- **Input JSON schema:**  
{
"type": "object",
"properties": {
"text": { "type": "string", "maxLength": 1024 }
},
"required": ["text"]
}
- **Output JSON schema:**  
{
"type": "object",
"properties": {
"sentiment": { "type": "string", "enum": ["positive", "neutral", "negative"] }
},
"required": ["sentiment"]
}

---

## 3. Performance Targets (SLOs)

| Metric           | Target         | Measurement Method        |
|------------------|----------------|--------------------------|
| P95 Latency      | ≤ 700 ms       | CloudWatch latency logs   |
| Availability     | ≥ 99.9%        | CloudWatch errors & uptime|
| Error Rate       | < 1% 4xx/5xx   | CloudWatch error metrics  |

Define error budget for monthly SLA compliance.

---

## 4. Security

- All API requests must include a valid JWT token issued by the configured identity provider.  
- Rate limiting: Max 100 requests per user per minute, burst up to 200.  
- Least-privilege IAM roles: Lambda only allowed to call `sagemaker:InvokeEndpoint` on the specific SageMaker endpoint ARN.  
- All traffic encrypted with TLS.

---

## 5. Architecture

- Client → API Gateway (JWT auth + rate limiting) → AWS Lambda (input validation, SageMaker invoke) → SageMaker endpoint → Lambda → API Gateway → Client

*Insert architecture diagram image `/docs/diagram.png` here*

---

## 6. Deployment & Versioning

- Deploy model endpoint using SageMaker Autopilot on pre-selected dataset.  
- Use Lambda aliases to route 90% traffic to stable model version, 10% to canary version.  
- Automatic rollback if canary’s P95 latency exceeds stable version by >30% or error rate >2% sustained 5 minutes.  
- Maintain deployment scripts in `/deploy` directory for IaC.

---

## 7. Observability & Monitoring

- CloudWatch dashboards to track: request count, P50/P95 latency, 4xx and 5xx errors, SageMaker invocation errors.  
- Set CloudWatch alarms to notify Slack/email on SLA breaches or anomalous error spikes.  
- Log correlation IDs passed through Lambda for tracing.

---

## 8. Input Validation & Error Handling

- Validate all input against JSON schema.  
- Reject oversize payloads (>1KB) with HTTP 413 error.  
- Return meaningful error responses with explanation on 4xx errors.

---

## 9. Governance & Explainability

- Schedule SageMaker Clarify jobs weekly for bias detection and feature importance.  
- Store reports in S3 with access logs.  
- Include Explainability summary endpoint (optional).

---

## 10. Incident Response

- Runbook outlining steps to diagnose, mitigate, and recover from service issues (e.g., high error rates, endpoint unavailability).  
- Include rollback instructions for deployment issues.

---

## 11. Future Enhancements

- Batch inference to reduce cost at high traffic.  
- Integrate caching layer for repeated queries.  
- Fine-grained IAM using resource tagging.  
- End-to-end CI/CD with automated integration tests.

---

*End of spec.md template.*

