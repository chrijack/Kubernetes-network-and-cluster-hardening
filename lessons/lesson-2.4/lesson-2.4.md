# Lesson 2.4 — Validating and Troubleshooting TLS Ingress Configuration

> From the [Certified Kubernetes Security Specialist (CKS) Video Course](https://www.pearsonitcertification.com/store/certified-kubernetes-security-specialist-cks-video-9780138296476)

## Objective
Learn how to validate TLS Ingress configuration, verify backend connectivity, test secure connections, and troubleshoot common issues.

## Prerequisites
- Kubernetes cluster with ingress-nginx controller installed
- TLS Secret (myapp-tls) created and configured
- Backend applications (app1, app2) deployed with corresponding services
- kubectl and curl command-line tools available

## Steps

### Step 1: Verify Ingress Controller Health
Check the status and logs of the Ingress controller pods:

```bash
kubectl get pods -n ingress-nginx
kubectl logs ingress-nginx-controller-65496f9567-gktmm -n ingress-nginx
```

### Step 2: Inspect Ingress Resource Configuration
Verify the Ingress resource is properly configured:

```bash
kubectl describe ingress app-ingress-tls
```

### Step 3: Review NGINX Configuration
Inspect the generated NGINX configuration from the controller:

```bash
kubectl exec ingress-nginx-controller-65496f9567-gktmm -n ingress-nginx -- cat /etc/nginx/nginx.conf
```

### Step 4: Check Backend Health
Verify that backend Pods and Services are running and accessible:

```bash
kubectl get pods -l app=app1
kubectl get pods -l app=app2
kubectl describe service app1-svc
kubectl describe service app2-svc
```

### Step 5: Verify TLS Secret
Confirm the TLS Secret is present and properly configured:

```bash
kubectl get secret myapp-tls
```

### Step 6: Test HTTP Connectivity
Test connectivity to the Ingress using HTTP:

```bash
curl -H "Host: myapp.com" http://127.0.0.1/app1
curl -H "Host: myapp.com" http://127.0.0.1/app2
```

### Step 7: Test HTTPS Connectivity
Test the secure Ingress with TLS:

```bash
curl -kv https://myapp.com/app1
curl -kv https://myapp.com/app2
```

**Note:** The `-k` flag disables certificate verification for testing purposes with self-signed certificates. These commands assume the Ingress controller is accessible on 127.0.0.1 locally.

### Step 8: Verify TLS Handshake
Check the TLS handshake and certificate details:

```bash
openssl s_client -connect 127.0.0.1:443 -servername myapp.com
```

### Step 9: Verify RBAC Permissions
Confirm the Ingress controller has the necessary RBAC permissions:

```bash
kubectl get clusterrole,clusterrolebinding -n ingress-nginx | grep nginx
```

## Verification
All commands should execute successfully without errors. HTTP requests should reach the backend services, and HTTPS requests should establish a TLS connection and complete the handshake with the configured certificate.

## Cleanup
Remove or update the Ingress resource and TLS Secret when testing is complete:

```bash
kubectl delete ingress app-ingress-tls
kubectl delete secret myapp-tls
```
