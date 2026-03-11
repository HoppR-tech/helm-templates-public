# Datadog Monitors Helm Chart

A Helm chart to declaratively manage **Datadog monitors** from Kubernetes using the **Datadog Operator**.

This chart allows teams to define monitoring and alerting rules as code and deploy them through standard **GitOps workflows**.

Instead of manually creating monitors in the Datadog UI, developers can define them in a `values.yaml` file and deploy them alongside their applications.

---

# Why?

Monitoring and alerting should be part of the **application delivery process**, not an afterthought.

This chart enables teams to:

- Define Datadog monitors as **code**
- Version monitoring rules in **Git**
- Deploy monitors automatically through **CI/CD**
- Integrate alerting directly into **Pull Requests**
- Manage observability through **GitOps workflows**

---

# Requirements

This chart **requires the Datadog Operator to be installed in the cluster**.

The operator provides the `DatadogMonitor` Custom Resource Definition (CRD) used by this chart.

Datadog Operator repository:

https://github.com/DataDog/helm-charts/tree/main/charts/datadog-operator

Requirements:

- Kubernetes cluster
- Helm 3+
- Datadog Operator installed
- `DatadogMonitor` CRD available in the cluster
- Datadog Agent configured and connected to your Datadog account

---

# Installing the Datadog Operator

Install the operator before using this chart.

```bash
helm repo add datadog https://helm.datadoghq.com
helm repo update

helm install datadog-operator datadog/datadog-operator
```
Verify the CRD is installed:
```bash
kubectl get crd datadogmonitors.datadoghq.com
```

# Installation
Install the chart:
```bash
helm install datadog-monitors ./charts/datadog-monitors \
  -f values.yaml
```

Upgrade:
```bash
helm upgrade datadog-monitors ./charts/datadog-monitors \
  -f values.yaml
```
# Configuration
Monitors are defined in the `values.yaml` file.

Example:
```yaml
datadog_monitors:
  high-cpu-usage-by-node:
    type: "query alert"
    query: "avg(last_15m):avg:kubernetes.cpu.usage.total{*} by {node} > 0.85"
    message: "High CPU usage detected on {{node.name}}"

    tags:
      - "severity:critical"
      - "team:platform"

    priority: 1

    options:
      thresholds:
        critical: "0.95"
        warning: "0.85"

      evaluationDelay: 310
      includeTags: true
      locked: false
      newGroupDelay: 300
      notifyNoData: false
      noDataTimeframe: 30
      renotifyInterval: 10
```
Each item in `datadog_monitors` generates a corresponding **DatadogMonitor resource**.

# Example GitOps Workflow

Typical workflow using Helm + GitOps:

1. Developer defines monitoring rules in values.yaml
2. Developer opens a Pull Request
3. CI validates the Helm chart
4. Changes are merged
5. A GitOps tool deploys the monitors automatically

Compatible with:

- ArgoCD
- Flux
- Any Helm-based deployment pipeline

 # Chart Structure

 ```bash
datadog-monitors/
├── Chart.yaml
├── values-example.yaml
├── values.yaml
└── templates/
    └── datadog-monitors.yaml
```
Where:

- `values.yaml` defines the monitors
- `values-examples` is just an implementation example
- `templates/` renders `DatadogMonitor` resources

# Use Cases

This chart is useful for teams that want to:

- Manage **observability as code**
- Standardize monitoring across services
- Integrate alerting into the **deployment lifecycle**
- Avoid manual configuration in the Datadog UI
- Manage monitors through **GitOps**

# Best Practices

Recommended practices:
- Store monitors in the **same repository as the service**
- Review monitoring changes through **Pull Requests**
- Use **environment-specific values files**
- Or declare a global DatadogMonitor which is able to manage all of your environments

Example:
```bash
values-dev.yaml
values-staging.yaml
values-prod.yaml
```
Tag monitors with metadata such as:
```bash
team
service
severity
environment
```

