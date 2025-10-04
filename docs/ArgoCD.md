# ⚡ GitOps Continuous Delivery with ArgoCD

This guide demonstrates how to deploy and manage the MERN stack application using ArgoCD for GitOps continuous delivery, enabling automated synchronization between Git repository and Kubernetes cluster.

## Overview

ArgoCD provides declarative GitOps continuous delivery for Kubernetes applications. This setup enables:
- Automated deployment from Git repository
- Real-time synchronization with cluster state
- Easy rollback and application lifecycle management
- Multi-environment deployment capabilities

## Cluster Configuration

```yaml
# kind-config.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 30001
    hostPort: 30001
    protocol: TCP
```

## 1. Install and Configure ArgoCD

### Create ArgoCD Namespace
```bash
kubectl create namespace argocd
```

### Install ArgoCD
```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Verify Installation
```bash
watch kubectl get pods -n argocd
```
Ensure all pods are in **Running** state.

### Configure Service Access
```bash
# Check services
kubectl get svc -n argocd

# Change to NodePort
kubectl patch svc argocd-server -n argocd \
  -p '{"spec": {"type": "NodePort", "ports": [{"port": 80, "targetPort": 8080, "nodePort": 30001}]}}'

# Confirm change
kubectl get svc -n argocd
```

### Access ArgoCD UI
```bash
# Get NodePort
kubectl get svc argocd-server -n argocd
```
Access at: `http://<public-ip-worker>:<NodePort>`

### Get Admin Credentials
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```
- **Username:** `admin`
- **Password:** Use output from above command

> Change the default password after first login in **User Info**.

## 2. Deploy MERN Application

### Method 1: Using Manifests
```bash
kubectl apply -f ./argocd/project.yml
kubectl apply -f ./argocd/application.yml
```

### Method 2: Using ArgoCD UI

#### Connect to Repository
1. Click **New App** in ArgoCD dashboard
2. Configure application:
   - **Application Name:** `mern-devops`
   - **Project:** `default`
   - **Sync Policy:** `Manual` or `Automatic`
   - Select `Auto-Create Namespace`

![argocd-1](./assets/argocd-1.png)

#### Configure Repository
- **Repository URL:** `https://github.com/atkaridarshan04/CloudNative-DevOps-Blueprint.git`
- **Revision:** `main`
- **Path:** `kubernetes`

![argocd-2](./assets/argocd-2.png)

#### Set Destination
- **Cluster:** Default cluster
- **Namespace:** `mern-devops`

![argocd-3](./assets/argocd-3.png)

#### Deploy Application
1. Click **Create**
2. Sync the application in ArgoCD dashboard

![argocd-4](./assets/terraform_argocd.png)
![argocd-5](./assets/argocd-5.png)
