# Prometheus with Helm and ArgoCD on Kubernetes

This repository documents deploying Prometheus using Helm, customizing configurations, and integrating with ArgoCD for GitOps-based management.

---

## Prerequisites

- Kubernetes cluster with `kubectl` configured.
- Helm and ArgoCD installed on the cluster.
- Permissions to deploy resources.

---

## Installation

### 1. Deploy Prometheus with Helm

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/prometheus --namespace monitoring --create-namespace
```

---

### 2. Customize Configuration

1. Modify `values.yaml` for custom scrape jobs or persistent storage:


2. Apply updates:
   ```bash
   helm upgrade prometheus prometheus-community/prometheus -n monitoring -f values.yaml
   ```

---

### 3. Integrate with ArgoCD

1. Add Prometheus as an ArgoCD application (`prometheus-argocd-app.yaml`):
   ```yaml
   apiVersion: argoproj.io/v1alpha1
   kind: Application
   metadata:
     name: prometheus
     namespace: argocd
   spec:
     source:
       repoURL: https://github.com/prometheus-community/helm-charts
       targetRevision: main
       chart: prometheus
       helm:
         valueFiles:
           - custom-values.yaml
     destination:
       server: https://kubernetes.default.svc
       namespace: monitoring
     syncPolicy:
       automated:
         prune: true
         selfHeal: true
   ```

2. Apply:
   ```bash
   kubectl apply -f prometheus-argocd-app.yaml -n argocd
   ```

---
![image](https://github.com/user-attachments/assets/9d5c9dca-63be-47ec-b477-22e5e9ed8458)

## Notifications with ArgoCD

1. Configure `argocd-notifications-cm.yaml` for Prometheus-specific events:
   ```yaml
   triggers:
     - name: on-sync-success-prometheus
       condition: app.metadata.name == 'prometheus' && app.status.operationState.phase == 'Succeeded'
       template: sync-success
   ```

2. Apply changes:
   ```bash
   kubectl apply -f argocd-notifications-cm.yaml -n argocd
   ```

---

## Access Prometheus

1. Port-forward:
   ```bash
   kubectl port-forward -n monitoring svc/prometheus-server 9090:80
   ```






Let me know if you'd like to add or remove anything!
