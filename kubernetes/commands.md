# Kubernetes Monitoring Setup Cheat Sheet (GKE)

## 1. Connect to Cluster

``` bash
gcloud container clusters get-credentials resumatches-cluster-1 \
--region asia-east1 \
--project <PROJECT_ID>
```

## 2. Verify Cluster

``` bash
kubectl get ns
kubectl get nodes
kubectl top nodes
```

## 3. Monitoring Namespace

``` bash
kubectl create namespace monitoring
kubectl get pods -n monitoring
```

## 4. Helm Repositories

``` bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

## 5. Install Prometheus + Grafana

``` bash
helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring

helm upgrade prometheus prometheus-community/kube-prometheus-stack \
-n monitoring

helm upgrade prometheus prometheus-community/kube-prometheus-stack \
-n monitoring -f persistent.yml --reuse-values
```

## 6. Install Loki

``` bash
helm install loki grafana/loki -n monitoring

helm install loki grafana/loki \
-n monitoring \
--set loki.storage.type=gcs \
--set loki.storage.bucketNames.chunks=<bucket> \
--set loki.storage.bucketNames.ruler=<bucket> \
--set loki.storage.bucketNames.admin=<bucket> \
--set loki.storage.gcs.bucket_name=<bucket> \
--set loki.useTestSchema=true

helm upgrade loki grafana/loki -n monitoring --reuse-values
```

## 7. Install Promtail

``` bash
helm install promtail grafana/promtail -n monitoring

helm upgrade --install promtail grafana/promtail \
-n monitoring \
--set config.clients[0].url=http://loki-gateway/loki/api/v1/push \
--set config.clients[0].tenant_id=1
```

## 8. Helm Operations

``` bash
helm list -n monitoring

helm uninstall prometheus -n monitoring
helm uninstall loki -n monitoring
helm uninstall promtail -n monitoring
```

## 9. Monitoring Commands

``` bash
kubectl get pods -n monitoring
kubectl get pods -n monitoring -o wide
kubectl get pods -w -n monitoring

kubectl top nodes
kubectl top pods -n monitoring

kubectl get pvc -n monitoring
kubectl get pv
```

## 10. Troubleshooting

``` bash
kubectl describe pod <pod> -n monitoring
kubectl logs -f <pod> -n monitoring

kubectl rollout restart deployment prometheus-grafana -n monitoring

kubectl get secret prometheus-grafana \
-n monitoring \
-o jsonpath="{.data.admin-password}" | base64 --decode

kubectl edit secret prometheus-grafana -n monitoring

kubectl delete pod <pod> -n monitoring
kubectl delete pvc prometheus-grafana -n monitoring
```

## 11. Loki Health Checks

``` bash
curl http://loki-gateway.monitoring.svc.cluster.local/ready

curl http://127.0.0.1:3101/metrics

curl http://127.0.0.1:3100/loki/api/v1/query_range \
--data-urlencode 'query={job="test"}' \
-H X-Scope-OrgID:foo
```

## 12. Useful kubectl Commands

``` bash
kubectl get deployment -n monitoring
kubectl get daemonset -n monitoring
kubectl get configmap -n monitoring
kubectl get secret -n monitoring
```

## 13. Cleanup Pending Pods

``` bash
kubectl get pods -n monitoring | grep Pending

kubectl get pods -n monitoring \
| grep Pending \
| awk '{print $1}' \
| xargs kubectl delete pod -n monitoring
```

## 14. Recommended Setup Flow

1.  Connect to GKE
2.  Create monitoring namespace
3.  Add Helm repositories
4.  Install kube-prometheus-stack
5.  Install Loki
6.  Configure storage (PVC/GCS)
7.  Install Promtail
8.  Connect Promtail → Loki
9.  Verify logs in Grafana
10. Expose Grafana/Prometheus
11. Monitor metrics and logs
12. Troubleshoot using `kubectl logs`, `describe`, and `rollout restart`

## 15. Interview Quick Commands

``` bash
helm list -n monitoring

helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring
helm install loki grafana/loki -n monitoring
helm upgrade --install promtail grafana/promtail -n monitoring

kubectl get pods -n monitoring
kubectl logs -f <pod> -n monitoring
kubectl describe pod <pod> -n monitoring
kubectl top nodes
kubectl top pods -n monitoring
kubectl rollout restart deployment prometheus-grafana -n monitoring
kubectl get pvc -n monitoring
```
