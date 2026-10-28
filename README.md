# Sloth SLO Integration with VictoriaMetrics

## 1. Overview

This POC demonstrates how Sloth can be integrated with an existing Kubernetes monitoring environment using VictoriaMetrics.

Sloth is used to define SLOs and automatically generate monitoring and alerting rules. The generated rules are processed by the existing VictoriaMetrics monitoring components and alerts are displayed in Grafana.

## 2. POC Objective

- Integrate Sloth with the existing Kubernetes cluster.
- Define SLOs using YAML configuration.
- Generate monitoring and alerting rules automatically.
- Integrate generated rules with VictoriaMetrics.
- Verify SLO alerts in Grafana.

## 3. SLO Configuration

The POC was implemented for the following service:

| Configuration | Value |
|---|---|
| Service | rems-api-server |
| SLI | API Availability |
| SLO | 99% |
| Time Period | 30 days |
| Error Budget | 1% |
| Environment | prod |

### Explanation

- **SLI:** The metric used to measure service performance. In this POC, API availability is measured using the existing `probe_success` metric.
- **SLO:** The target availability level for the service. Here, the target is 99% over 30 days.
- **Error Budget:** The allowed amount of failure within the SLO period. For a 99% SLO, the error budget is 1%.

## 4. Implementation

### Step 1: Install Sloth CRD

The Sloth CRD was installed in the Kubernetes cluster.

It allows Kubernetes to recognize Sloth resources such as `PrometheusServiceLevel`, which is used to define SLO configurations.

### Step 2: Create SLO YAML

A YAML configuration was created for `rems-api-server`.

It defines the service, availability target, time period, SLI queries, and alerting configuration.

This configuration is applied to Kubernetes as a `PrometheusServiceLevel` resource.

### Step 3: Deploy Sloth Controller

The Sloth controller was deployed in the `sloth-poc` namespace.

The controller watches the SLO configuration and generates the required Prometheus recording and alerting rules.

### Step 4: Generate PrometheusRule

After applying the SLO configuration, Sloth generates a PrometheusRule.

It contains rules for calculating SLI, error budget, burn rate, and SLO alerts.

### Step 5: Integrate with VictoriaMetrics

The existing VictoriaMetrics Operator processes the generated PrometheusRule and creates the corresponding VMRule.

VMRule contains the recording and alerting rules in a format supported by the existing VictoriaMetrics setup.

### Step 6: Evaluate Alerts using VMAlert

The existing VMAlert component evaluates the generated rules.

When an alert condition is met, the alert becomes active and can be viewed in Grafana.

## 5. Architecture Flow

Sloth CRD  
→ SLO YAML  
→ Sloth Controller  
→ PrometheusRule  
→ VictoriaMetrics Operator  
→ VMRule  
→ VMAlert  
→ Grafana

### Flow Explanation

- **Sloth CRD:** Enables SLO resources in Kubernetes.
- **SLO YAML:** Defines the service-level objective.
- **Sloth Controller:** Reads the configuration and generates rules.
- **VictoriaMetrics Operator:** Integrates the generated rules into the existing monitoring setup.
- **VMAlert:** Evaluates the rules and alert conditions.
- **Grafana:** Displays the SLO-related alerts.

## 6. Outcome

The POC successfully demonstrated the integration of Sloth with the existing VictoriaMetrics monitoring stack.

SLOs were defined using YAML, and Sloth generated the required monitoring and alerting rules.

The generated rules were integrated with the existing VictoriaMetrics setup, and SLO alerts were verified in Grafana.

VMAlert evaluates the rules and alert conditions, while Grafana displays the SLO-related alerts.
