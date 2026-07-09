# Argo CD bootstrap repository

This repository provides a small GitOps bootstrap setup for Argo CD and a monitoring stack that integrates with it. It uses Argo CD Applications to manage additional applications from the same repository, including Prometheus, Alertmanager, Grafana, and a sample alerting helper script.

## What this repo does

- Installs and manages an Argo CD instance.
- Bootstraps a base Argo CD Application that watches the applications directory in this repository.
- Deploys a kube-prometheus-stack stack with Prometheus, Alertmanager, and Grafana.
- Adds Prometheus Node Exporter.
- Includes a Grafana dashboard for Argo CD.
- Provides a helper script to send sample alerts to Alertmanager.

## Repository layout

- argo/base/base.yaml: the top-level Argo CD Application that points Argo CD at the applications directory.
- argo/applications/: child applications managed by the base Application.
  - argo/applications/kustomization.yaml: aggregates the child Applications.
  - argo/applications/prometheus/application.yaml: installs kube-prometheus-stack with custom scrape config for Argo CD metrics.
  - argo/applications/prometheus-node-exporter/application.yaml: installs Prometheus Node Exporter.
  - argo/applications/prometheus-operator-crds/application.yaml: installs the required CRDs for kube-prometheus-stack.
  - argo/applications/grafana/dashboards/argo-cd.yaml: imports an Argo CD Grafana dashboard.
- argo/trigger_alert.sh: sends a firing and resolved alert to Alertmanager for testing.

## Prerequisites

- A Kubernetes cluster
- kubectl installed and configured
- Helm 3 installed
- Argo CD installed in the cluster before applying the base Application

## Install Argo CD

Create the namespace:

```bash
kubectl create namespace argocd
```

Add the Argo CD Helm repository:

```bash
helm repo add argo https://argoproj.github.io/argo-helm
```

Install Argo CD:

```bash
helm upgrade -i argocd --namespace argocd \
  --set redis.exporter.enabled=true \
  --set redis.metrics.enabled=true \
  --set server.metrics.enabled=true \
  --set controller.metrics.enabled=true \
  argo/argo-cd
```

Check the pods:

```bash
kubectl get pods -n argocd
```

## Bootstrap the GitOps applications

After Argo CD is running, apply the base Application:

```bash
kubectl apply -f argo/base/base.yaml
```

This tells Argo CD to watch the applications directory and create the child Applications for Prometheus, Alertmanager, Grafana, and the monitoring dependencies.

## Access the Argo CD UI

Port-forward the Argo CD server:

```bash
kubectl port-forward service/argocd-server -n argocd 8080:443
```

Retrieve the initial admin password:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

Then open:

```
https://localhost:8080/
```

If you are using kind or a local cluster, you may need to adjust your cluster configuration so port 8080 is exposed.

## Access the prometheus UI

```bash
kubectl port-forward service/kube-prometheus-stack-prometheus -n argocd 9090:9090
```

```text
http://localhost:9090/
```

## Access grafana UI

```bash
kubectl port-forward service/kube-prometheus-stack-grafana -n argocd 9092:80
```

Granafa secret:

```bash
kubectl get secret \
-n argocd \
kube-prometheus-stack-grafana \
-o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

```text
http://localhost:9092/
```

## ArgoCD cli with kind

```bash
kubectl config get-contexts -o name
```

```bash
argocd cluster add kind-kind
```

```bash
argocd login localhost:8080
```

```bash
andrew@andrews-MacBook-Pro kind % argocd app list
NAME                               CLUSTER                         NAMESPACE  PROJECT  STATUS  HEALTH   SYNCPOLICY  CONDITIONS  REPO                                                     PATH                                TARGET
argocd/base                        https://kubernetes.default.svc  argocd     default  Synced  Healthy  Auto-Prune  <none>      https://github.com/andrewstewartelliott-lang/argocd      argo/applications/                  main
argocd/kube-prometheus-stack       in-cluster                      argocd     default  Synced  Healthy  Auto-Prune  <none>      https://prometheus-community.github.io/helm-charts                                           45.6.0
argocd/kube-prometheus-stack-crds  in-cluster                      argocd     default  Synced  Healthy  Auto-Prune  <none>      https://github.com/prometheus-community/helm-charts.git  charts/kube-prometheus-stack/crds/  kube-prometheus-stack-44.3.0
argocd/kube-state-metrics          in-cluster                      argocd     default  Synced  Healthy  Auto-Prune  <none>      https://prometheus-community.github.io/helm-charts                                           7.4.0
argocd/prometheus-node-exporter    in-cluster                      argocd     default  Synced  Healthy  Auto-Prune  <none>      https://prometheus-community.github.io/helm-charts                                           4.13.0
```

## Test Alertmanager

A small helper script is included to trigger an alert through Alertmanager:

```bash
bash argo/trigger_alert.sh
```

This sends a sample alert to Alertmanager and then resolves it, which is useful for verifying alert flow.





helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install kube-state-metrics prometheus-community/kube-state-metrics \
  --namespace monitoring --create-namespace








  ## Notes

- The base Application references the GitHub repository URL in argo/base/base.yaml. If you fork this repository or use a different repository name, update that value to match your environment.
- The monitoring stack is configured to scrape Argo CD metrics from the cluster service endpoints.