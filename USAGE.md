# argocd
Project inspired by https://github.com/naturalett/continuous-delivery

# Create a namespace
```
kubectl create namespace argocd
```

# Add the ArgoCD Helm Chart
```
helm repo add argo https://argoproj.github.io/argo-helm
```

# Install the ArgoCD
```
helm upgrade -i argocd --namespace argocd --set redis.exporter.enabled=true --set redis.metrics.enabled=true --set server.metrics.enabled=true --set controller.metrics.enabled=true argo/argo-cd
```

# Check the status of the pods
```
kubectl get pods -n argocd
```

# Proxy the connection
```
kubectl port-forward service/argocd-server -n argocd 8080:443
```

# Get admin secret
```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

# Access UI
```
https://localhost:8080/
```

If you're using kind (Kubernetes in docker) you may have to alter your config.yaml for exposing :8080 in the cluster.


