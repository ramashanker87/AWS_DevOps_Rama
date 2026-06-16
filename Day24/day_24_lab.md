# Day 24 – 18-Jun-2026 (Thursday)
# Module 7 & 8 – Monitoring & Production Readiness

# Lab: Centralized Logging, Distributed Tracing, Disaster Recovery & FinOps

## AWS Resources Used

- AWS X-Ray
- AWS Distro for OpenTelemetry (ADOT)
- Amazon OpenSearch Service
- Amazon Managed Grafana
- Amazon Route 53
- AWS Global Accelerator
- AWS Cost Explorer

---

# Lab Objectives

Participants will:

- Configure distributed tracing using AWS X-Ray.
- Understand OpenTelemetry integration.
- Explore centralized logging architecture.
- Simulate multi-region disaster recovery.
- Configure traffic failover using Route 53.
- Review Global Accelerator architecture.
- Analyze AWS costs using Cost Explorer.
- Build cost optimization dashboards.
- Understand production readiness best practices.

---

# Prerequisites

## Verify AWS Access

```bash
aws sts get-caller-identity --profile devops
```

Purpose:
Verifies AWS credentials and confirms access to the target AWS account.

## Verify Current Region

```bash
aws configure get region --profile devops
```

Purpose:
Confirms the AWS region configured for the devops profile.

---

# Lab 1: Explore AWS X-Ray

## Objective

Understand distributed tracing and application observability.

## Verify X-Ray Access

```bash
aws xray get-service-graph \
  --start-time $(date -u -d "1 hour ago" +%FT%TZ) \
  --end-time $(date -u +%FT%TZ) \
  --profile devops
```

Purpose:
Retrieves service dependency graph data captured by X-Ray.

Discussion:
- Service Map
- Trace Analysis
- Latency Investigation
- Error Tracking

---

# Lab 2: Review X-Ray Traces

## List Trace Summaries

```bash
aws xray get-trace-summaries \
  --start-time $(date -u -d "1 hour ago" +%FT%TZ) \
  --end-time $(date -u +%FT%TZ) \
  --profile devops
```

Purpose:
Displays traces collected during the selected time period.

Review:

- Request path
- Response times
- Errors
- Downstream service calls

---

# Lab 3: Explore OpenTelemetry Architecture

## Objective

Understand how telemetry data is collected and exported.

## Verify EKS Resources

```bash
kubectl get pods -A
```

Purpose:
Lists all running pods.

## Verify OpenTelemetry Collector

```bash
kubectl get pods -A | grep -i telemetry
```

Purpose:
Checks whether OpenTelemetry Collector is deployed.

Discussion:

- Metrics Collection
- Logs Collection
- Traces Collection
- Exporters
- Observability Pipeline

---

# Lab 4: Centralized Logging with OpenSearch

## List OpenSearch Domains

```bash
aws opensearch list-domain-names \
  --profile devops
```

Purpose:
Displays available OpenSearch domains.

## Describe Domain

```bash
aws opensearch describe-domain \
  --domain-name training-logs \
  --profile devops
```

Purpose:
Shows configuration and endpoint information.

Discussion:

Centralized logging enables:

- Faster troubleshooting
- Security analysis
- Audit visibility
- Operational monitoring

---

# Lab 5: Review Application Logs

## View Kubernetes Events

```bash
kubectl get events -A --sort-by=.metadata.creationTimestamp
```

Purpose:
Displays recent Kubernetes cluster events.

## View Application Logs

```bash
kubectl logs deployment/sample-app
```

Purpose:
Displays application logs.

Review:

- Errors
- Warnings
- Performance issues
- Security events

---

# Lab 6: Explore Amazon Managed Grafana

## Objective

Review monitoring dashboards.

## List Grafana Workspaces

```bash
aws grafana list-workspaces \
  --profile devops
```

Purpose:
Displays available Grafana workspaces.

Review Dashboards:

- Infrastructure Metrics
- Kubernetes Metrics
- Application Metrics
- Cost Metrics

---

# Lab 7: Multi-Region Disaster Recovery Planning

