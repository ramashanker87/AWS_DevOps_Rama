# Day 23 – 17-Jun-2026 (Wednesday)
# Module 6 & 7

# Lab: Security Pipeline Integration & Monitoring Dashboards

## AWS Resources Used

- Amazon GuardDuty
- Amazon Security Hub
- Amazon CloudWatch

---

# Lab Objectives

Participants will:

- Enable GuardDuty.
- Enable Security Hub.
- Review security findings.
- Configure monitoring.
- Create CloudWatch dashboards.
- Create CloudWatch alarms.
- Validate security visibility.

---

# Prerequisites

Verify AWS Access:

```bash
aws sts get-caller-identity
```

---

# Lab 1: Enable GuardDuty

AWS Console:

GuardDuty → Get Started

Enable:

- GuardDuty
- EKS Protection
- Runtime Monitoring

Verify Status:

Enabled

---

# Lab 2: Review GuardDuty Findings

Navigate:

GuardDuty → Findings

Review:

- Severity
- Resource Impacted
- Recommendation

Document sample findings.

---

# Lab 3: Enable Security Hub

Security Hub → Enable

Enable Standards:

- AWS Foundational Security Best Practices
- CIS AWS Foundations Benchmark

Verify Dashboard.

---

# Lab 4: Review Security Hub Findings

Navigate:

Security Hub → Findings

Review:

- Critical Findings
- Compliance Results

---

# Lab 5: Explore Integrated Services

Verify Integrations:

- GuardDuty
- IAM Access Analyzer
- Inspector

Review imported findings.

---

# Lab 6: Enable CloudWatch Container Insights

For EKS:

CloudWatch → Container Insights

Verify:

- Cluster Metrics
- Node Metrics
- Pod Metrics

---

# Lab 7: Review CloudWatch Metrics

Navigate:

CloudWatch → Metrics

Review:

- CPU Utilization
- Memory Utilization
- Network Traffic

---

# Lab 8: Create CloudWatch Dashboard

Dashboard Name:

training-security-dashboard

Add Widgets:

- CPU Usage
- Memory Usage
- Network Traffic
- Error Metrics

Save Dashboard.

---

# Lab 9: Create Log Group

CloudWatch → Logs

Create Log Group:

training-app-logs

Verify creation.

---

# Lab 10: Review Application Logs

Navigate:

CloudWatch Logs

Review:

- Application Logs
- Kubernetes Logs
- Security Events

---

# Lab 11: Create Alarm

Metric:

CPUUtilization

Threshold:

80%

Action:

Notification (Optional)

Verify Alarm Creation.

---

# Lab 12: Create Security Dashboard

Dashboard Name:

security-operations-dashboard

Add:

- GuardDuty Findings
- Security Hub Findings
- Compliance Summary

---

# Lab 13: Generate Test Events

Perform:

- Application deployment
- Pod scaling
- Security configuration review

Observe:

Metrics and Logs updates.

---

# Lab 14: Validate Monitoring Architecture

Verify:

- Metrics Collection
- Logs Collection
- Dashboard Visibility
- Security Findings

---

# Lab 15: Cleanup

Remove:

- Test Dashboards
- Test Log Groups
- Test Alarms

Retain:

- GuardDuty
- Security Hub

---

# Challenge Exercise

Implement:

1. GuardDuty Enablement
2. Security Hub Enablement
3. Dashboard Creation
4. Log Analysis
5. Alarm Configuration

Document:

- Findings
- Screenshots
- Recommendations

---

# Lab Deliverables

Submit:

- GuardDuty Screenshot
- Security Hub Screenshot
- CloudWatch Dashboard Screenshot
- Alarm Screenshot
- Logs Screenshot
- Findings Summary

---

# Expected Learning Outcomes

✓ Enable Security Monitoring

✓ Review Security Findings

✓ Configure CloudWatch Dashboards

✓ Create Monitoring Alarms

✓ Analyze Logs

✓ Improve Operational Visibility

✓ Implement Security Automation Practices
