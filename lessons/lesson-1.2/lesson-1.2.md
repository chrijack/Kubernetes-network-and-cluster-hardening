# Lesson 1.2 — Applying Kubernetes Network Policies with Cilium

> From the [Certified Kubernetes Security Specialist (CKS) Video Course](https://www.pearsonitcertification.com/store/certified-kubernetes-security-specialist-cks-video-9780138296476)

## Objective

Understand how to implement and test Kubernetes Network Policies using Cilium as the network plugin. This lesson demonstrates setting up a Minikube cluster with Cilium, deploying network-connected services, and verifying connectivity restrictions using Network Policy rules.

## Prerequisites

- Minikube installed and running
- `kubectl` command-line tool configured
- `cilium` CLI installed
- Docker or compatible container runtime
- Basic understanding of Kubernetes networking concepts

## Steps

### Step 1: Start Minikube Cluster with Two Nodes

Start a Minikube cluster with two nodes using Kubernetes v1.29.4:

```bash
minikube start --nodes 2 --kubernetes-version=v1.29.4
```

This creates a multi-node Minikube cluster suitable for testing network policies across nodes.

### Step 2: Enable Ingress-DNS Addon

Enable the ingress-DNS addon for Minikube:

```bash
minikube addons enable ingress-dns
```

This addon enables DNS resolution for Ingress resources within the cluster.

### Step 3: Check Initial Cilium Status

Check the status of Cilium in the cluster (it may not be installed yet):

```bash
cilium status
```

### Step 4: Install Cilium

Install Cilium as the Container Networking Interface (CNI) plugin:

```bash
cilium install
```

Cilium provides advanced networking capabilities including Network Policy enforcement and observability.

### Step 5: Enable Cilium Hubble

Enable Cilium Hubble for network visibility and flow monitoring:

```bash
cilium hubble enable
```

Hubble provides real-time insight into network communication between services.

### Step 6: Verify Cilium Installation

Wait for Cilium to be fully ready and check its status:

```bash
cilium status --wait
```

The `--wait` flag ensures the command waits until Cilium is fully operational.

### Step 7: Port Forward for Hubble

Set up port forwarding to access the Hubble UI (runs in background):

```bash
cilium hubble port-forward&
```

This enables access to Hubble's web interface for observing network flows.

### Step 8: Check Hubble Status

Verify that Hubble is operational:

```bash
hubble status
```

### Step 9: Deploy nginx Web Server

Deploy an nginx pod with the `app=web` label and expose it as a service:

```bash
kubectl run web --image=nginx --labels="app=web" --expose --port=80
```

This creates:
- A pod named `web` running nginx
- A service named `web` exposing port 80
- Labels for Network Policy targeting

### Step 10: Test Connectivity with Alpine Pod

Launch an interactive Alpine pod and test connectivity to the nginx service:

```bash
kubectl run --rm -i -t --image=alpine test-$RANDOM -- sh
```

Once inside the Alpine pod shell, test connectivity to the nginx web service:

```bash
wget -qO- --timeout=2 http://web
```

This command attempts to download content from the nginx service using HTTP. A successful response indicates connectivity is allowed.

## Verification

After completing the steps above, you should observe:

1. **Cilium Running**: `cilium status --wait` shows all Cilium components as operational
2. **Hubble Operational**: `hubble status` indicates Hubble is ready
3. **nginx Accessible**: The `wget` command from the Alpine pod returns the nginx default page (HTTP 200)
4. **Network Flows Visible**: Cilium Hubble UI displays network flows between the Alpine test pod and nginx service

## Cleanup

To clean up resources and stop the Minikube cluster:

```bash
minikube delete
```

This removes the Minikube cluster and all deployed resources.

### Network Policy Files Reference

The following YAML files are available in this directory for defining and testing Network Policies:

- `default_deny_all.yml` — Default deny Network Policy for blocking all ingress traffic
- `deny-external-egress.yaml` — Egress policy for restricting outbound connections
- `nginx-deployment.yaml` — Nginx deployment manifest
- `redis-deployment.yaml` — Redis deployment manifest

These files can be applied using `kubectl apply -f <filename>` to enforce specific network policies in your cluster.
