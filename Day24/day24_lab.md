# Day 24 – 18-Jun-2026 (Thursday)
# Module 7 & 8 – Monitoring & Production Readiness

# Lab: Centralized Logging, DR Simulation & Cost Optimization Dashboards

## AWS Resources Used

- AWS X-Ray
- Amazon OpenSearch
- Amazon Managed Grafana
- Amazon Route53
- AWS Global Accelerator
- AWS Cost Explorer

---

# Lab Objectives

Participants will:

- Enable distributed tracing.
- Configure centralized logging.
- Create observability dashboards.
- Simulate disaster recovery scenarios.
- Configure Route53 failover.
- Review cost optimization opportunities.
- Build FinOps dashboards.

---

# Prerequisites

Verify AWS Access:

```bash
aws sts get-caller-identity
```

Verify EKS Access:

```bash
kubectl get nodes
```

---

# Lab 1: Enable AWS X-Ray

Navigate:

AWS X-Ray Console

Review:

- Service Map
- Trace Data

Verify X-Ray access.

---

# Lab 2: Review Traces

Generate application traffic.

Navigate:

X-Ray → Traces

Review:

- Request Latency
- Errors
- Dependencies

---

# Lab 3: Deploy OpenTelemetry Collector

Verify Collector Deployment:

```bash
kubectl get pods -A
```

Review telemetry collection architecture.

---

# Lab 4: Configure Centralized Logging

Review Log Sources:

- EKS Logs
- Application Logs
- Audit Logs

Verify log aggregation.

---

# Lab 5: Configure Amazon OpenSearch

Navigate:

OpenSearch Console

Create Domain:

training-logs-domain

Verify Domain Status:

Active

---

# Lab 6: Review Log Analytics

Search Logs:

- Errors
- Warnings
- Security Events

Document findings.

---

# Lab 7: Configure Grafana

Create Dashboard:

training-observability-dashboard

Add Widgets:

- CPU Usage
- Memory Usage
- Request Count
- Error Rate

---

# Lab 8: Create Operations Dashboard

Dashboard Name:

operations-dashboard

Include:

- Application Health
- Node Metrics
- Log Metrics

---

# Lab 9: Multi-Region DR Architecture Review

Regions:

Primary:
eu-north-1

Secondary:
eu-west-1

Document:

- Failover Process
- Recovery Process

---

# Lab 10: Configure Route53 Failover

Create:

Primary Record

Secondary Record

Enable Health Checks.

Verify configuration.

---

# Lab 11: Review Global Accelerator

Navigate:

Global Accelerator Console

Review:

- Endpoints
- Traffic Routing

Discuss use cases.

---

# Lab 12: Simulate Failover

Scenario:

Primary endpoint unavailable.

Observe:

- Route53 failover
- Traffic redirection

Record observations.

---

# Lab 13: Enable Cost Explorer

Navigate:

Cost Explorer

Review:

- Monthly Costs
- Service Costs
- Cost Trends

---

# Lab 14: Build Cost Dashboard

Dashboard Name:

finops-dashboard

Include:

- Monthly Spend
- Service Breakdown
- Cost Forecast

---

# Lab 15: Identify Optimization Opportunities

Review:

- Idle Resources
- Underutilized Instances
- Unused Storage

Document recommendations.

---

# Lab 16: Cost Optimization Exercise

Implement:

- Resource tagging
- Auto Scaling review
- Storage optimization review

---

# Lab 17: Cleanup

Delete:

- Test Dashboards
- Test OpenSearch Domain (Optional)

Retain:

- Cost Reports
- Documentation

---

# Challenge Exercise

Implement:

1. X-Ray Review
2. OpenTelemetry Review
3. OpenSearch Logging
4. Grafana Dashboard
5. Route53 Failover Design
6. Cost Optimization Analysis

Document:

- Architecture
- Screenshots
- Recommendations

---

# Lab Deliverables

Submit:

- X-Ray Screenshot
- OpenSearch Screenshot
- Grafana Dashboard Screenshot
- Route53 Configuration Screenshot
- DR Simulation Results
- Cost Optimization Report

---

# Expected Learning Outcomes

✓ Understand Distributed Tracing

✓ Configure Centralized Logging

✓ Build Observability Dashboards

✓ Understand Disaster Recovery

✓ Configure Route53 Failover

✓ Review Global Accelerator

✓ Apply FinOps Principles

✓ Analyze Cloud Costs
