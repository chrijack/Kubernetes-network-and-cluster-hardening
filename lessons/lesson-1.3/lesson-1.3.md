# Lesson 1.3 — Kubernetes Network Policies and Cilium

> From the [Certified Kubernetes Security Specialist (CKS) Video Course](https://www.pearsonitcertification.com/store/certified-kubernetes-security-specialist-cks-video-9780138296476)

## Objective
Deploy a multi-tier application (frontend, backend, database) on Kubernetes and progressively enforce network policies using Cilium to restrict traffic flows between components, demonstrating how to secure east-west communication while monitoring with Hubble.

## Prerequisites
- Minikube installed and configured
- kubectl CLI available
- Cilium CLI tools installed
- Access to the deployment YAML files in this lesson directory
- Understanding of basic Kubernetes concepts (namespaces, pods, services)

## Steps

### Step 1: Setup Minikube and Cilium

Start a Minikube cluster with multiple nodes and enable the Cilium CNI for network policy enforcement.

```bash
minikube start --nodes 2 --kubernetes-version=v1.29.4
```

Enable Ingress and DNS addons:

```bash
minikube addons enable ingress
minikube addons enable ingress-dns
```

Install Cilium and enable Hubble for network observability:

```bash
cilium install
cilium hubble enable
cilium status --wait
cilium hubble port-forward&
```

### Step 2: Deploy the Three-Tier Application

Create a dedicated namespace for the application:

```bash
kubectl create namespace open-app
```

Deploy the frontend, backend, and database components:

```bash
kubectl apply -f backend-deployment.yaml
kubectl apply -f frontend-deployment.yaml
kubectl apply -f database-deployment.yaml
```

### Step 3: Verify Initial Connectivity (Before Policies)

Test frontend-to-backend communication:

```bash
kubectl exec $(kubectl get pods -l component=frontend -n open-app -o jsonpath="{.items[0].metadata.name}") -n open-app -- curl --connect-timeout 5 http://backend-service:5000
```

Test backend-to-database connectivity:

```bash
kubectl exec -it $(kubectl get pods -l component=backend -n open-app -o jsonpath="{.items[0].metadata.name}" --field-selector=status.phase=Running | head -n 1) -n open-app -- psql -h database-service -U postgres
```

Test frontend-to-database connectivity:

```bash
kubectl exec -it $(kubectl get pods -l component=frontend -n open-app -o jsonpath="{.items[0].metadata.name}" --field-selector=status.phase=Running | head -n 1) -n open-app -- psql -h database-service -U postgres
```

### Step 4: Apply Default Deny-All Policy

Apply a default deny-all network policy to block all traffic by default:

```bash
k apply -f default_deny_all.yaml
```

Verify that the deny policy blocks the frontend-to-backend connection:

```bash
kubectl exec $(kubectl get pods -l component=frontend -n open-app -o jsonpath="{.items[0].metadata.name}") -n open-app -- curl --connect-timeout 5 http://backend-service:5000
```

Monitor traffic with Hubble to observe the denied connections:

```bash
hubble observe -n open-app --follow
```

### Step 5: Apply Default Deny with DNS Exception

Apply a policy that allows DNS while continuing to deny all other traffic:

```bash
k apply -f default_deny_all_plus_dns.yaml
```

### Step 6: Apply Frontend Egress and Ingress Policies

Allow the frontend to make outbound requests:

```bash
k apply -f frontend-egress-policy.yaml
k apply -f frontend-ingress-allow-any.yaml
```

### Step 7: Test External Access

Create a temporary pod in the default namespace and attempt to access the frontend service:

```bash
kubectl run test-pod --rm -ti --image=busybox --restart=Never -- wget --spider --timeout 1 http://frontend-service.open-app.svc.cluster.local
```

Verify frontend-to-backend communication again:

```bash
kubectl exec $(kubectl get pods -l component=frontend -n open-app -o jsonpath="{.items[0].metadata.name}") -n open-app -- curl --connect-timeout 5 http://backend-service:5000
```

### Step 8: Apply Backend Policies

Allow frontend to reach backend:

```bash
k apply -f frontend-to-backend-ingress-policy.yaml
```

Allow backend to reach database:

```bash
k apply -f backend-egress-to-database-policy.yaml
```

### Step 9: Apply Database Ingress Policy

Restrict database access to only authorized sources:

```bash
k apply -f database-ingress-policy.yaml
```

### Step 10: Verify Multi-Tier Connectivity

Test backend-to-database connection:

```bash
kubectl exec -it $(kubectl get pods -l component=backend -n open-app -o jsonpath="{.items[0].metadata.name}" --field-selector=status.phase=Running | head -n 1) -n open-app -- psql -h database-service -U postgres
```

Verify that frontend cannot directly access the database (should be blocked):

```bash
kubectl exec -it $(kubectl get pods -l component=frontend -n open-app -o jsonpath="{.items[0].metadata.name}" --field-selector=status.phase=Running | head -n 1) -n open-app -- psql -h database-service -U postgres
```

### Step 11: Analyze Policy Coverage with Cyclonus

Query the network policies to analyze what traffic is allowed by your policies:

```bash
kubectl cyclonus --mode query-target -n open-app
```

This helps visualize the full policy matrix and identify any gaps or unintended permissions.

## Verification

You should observe the following outcomes:

1. **Before policies**: All services can communicate with each other
2. **After default deny**: No inter-service communication succeeds
3. **After frontend policies**: Frontend can initiate outbound traffic but cannot accept all inbound
4. **After tiered policies**: Only specified traffic paths succeed (frontend→backend, backend→database)
5. **Hubble monitoring**: Denied connections show up in the Hubble flow logs with appropriate drop reasons
6. **Cyclonus analysis**: Policies allow only the intended communication flows

## Cleanup

To remove the application and policies:

```bash
kubectl delete namespace open-app
cilium hubble port-forward&  # Stop the port-forward if running
minikube stop
```

To fully delete the Minikube cluster:

```bash
minikube delete
```
