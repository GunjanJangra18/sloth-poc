# Sloth SLO Integration with VictoriaMetrics

## 1. Overview

This POC demonstrates how Sloth can be integrated with an existing Kubernetes monitoring environment using VictoriaMetrics.

Sloth is used to define SLOs and automatically generate monitoring and alerting rules. The generated rules are processed by the existing VictoriaMetrics monitoring components and alerts are displayed in Grafana.

---

## 2. POC Objective

- Integrate Sloth with the existing Kubernetes cluster.
- Define SLOs using YAML configuration.
- Generate monitoring and alerting rules automatically.
- Integrate generated rules with VictoriaMetrics.
- Verify SLO alerts in Grafana.

---

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

---

## 4. Implementation

### Step 1: Install Sloth CRD

The Sloth CRD was installed in the Kubernetes cluster.

It allows Kubernetes to recognize Sloth resources such as `PrometheusServiceLevel`, which is used to define SLO configurations.

### Step 2: Create SLO YAML

A YAML configuration was created for `rems-api-server`.

```
apiVersion: sloth.slok.dev/v1
kind: PrometheusServiceLevel

metadata:
  name: rems-api-server-slo
  namespace: sloth-poc

spec:
  service: rems-api-server

  labels:
    app: rems-api-server
    env: prod

  slos:
    - name: api-availability
      objective: 99
      description: "REMS API Server API Availability SLO"

      sli:
        events:
          errorQuery: |
            sum(
              count_over_time(
                probe_success{
                  instance="https://rems-api.opstree.net/api/v1/healthz",
                  rems_service="rems-api-server",
                  rems_environment="prod"
                }[{{.window}}]
              )
            )
            -
            sum(
              sum_over_time(
                probe_success{
                  instance="https://rems-api.opstree.net/api/v1/healthz",
                  rems_service="rems-api-server",
                  rems_environment="prod"
                }[{{.window}}]
              )
            )

          totalQuery: |
            sum(
              count_over_time(
                probe_success{
                  instance="https://rems-api.opstree.net/api/v1/healthz",
                  rems_service="rems-api-server",
                  rems_environment="prod"
                }[{{.window}}]
              )
            )

      alerting:
        name: rems-api-server-alerts
```

It defines the service, availability target, time period, SLI queries, and alerting configuration.

This configuration is applied to Kubernetes as a `PrometheusServiceLevel` resource.

### Step 3: Deploy Sloth Controller

The Sloth controller was deployed in the `sloth-poc` namespace.

The controller watches the SLO configuration and generates the required Prometheus recording and alerting rules.

### Step 4: Generate PrometheusRule

After applying the SLO configuration, Sloth generates a PrometheusRule.

