---
title: "How to Set Up ArgoCD for Kubernetes Continuous Deployment"
date: 2024-12-11
tags: ["ArgoCD", "Kubernetes", "DevOps", "CI/CD"]
draft: false
---

ArgoCD is a powerful GitOps tool designed to automate Kubernetes deployments. It provides a declarative approach to continuous deployment, ensuring your Kubernetes clusters are always in sync with your Git repository.

In this guide, we’ll walk through setting up ArgoCD in a Kubernetes cluster and deploying a sample application.

---

## **Prerequisites**

Before getting started, ensure you have:
- A running **Kubernetes cluster**.
- **kubectl** installed and configured to access your cluster.
- Administrative access to the cluster (for namespace creation and permissions).
- A **Git repository** containing Kubernetes manifests or Helm charts.

---

## **Step 1: Install ArgoCD**

1. Add the `argocd` namespace to your Kubernetes cluster:

```bash
kubectl create namespace argocd
```

2. Install ArgoCD using the official manifests:

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml 
```

3. Verify that the ArgoCD pods are running:

```bash
kubectl get pods -n argocd
```

You should see pods like argocd-server, argocd-repo-server, argocd-application-controller, and argocd-dex-server in a Running state.

## **Step 2: Expose the ArgoCD Server**
ArgoCD’s server needs to be accessible for managing your deployments. Choose one of the following methods to expose it:

Option 1: Port Forwarding (For Local Development)
Run the following command to access the ArgoCD server locally:

```shell
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
Visit https://localhost:8080 in your browser.

Option 2: LoadBalancer or Ingress (For Production)
Edit the argocd-server service to use a LoadBalancer:

```shell
kubectl edit svc argocd-server -n argocd
```

Change the type to LoadBalancer and save. The external IP will be assigned by your cloud provider:

```shell
kubectl get svc -n argocd
```

## **Step 3: Log in to ArgoCD**

1. Retrieve the initial admin password:

```shell
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

2. Log in to the ArgoCD UI using the admin username and password:

## **Step 4: Configure a Git Repository**

1. Add your Git repository to ArgoCD:
```shell
argocd repo add https://github.com/your-repo/kubernetes-manifests.git \
    --username <your-username> \
    --password <your-password>
```

2. Verify the repository was added successfully:

```shell
argocd repo list
```

## **Step 5: Deploy an Application**
1. Create an ArgoCD Application that syncs your Git repository to a Kubernetes namespace:

```shell
kubectl apply -f - <<EOF
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: sample-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/your-repo/kubernetes-manifests.git
    targetRevision: HEAD
    path: manifests/sample-app
  destination:
    server: https://kubernetes.default.svc
    namespace: sample-app
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
EOF
```

2. Verify the application status:

```shell
argocd app get sample-app
```

3. Check if the application is synced and running:

```shell
kubectl get all -n sample-app
```

## **Step 6: Automating Syncing**

Enable automated syncing to ensure that the cluster always matches the desired state in the Git repository. This can be done by setting syncPolicy.automated in the application manifest.

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
```
Prune: Automatically delete resources no longer defined in Git.
SelfHeal: Reconcile cluster state if drift occurs.