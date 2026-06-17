# Grafana k6 Operator Deployment Guide

This guide provides the streamlined steps to deploy the k6 Operator and configure automated load testing with Prometheus integration.

---

## 1. Quick Installation

### Prerequisites
* Kubernetes cluster with `kubectl` access.
* Helm 3.x installed.
* Prometheus Operator (`kube-prometheus-stack`) installed.
* `monitoring` namespace exists.

### Deployment Steps

1. **Add Helm Repo:**
   ```bash
   helm repo add grafana [https://grafana.github.io/helm-charts](https://grafana.github.io/helm-charts)
   helm repo update

```

2. **Install k6 Operator:**
```bash
helm install k6-operator grafana/k6-operator \
  --namespace monitoring \
  --values values.yaml

```


3. **Enable Prometheus Remote Write:**
```bash
kubectl patch prometheus prometheus-kube-prometheus-prometheus \
  -n monitoring \
  --type=merge \
  -p '{"spec":{"enableRemoteWriteReceiver":true}}'

```


4. **Deploy Test Resources:**
```bash
kubectl apply -f test-scripts/
kubectl apply -f examples/health-check-testrun.yaml

```



---

## 2. Mandatory Configuration Changes

Before deploying, ensure you have updated the following fields in your configuration files:

### `values.yaml` (Operator Configuration)

Configure the `ServiceMonitor` to ensure Prometheus discovers the operator:

* **`metrics.serviceMonitor.namespace`**: Set to the namespace where Prometheus resides.
* **`metrics.serviceMonitor.labels`**: Must match your Prometheus `serviceMonitorSelector`.
* *Find yours:* `kubectl get prometheus -n monitoring -o yaml | grep serviceMonitorSelector`



### `examples/health-check-testrun.yaml` (Test Configuration)

Update the environment variables to point to your specific infrastructure:

| Environment Variable | Description | How to find |
| --- | --- | --- |
| `K6_PROMETHEUS_RW_SERVER_URL` | Prometheus Remote Write endpoint | `kubectl get svc -n monitoring` |
| `TARGET_URL` | The service you want to test | `kubectl get svc -n <target-ns>` |
| `CLUSTER_NAME` | Tag for metric filtering | Custom (e.g., `prod-us-east`) |
| `TEST_ID` | Unique identifier for the test | Custom (e.g., `api-health-check`) |
| `ENVIRONMENT` | Deployment environment | Custom (e.g., `production`) |

> **Service URL Format:** `http://<service-name>.<namespace>.svc.cluster.local:<port>`

---

## 3. Verification

* **Check Operator Status:**
```bash
kubectl get pods -n monitoring -l app.kubernetes.io/name=k6-operator

```


* **Verify Test Execution:**
```bash
kubectl get testrun -n monitoring
kubectl logs -n monitoring <test-pod-name>

```


* **Query Metrics in Prometheus:**
```bash
# Port-forward Prometheus
kubectl port-forward -n monitoring svc/prometheus-kube-prometheus-prometheus 9090:9090

```



### Run Query

```bash
curl 'http://localhost:9090/api/v1/query?query=k6_http_reqs_total'

```

---

## 4. Troubleshooting

* **Metrics Not Appearing:** Ensure `enableRemoteWriteReceiver` is set to `true` on your Prometheus object.
* **Test Pod Fails:** Use `kubectl logs -n monitoring <test-pod-name>` to check for unreachable `TARGET_URL` or misconfigured Prometheus URLs.
* **Custom Labels Missing:** If labels aren't showing up, ensure you have re-applied the `TestRun` after updating the environment variables.

```

```
