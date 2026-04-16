# Lesson 2.2 — Kubernetes Ingress with Cilium and Hubble Observability

> From the [Certified Kubernetes Security Specialist (CKS) Video Course](https://www.pearsonitcertification.com/store/certified-kubernetes-security-specialist-cks-video-9780138296476)

## Objective
Deploy a multi-node Kubernetes cluster with Cilium networking and observe ingress traffic using Hubble. This lesson demonstrates how to route traffic to multiple applications using Kubernetes Ingress while leveraging network observability.

## Prerequisites
- Minikube installed and configured
- kubectl installed and accessible
- Basic understanding of Kubernetes deployments and services
- Familiarity with Cilium as a CNI plugin

## Steps

### Step 1: Start Minikube with Multiple Nodes
Create a multi-node Minikube cluster running Kubernetes v1.29.4.

```bash
minikube start --nodes 2 --kubernetes-version=v1.29.4
```

### Step 2: Enable Ingress Addon
Enable the Nginx Ingress Controller addon in Minikube.

```bash
minikube addons enable ingress
```

### Step 3: Install and Configure Cilium
Install Cilium as the network plugin and enable Hubble for network observability.

```bash
cilium install
cilium hubble enable
cilium status --wait
cilium status
```

The `--wait` flag ensures the Cilium components are fully ready before proceeding.

### Step 4: Deploy Applications
Apply the application deployments to the cluster.

```bash
kubectl apply -f app1-deployment.yml
kubectl apply -f app2-deployment.yml
```

### Step 5: Create Ingress Resource
Deploy the ingress configuration to route traffic to the applications.

```bash
kubectl apply -f app-ingress.yml
```

### Step 6: Verify Ingress Setup
Check the ingress controller service and inspect the ingress resource details.

```bash
kubectl get svc -n ingress-nginx
kubectl describe ingress example-ingress
```

### Step 7: Enable Traffic Forwarding
Start the Minikube tunnel to enable access to ingress services from your local machine.

```bash
minikube tunnel
```

### Step 8: Observe Network Traffic with Hubble
Monitor network flows and traffic patterns using Hubble observability.

```bash
hubble observe
```

### Step 9: Test Application Connectivity
Verify that both applications are accessible through the ingress controller using host header routing.

```bash
curl -H "Host: app1.com" http://127.0.0.1
curl -H "Host: app2.com" http://127.0.0.1
```

## Verification
Both curl commands should return successful responses from their respective applications (app1 and app2). The Hubble observability tool should display network flows showing traffic routing through the ingress controller to the backend services.

## Cleanup
To clean up the demonstration:

```bash
minikube delete
```

This removes the Minikube cluster and all deployed resources.
