# Lesson 8.2 — Installing Metrics Server and Kubernetes Dashboard with RBAC

> From the [Certified Kubernetes Security Specialist (CKS) Video Course](https://www.pearsonitcertification.com/store/certified-kubernetes-security-specialist-cks-video-9780138296476)

## Objective
Deploy Metrics Server to enable resource monitoring, and install Kubernetes Dashboard with multiple service accounts configured with different RBAC roles (admin and read-only access).

## Prerequisites
- A running Kubernetes cluster
- `helm` CLI installed and configured
- `kubectl` access to your cluster

## Steps

### Step 1: Install Metrics Server via Helm

Add the Metrics Server Helm repository:

```bash
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/
```

Search for available Metrics Server versions:

```bash
helms search repo metrics-server
```

Review the Helm chart values for version 3.12.1:

```bash
helm show values metrics-server/metrics-server --version 3.12.1
```

Create a dedicated namespace for Metrics Server:

```bash
kubectl create ns metrics
```

Install Metrics Server using Helm (with a custom values file):

```bash
helm upgrade --install metrics-server metrics-server/metrics-server --namespace metrics -f values.yaml
```

### Step 2: Verify Metrics Server Installation

Check metrics for pods in your cluster:

```bash
kubectl top pods
```

Check metrics for nodes in your cluster:

```bash
kubectl top nodes
```

### Step 3: Install Kubernetes Dashboard via Helm

Add the Kubernetes Dashboard Helm repository:

```bash
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
```

Search for available Kubernetes Dashboard versions:

```bash
helm search repo kubernetes-dashboard
```

Export the default values to a file for customization:

```bash
helm show values kubernetes-dashboard/kubernetes-dashboard > dashboard_values.yaml
```

Edit the dashboard values as needed:

```bash
nano dashboard_values.yaml
```

Install Kubernetes Dashboard using Helm:

```bash
helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard -f dashboard_values.yaml
```

### Step 4: Access the Dashboard

Create a port-forward tunnel to access the dashboard:

```bash
kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard-kong-proxy 8443:443
```

Open the dashboard in your browser:

```
https://localhost:8443
```

### Step 5: Create Admin User for Dashboard Access

Create a service account for dashboard admin access:

```bash
cat <<EOF | k apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
EOF
```

Create a ClusterRoleBinding to grant admin privileges to the service account:

```bash
cat <<EOF | k apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF
```

Generate a token for the admin user:

```bash
kubectl -n kubernetes-dashboard create token admin-user
```

**Log in to the dashboard** using the token generated above.

**Note:** The older method to retrieve the token was:
```bash
#kubectl get secret admin-user -n kubernetes-dashboard -o jsonpath={".data.token"} | base64 -d
```

### Step 6: Create Read-Only User for Dashboard Access

Create a Role with read-only permissions for pods, services, and related resources:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: kubernetes-dashboard
  name: dashboard-read-only-role
rules:
- apiGroups: [""]
  resources: ["pods", "services", "configmaps", "secrets", "deployments", "replicasets", "pods/log"]
  verbs: ["get", "list", "watch"]
EOF
```

Create a service account for the read-only user:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dashboard-read-only-sa
EOF
```

Bind the read-only role to the service account:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dashboard-read-only-role-binding
  namespace: kubernetes-dashboard
subjects:
- kind: ServiceAccount
  name: dashboard-read-only-sa
  apiGroup: ""
roleRef:
  kind: Role
  name: dashboard-read-only-role
  apiGroup: ""
EOF
```

Generate a token for the read-only user:

```bash
kubectl get secret $(kubectl get serviceaccount dashboard-read-only-sa -o jsonpath="{.secrets[0].name}") -o jsonpath="{.data.token}" | base64 --decode
```

### Step 7: Create Admin User with Namespace-Level Permissions

Create a service account for namespace-level admin access:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dashboard-admin
  namespace: kubernetes-dashboard
EOF
```

Create a RoleBinding to grant admin role permissions within the kubernetes-dashboard namespace:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dashboard-admin-binding
  namespace: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: admin
subjects:
- kind: ServiceAccount
  name: dashboard-admin
  namespace: kubernetes-dashboard
EOF
```

Create a ClusterRoleBinding to allow the admin user to list namespaces:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: dashboard-admin-list-namespace-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: list-namespace
subjects:
- kind: ServiceAccount
  name: dashboard-admin
  namespace: kubernetes-dashboard
EOF
```

## Verification

- Confirm Metrics Server is collecting data: `kubectl top pods` and `kubectl top nodes` return resource metrics
- Verify dashboard is accessible at `https://localhost:8443`
- Test each service account token by logging in to the dashboard with admin, read-only, and namespace-admin credentials
- Confirm that read-only user cannot modify resources
- Confirm that namespace-admin user has appropriate permissions within the kubernetes-dashboard namespace

## Cleanup

To remove the components deployed in this lesson:

```bash
helm uninstall kubernetes-dashboard --namespace kubernetes-dashboard
helm uninstall metrics-server --namespace metrics
kubectl delete namespace kubernetes-dashboard
kubectl delete namespace metrics
```