```
Name:         rems-api-server-slo
Namespace:    sloth-poc
Labels:       app.kubernetes.io/component=SLO
              app.kubernetes.io/managed-by=sloth
Annotations:  <none>
API Version:  monitoring.coreos.com/v1
Kind:         PrometheusRule
Metadata:
  Creation Timestamp:  2026-10-27T11:52:39Z
  Generation:          1
  Owner References:
    API Version:     sloth.slok.dev/v1
    Kind:            PrometheusServiceLevel
    Name:            rems-api-server-slo
    UID:             25db78bd-983e-4cae-9a4b-a3a877fcabc2
  Resource Version:  142013820
  UID:               ff9f5a5f-08d7-4c8d-97c5-b2f4fb832999
Spec:
  Groups:
    Name:  sloth-slo-sli-recordings-rems-api-server-api-availability
    Rules:
      Expr:  (sum(
  count_over_time(
    probe_success{
      instance="https://rems-api.opstree.net/api/v1/healthz",
      rems_service="rems-api-server",
      rems_environment="prod"
    }[5m]
  )
)
-
sum(
  sum_over_time(
    probe_success{
      instance="https://rems-api.opstree.net/api/v1/healthz",
      rems_service="rems-api-server",
      rems_environment="prod"
    }[5m]
  )
)
)
/
(sum(
  count_over_time(
    probe_success{
      instance="https://rems-api.opstree.net/api/v1/healthz",
      rems_service="rems-api-server",
      rems_environment="prod"
    }[5m]
  )
)
)

      Labels:
        App:            rems-api-server
        Env:            prod
        sloth_id:       rems-api-server-api-availability
        sloth_service:  rems-api-server
        sloth_slo:      api-availability
        sloth_window:   5m
      Record:           slo:sli_error:ratio_rate5m
      Expr:             (sum(
  count_over_time(
    probe_success{
      instance="https://rems-api.opstree.net/api/v1/healthz",
      rems_service="rems-api-server",
      rems_environment="prod"
    }[30m]
  )
)
-
sum(
  sum_over_time(
    probe_success{
      instance="https://rems-api.opstree.net/api/v1/healthz",
      rems_service="rems-api-server",
      rems_environment="prod"
    }[30m]
  )
)
)
/
(sum(
  count_over_time(
    probe_success{
      instance="https://rems-api.opstree.net/api/v1/healthz",
      rems_service="rems-api-server",
      rems_environment="prod"
    }[30m]
  )
)
)

      Labels:
        App:            rems-api-server
        Env:            prod
        sloth_id:       rems-api-server-api-availability
        sloth_service:  rems-api-server
        sloth_slo:      api-availability
        sloth_window:   30m
      Record:           slo:sli_error:ratio_rate30m
      Expr:             (sum(
  count_over_time(
    probe_success{
      instance="https://rems-api.opstree.net/api/v1/healthz",
      rems_service="rems-api-server",
      rems_environment="prod"
    }[1h]
  )
)
-
sum(
  sum_over_time(
    probe_success{
      instance="https://rems-api.opstree.net/api/v1/healthz",
      rems_service="rems-api-server",
      rems_environment="prod"
    }[1h]
  )
)
)
/
(sum(
  count_over_time(
    probe_success{
      instance="https://rems-api.opstree.net/api/v1/healthz",
      rems_service="rems-api-server",
      rems_environment="prod"
    }[1h]
  )
)
)

      Labels:
        App:            rems-api-server
        Env:            prod
        sloth_id:       rems-api-server-api-availability
        sloth_service:  rems-api-server
        sloth_slo:      api-availability
        sloth_window:   1h
      Record:           slo:sli_error:ratio_rate1h
      Expr:             (sum(
  count_over_time(
    probe_success{
      instance="https://rems-api.opstree.net/api/v1/healthz",
      rems_service="rems-api-server",
      rems_environment="prod"
    }[2h]
  )
)
-
sum(
  sum_over_time(
    probe_success{
      instance="https://rems-api.opstree.net/api/v1/healthz",
      rems_service="rems-api-server",
      rems_environment="prod"
    }[2h]
  )
)
)
/
(sum(
  count_over_time(
    probe_success{
      instance="https://rems-api.opstree.net/api/v1/healthz",
      rems_service="rems-api-server",
      rems_environment="prod"
    }[2h]
  )
)
)

      Labels:
        App:            rems-api-server
        Env:            prod
        sloth_id:       rems-api-server-api-availability
        sloth_service:  rems-api-server
        sloth_slo:      api-availability
        sloth_window:   2h
      Record:           slo:sli_error:ratio_rate2h
      Expr:             (sum(
  count_over_time(
    probe_success{
      instance="https://rems-api.opstree.net/api/v1/healthz",
      rems_service="rems-api-server",
      rems_environment="prod"
    }[6h]
  )
)
-
sum(
  sum_over_time(
    probe_success{
      instance="https://rems-api.opstree.net/api/v1/healthz",
      rems_service="rems-api-server",
      rems_environment="prod"
    }[6h]
  )
)
)
/
(sum(
  count_over_time(
    probe_success{
      instance="https://rems-api.opstree.net/api/v1/healthz",
      rems_service="rems-api-server",
      rems_environment="prod"
    }[6h]
  )
)
)

      Labels:
        App:            rems-api-server
        Env:            prod
        sloth_id:       rems-api-server-api-availability
        sloth_service:  rems-api-server
        sloth_slo:      api-availability
        sloth_window:   6h
      Record:           slo:sli_error:ratio_rate6h
      Expr:             (sum(
  count_over_time(
    probe_success{
      instance="https://rems-api.opstree.net/api/v1/healthz",
      rems_service="rems-api-server",
      rems_environment="prod"
    }[1d]
  )
)
-
sum(
  sum_over_time(
    probe_success{
      instance="https://rems-api.opstree.net/api/v1/healthz",
      rems_service="rems-api-server",
      rems_environment="prod"
    }[1d]
  )
)
)
/
(sum(
  count_over_time(
    probe_success{
      instance="https://rems-api.opstree.net/api/v1/healthz",
      rems_service="rems-api-server",
      rems_environment="prod"
    }[1d]
  )
)
)

      Labels:
        App:            rems-api-server
        Env:            prod
        sloth_id:       rems-api-server-api-availability
        sloth_service:  rems-api-server
        sloth_slo:      api-availability
        sloth_window:   1d
      Record:           slo:sli_error:ratio_rate1d
      Expr:             (sum(
  count_over_time(
    probe_success{
      instance="https://rems-api.opstree.net/api/v1/healthz",
      rems_service="rems-api-server",
      rems_environment="prod"
    }[3d]
  )
)
-
sum(
  sum_over_time(
    probe_success{
      instance="https://rems-api.opstree.net/api/v1/healthz",
      rems_service="rems-api-server",
      rems_environment="prod"
    }[3d]
  )
)
)
/
(sum(
  count_over_time(
    probe_success{
      instance="https://rems-api.opstree.net/api/v1/healthz",
      rems_service="rems-api-server",
      rems_environment="prod"
    }[3d]
  )
)
)

      Labels:
        App:            rems-api-server
        Env:            prod
        sloth_id:       rems-api-server-api-availability
        sloth_service:  rems-api-server
        sloth_slo:      api-availability
        sloth_window:   3d
      Record:           slo:sli_error:ratio_rate3d
      Expr:             sum_over_time(slo:sli_error:ratio_rate5m{sloth_id="rems-api-server-api-availability", sloth_service="rems-api-server", sloth_slo="api-availability"}[30d])
/ ignoring (sloth_window)
count_over_time(slo:sli_error:ratio_rate5m{sloth_id="rems-api-server-api-availability", sloth_service="rems-api-server", sloth_slo="api-availability"}[30d])

      Labels:
        App:            rems-api-server
        Env:            prod
        sloth_id:       rems-api-server-api-availability
        sloth_service:  rems-api-server
        sloth_slo:      api-availability
        sloth_window:   30d
      Record:           slo:sli_error:ratio_rate30d
    Name:               sloth-slo-meta-recordings-rems-api-server-api-availability
    Rules:
      Expr:  vector(0.99)
      Labels:
        App:            rems-api-server
        Env:            prod
        sloth_id:       rems-api-server-api-availability
        sloth_service:  rems-api-server
        sloth_slo:      api-availability
      Record:           slo:objective:ratio
      Expr:             vector(1-0.99)
      Labels:
        App:            rems-api-server
        Env:            prod
        sloth_id:       rems-api-server-api-availability
        sloth_service:  rems-api-server
        sloth_slo:      api-availability
      Record:           slo:error_budget:ratio
      Expr:             vector(30)
      Labels:
        App:            rems-api-server
        Env:            prod
        sloth_id:       rems-api-server-api-availability
        sloth_service:  rems-api-server
        sloth_slo:      api-availability
      Record:           slo:time_period:days
      Expr:             slo:sli_error:ratio_rate5m{sloth_id="rems-api-server-api-availability", sloth_service="rems-api-server", sloth_slo="api-availability"}
/ on(sloth_id, sloth_slo, sloth_service) group_left
slo:error_budget:ratio{sloth_id="rems-api-server-api-availability", sloth_service="rems-api-server", sloth_slo="api-availability"}

      Labels:
        App:            rems-api-server
        Env:            prod
        sloth_id:       rems-api-server-api-availability
        sloth_service:  rems-api-server
        sloth_slo:      api-availability
      Record:           slo:current_burn_rate:ratio
      Expr:             slo:sli_error:ratio_rate30d{sloth_id="rems-api-server-api-availability", sloth_service="rems-api-server", sloth_slo="api-availability"}
/ on(sloth_id, sloth_slo, sloth_service) group_left
slo:error_budget:ratio{sloth_id="rems-api-server-api-availability", sloth_service="rems-api-server", sloth_slo="api-availability"}

      Labels:
        App:            rems-api-server
        Env:            prod
        sloth_id:       rems-api-server-api-availability
        sloth_service:  rems-api-server
        sloth_slo:      api-availability
      Record:           slo:period_burn_rate:ratio
      Expr:             1 - slo:period_burn_rate:ratio{sloth_id="rems-api-server-api-availability", sloth_service="rems-api-server", sloth_slo="api-availability"}
      Labels:
        App:            rems-api-server
        Env:            prod
        sloth_id:       rems-api-server-api-availability
        sloth_service:  rems-api-server
        sloth_slo:      api-availability
      Record:           slo:period_error_budget_remaining:ratio
      Expr:             vector(1)
      Labels:
        App:              rems-api-server
        Env:              prod
        sloth_id:         rems-api-server-api-availability
        sloth_mode:       ctrl-gen-k8s
        sloth_objective:  99
        sloth_service:    rems-api-server
        sloth_slo:        api-availability
        sloth_spec:       sloth.slok.dev/v1
        sloth_version:    v0.15.0
      Record:             sloth_slo_info
    Name:                 sloth-slo-alerts-rems-api-server-api-availability
    Rules:
      Alert:  rems-api-server-alerts
      Annotations:
        Summary:  {{$labels.sloth_service}} {{$labels.sloth_slo}} SLO error budget burn rate is over expected.
        Title:    (page) {{$labels.sloth_service}} {{$labels.sloth_slo}} SLO error budget burn rate is too fast.
      Expr:       (
    max(slo:sli_error:ratio_rate5m{sloth_id="rems-api-server-api-availability", sloth_service="rems-api-server", sloth_slo="api-availability"} > (14.4 * 0.01)) without (sloth_window)
    and
    max(slo:sli_error:ratio_rate1h{sloth_id="rems-api-server-api-availability", sloth_service="rems-api-server", sloth_slo="api-availability"} > (14.4 * 0.01)) without (sloth_window)
)
or
(
    max(slo:sli_error:ratio_rate30m{sloth_id="rems-api-server-api-availability", sloth_service="rems-api-server", sloth_slo="api-availability"} > (6 * 0.01)) without (sloth_window)
    and
    max(slo:sli_error:ratio_rate6h{sloth_id="rems-api-server-api-availability", sloth_service="rems-api-server", sloth_slo="api-availability"} > (6 * 0.01)) without (sloth_window)
)

      Labels:
        sloth_severity:  page
      Alert:             rems-api-server-alerts
      Annotations:
        Summary:  {{$labels.sloth_service}} {{$labels.sloth_slo}} SLO error budget burn rate is over expected.
        Title:    (ticket) {{$labels.sloth_service}} {{$labels.sloth_slo}} SLO error budget burn rate is too fast.
      Expr:       (
    max(slo:sli_error:ratio_rate2h{sloth_id="rems-api-server-api-availability", sloth_service="rems-api-server", sloth_slo="api-availability"} > (3 * 0.01)) without (sloth_window)
    and
    max(slo:sli_error:ratio_rate1d{sloth_id="rems-api-server-api-availability", sloth_service="rems-api-server", sloth_slo="api-availability"} > (3 * 0.01)) without (sloth_window)
)
or
(
    max(slo:sli_error:ratio_rate6h{sloth_id="rems-api-server-api-availability", sloth_service="rems-api-server", sloth_slo="api-availability"} > (1 * 0.01)) without (sloth_window)
    and
    max(slo:sli_error:ratio_rate3d{sloth_id="rems-api-server-api-availability", sloth_service="rems-api-server", sloth_slo="api-availability"} > (1 * 0.01)) without (sloth_window)
)

      Labels:
        sloth_severity:  ticket
Events:                  <none>

```
It contains rules for calculating SLI, error budget, burn rate, and SLO alerts.

