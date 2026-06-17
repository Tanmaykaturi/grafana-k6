Grafana k6 Operator Deployment GuideThis guide provides the streamlined steps to deploy the k6 Operator and configure automated load testing with Prometheus integration.1. Quick InstallationPrerequisitesKubernetes cluster with kubectl access.Helm 3.x installed.Prometheus Operator (kube-prometheus-stack) installed.monitoring namespace exists.Deployment StepsAdd Helm Repo:Bashhelm repo add grafana https://grafana.github.io/helm-charts
helm repo update
Install k6 Operator:Bashhelm install k6-operator grafana/k6-operator \
  --namespace monitoring \
  --values values.yaml
Enable Prometheus Remote Write:Bashkubectl patch prometheus prometheus-kube-prometheus-prometheus \
  -n monitoring \
  --type=merge \
  -p '{"spec":{"enableRemoteWriteReceiver":true}}'
Deploy Test Resources:Bashkubectl apply -f test-scripts/
kubectl apply -f examples/health-check-testrun.yaml
2. Mandatory Configuration ChangesBefore deploying, ensure you have updated the following fields in your YAML files:values.yaml (Operator Config)Configure the ServiceMonitor to ensure Prometheus discovers the operator.metrics.serviceMonitor.namespace: Set to the namespace where Prometheus resides.metrics.serviceMonitor.labels: Must match your Prometheus serviceMonitorSelector.Find yours: kubectl get prometheus -n monitoring -o yaml | grep serviceMonitorSelectorexamples/health-check-testrun.yaml (Test Config)Update environment variables to point to your specific infrastructure.Environment VariableDescriptionHow to findK6_PROMETHEUS_RW_SERVER_URLPrometheus Remote Write endpointkubectl get svc -n monitoringTARGET_URLThe service you want to testkubectl get svc -n <target-ns>CLUSTER_NAMETag for metric filteringCustom (e.g., prod-us-east)TEST_IDUnique identifier for the testCustom (e.g., api-health-check)ENVIRONMENTDeployment environmentCustom (e.g., production)Service URL Format: http://<service-name>.<namespace>.svc.cluster.local:<port>3. VerificationCheck Operator Status:Bashkubectl get pods -n monitoring -l app.kubernetes.io/name=k6-operator
Verify Test Execution:Bashkubectl get testrun -n monitoring
kubectl logs -n monitoring <test-pod-name>
Query Metrics in Prometheus:Bash# Port-forward Prometheus
kubectl port-forward -n monitoring svc/prometheus-kube-prometheus-prometheus 9090:9090

# Run query in browser or CLI
curl 'http://localhost:9090/api/v1/query?query=k6_http_reqs_total'
4. TroubleshootingMetrics Not Appearing: Ensure enableRemoteWriteReceiver is set to true on your Prometheus object.Test Pod Fails: Use kubectl logs -n monitoring <test-pod-name> to check for unreachable TARGET_URL or misconfigured Prometheus URLs.Custom Labels Missing: If labels aren't showing up, ensure you have re-applied the TestRun after updating the environment variables.