## Objective

Understand DR architecture.

Review:

- Primary Region
- Secondary Region
- Backup Strategy
- Recovery Procedures

## List Available Regions

```bash
aws ec2 describe-regions \
  --profile devops \
  --query "Regions[].RegionName"
```

Purpose:
Displays available AWS regions.

---

# Lab 8: Route 53 Failover Configuration

## List Hosted Zones

```bash
aws route53 list-hosted-zones \
  --profile devops
```

Purpose:
Displays Route 53 hosted zones.

Discussion:

Failover routing provides:

- Automatic recovery
- High availability
- Business continuity

---

# Lab 9: Review Global Accelerator

## List Accelerators

```bash
aws globalaccelerator list-accelerators \
  --profile devops
```

Purpose:
Displays configured accelerators.

Review:

- Endpoint Groups
- Traffic Distribution
- Health Checks

Benefits:

- Lower latency
- Faster failover
- Global traffic management

---

# Lab 10: Disaster Recovery Simulation

## Scenario

Simulate primary region outage.

Tasks:

1. Review failover strategy.
2. Validate secondary region readiness.
3. Verify Route 53 failover records.
4. Review application accessibility.

Verification Commands:

```bash
nslookup example.com
```

```bash
dig example.com
```

Purpose:
Validates DNS resolution and failover behavior.

---

# Lab 11: Explore AWS Cost Explorer

## Enable Cost Visibility

Open:

AWS Console → Cost Explorer

Review:

- Monthly Cost
- Service Breakdown
- Daily Trends

## Retrieve Cost Data

```bash
aws ce get-cost-and-usage \
  --time-period Start=2026-06-01,End=2026-06-30 \
  --granularity MONTHLY \
  --metrics UnblendedCost \
  --profile devops
```

Purpose:
Retrieves monthly AWS cost data.

---

# Lab 12: Analyze Top AWS Services

Review:

- EC2
- EKS
- CloudWatch
- OpenSearch
- Data Transfer

Identify:

- Cost drivers
- Underutilized resources
- Optimization opportunities

---

# Lab 13: Build Cost Optimization Dashboard

Dashboard Name:

cost-optimization-dashboard

Include:

- Monthly Cost
- Daily Cost Trend
- Top Services
- Forecasted Spend

Purpose:
Provides visibility into AWS spending patterns.

---

# Lab 14: FinOps Optimization Exercise

Identify Opportunities:

- Idle Resources
- Overprovisioned Instances
- Unused Storage
- Excessive Logging Costs

Examples:

```bash
aws ec2 describe-instances \
  --profile devops
```

```bash
aws ec2 describe-volumes \
  --profile devops
```

Purpose:
Helps identify potentially unused infrastructure.

---

# Lab 15: Production Readiness Review

Validate:

- Monitoring
- Logging
- Tracing
- Disaster Recovery
- Cost Controls
- Alerting
- Dashboard Visibility

Checklist:

✓ Monitoring Enabled

✓ Logging Centralized

✓ Tracing Configured

✓ DR Strategy Reviewed

✓ Cost Visibility Established

✓ Operational Dashboards Available

---

# Challenge Exercise

Implement:

1. X-Ray Trace Review
2. OpenTelemetry Verification
3. OpenSearch Log Analysis
4. DR Simulation
5. Cost Optimization Dashboard

Document:

- Screenshots
- Findings
- Recommendations
- Cost Savings Opportunities

---

# Lab Deliverables

Submit:

- X-Ray Screenshot
- Grafana Dashboard Screenshot
- OpenSearch Screenshot
- Route 53 Screenshot
- Global Accelerator Screenshot
- Cost Explorer Screenshot
- DR Validation Notes
- FinOps Recommendations

---

# Expected Learning Outcomes

✓ Understand Distributed Tracing

✓ Explore OpenTelemetry Architecture

✓ Implement Centralized Logging

✓ Understand Disaster Recovery Concepts

✓ Configure High Availability Services

✓ Analyze AWS Costs

✓ Apply FinOps Best Practices

✓ Improve Production Readiness
