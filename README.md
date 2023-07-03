# Overview
- based on https://github.com/pavolloffay/kubecon-eu-2023-opentelemetry-kubernetes-tutorial
- adapted for minikube, jaeger and prometheus

# Install cluster
```bash
minikube start --memory 8192 --cpus 4
```

```bash
minikube addons enable metrics-server
minikube addons enable ingress
```

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.12.2/cert-manager.yaml
```

# Install Prometheus, Grafana
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

```bash
helm install prometheus prometheus-community/kube-prometheus-stack
```

In an other terminal:
```bash
kubectl port-forward svc/prometheus-grafana 8080:80
```

Access to Grafana http://localhost:8080
```bash
kubectl get secret prometheus-grafana -o jsonpath='{.data.admin-user}' | base64 -d
kubectl get secret prometheus-grafana -o jsonpath='{.data.admin-password}' | base64 -d
```
user/pwd : admin/prom-operator


# Install Jaeger

```bash
kubectl create namespace observability
kubectl apply -f https://github.com/jaegertracing/jaeger-operator/releases/download/v1.46.0/jaeger-operator.yaml -n observability
```

```bash
kubectl apply -f ./backend/01-jaeger.yaml
```

In an other terminal:
```bash
kubectl port-forward svc/simplest-query 16686:16686
```
Access to Jaeger: http://localhost:16686

# Install Otel collector
```bash
kubectl apply -f https://github.com/open-telemetry/opentelemetry-operator/releases/download/v0.80.0/opentelemetry-operator.yaml
```

```bash
kubectl apply -f ./backend/02-collector.yaml
```
Restart the collector if config modified.

# Install Otel operator
```bash
kubectl apply -f ./app/instrumentation.yaml
```

# Install apps
```bash
kubectl apply -f ./app/k8s-annotated.yaml
```
The apps contain frontend, backend in python, backend in java and a load generator.

## Restart apps
If config modified.
```bash
kubectl rollout restart deployment -n tutorial-application -l app=backend1
kubectl rollout restart deployment -n tutorial-application -l app=backend2
kubectl rollout restart deployment -n tutorial-application -l app=frontend
```