### Step 5: Integrate with VictoriaMetrics

The existing VictoriaMetrics Operator processes the generated PrometheusRule and creates the corresponding VMRule.

VMRule contains the recording and alerting rules in a format supported by the existing VictoriaMetrics setup.

### Step 6: Evaluate Alerts using VMAlert

The existing VMAlert component evaluates the generated rules.

When an alert condition is met, the alert becomes active and can be viewed in Grafana.

## Alert Triggering Basis

Alerts are triggered based on the SLI error ratio and error budget burn rate.
- Page Alert: Triggered when API errors increase rapidly and consume the error budget at a high rate.
- Ticket Alert: Triggered when API errors continue over a longer period and consume the error budget slowly.

---

## 5. Architecture Flow

<img width="1774" height="887" alt="image" src="https://github.com/user-attachments/assets/c1fd0056-791f-4ee7-8e5a-5844b33e02c4" />

---

## 6. Outcome

The POC successfully demonstrated the integration of Sloth with the existing VictoriaMetrics monitoring stack.

SLOs were defined using YAML, and Sloth generated the required monitoring and alerting rules.

The generated rules were integrated with the existing VictoriaMetrics setup, and SLO alerts were verified in Grafana.

VMAlert evaluates the rules and alert conditions, while Grafana displays the SLO-related alerts.

---
