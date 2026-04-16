# Lesson 3.1 — Securing the Kubelet in Kubernetes

> From the [Certified Kubernetes Security Specialist (CKS) Video Course](https://www.pearsonitcertification.com/store/certified-kubernetes-security-specialist-cks-video-9780138296476)

## Objective

Understand how the kubelet operates in Kubernetes, identify security vulnerabilities in the kubelet API, and learn how to properly configure and secure the kubelet to protect your Kubernetes cluster.

## Prerequisites

- Access to a Kubernetes cluster with at least one worker node
- `kubectl` configured and authenticated
- SSH access to a Kubernetes node
- Basic familiarity with Kubernetes concepts (nodes, pods, API server)
- `curl` and `jq` installed on your node

## How the Kubelet Works

The kubelet is the primary "node agent" that runs on each Kubernetes node. It is responsible for:

1. **Node Registration**: Registering the node with the Kubernetes API server using one of:
   - The hostname
   - A flag to override the hostname
   - Specific logic for a cloud provider

2. **Pod Management**: Working with PodSpecs (YAML or JSON objects that describe pods). The kubelet gets PodSpecs through various mechanisms, primarily from the API server.

3. **Container Lifecycle**: Ensuring that the containers described in those PodSpecs are running and healthy. It does not manage containers not created by Kubernetes.

4. **Alternative Sources**: Besides getting PodSpecs from the API server, the kubelet can also get container manifests via:
   - A file path passed as a flag on the command line (files are monitored for periodic updates)
   - An HTTP endpoint passed as a parameter on the command line (checked every 20 seconds)

## Configuring the Kubelet

### Step 1: Locate the Kubelet Process

First, identify the running kubelet process on your node:

```bash
ps -aux | grep kubelet
```

### Step 2: Create or Edit the Kubelet Configuration File

The kubelet can be configured via a configuration file, which is the recommended approach as it simplifies node deployment and configuration management.

Create or edit a kubelet configuration YAML file at `/var/lib/kubelet/config.yaml`:

```bash
sudo nano /var/lib/kubelet/config.yaml
```

This file can contain parameters to configure kubelet behavior. Key configuration parameters include:

- `staticPodPath`: The directory where the kubelet looks for static pod manifests
- `authentication`: Configure authentication for the kubelet's API server
- `authorization`: Configure authorization for the kubelet's API server
- `clusterDNS`: The IP address for a cluster DNS server

### Step 3: Update the Kubelet Service Configuration

Add a line in the kubelet systemd service file (usually at `/etc/systemd/system/kubelet.service.d`) to point to the config file:

```
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
```

### Step 4: Restart the Kubelet Service

Apply the configuration changes:

```bash
systemctl daemon-reload
systemctl restart kubelet
```

## Security Demo: Probing the Kubelet API

The kubelet exposes an API on port 10250 that can be exploited if not properly secured. This demo shows how to identify these vulnerabilities.

### Step 5: Test Kubelet API Access (HTTP)

Attempt to access the kubelet API over unencrypted HTTP:

```bash
curl -sk http://localhost:10250/pods
```

### Step 6: Probe the Kubelet API (HTTPS - Localhost)

Access the kubelet API over HTTPS on the local node:

```bash
curl --insecure -sk https://localhost:10250/pods
```

### Step 7: Retrieve Kubelet Configuration

Retrieve the running kubelet configuration:

```bash
curl -sk https://localhost:10250/configz | jq .
```

### Step 8: List Running Pods (Localhost)

List all running pods on the node via the kubelet API:

```bash
curl -sk https://localhost:10250/runningpods/ | jq .
```

### Step 9: Identify Node IP Address

Get the IP address of your node:

```bash
kubectl get nodes -o wide
```

### Step 10: Probe Kubelet API from Network

Using the node IP address identified above (example: 192.168.56.11), attempt to access the kubelet API remotely:

```bash
curl -sk https://192.168.56.11:10250/runningpods/ | jq .
```

## Reference: Common Kubelet API Endpoints

The following are reference examples showing common kubelet API endpoints and operations. Replace `<kubelet>` with your actual kubelet address or hostname:

```bash
$ curl -k "https://<kubelet>:10250/runningpods" 
# Lists information about running pods

$ curl -k "https://<node_ip>:10250/configz"
# Returns kubelet configuration info

$ curl -k -XPOST "https://<kubelet>:10250/run/<namespace>/<pod>/<container>" -d "cmd=<command>"
# Executes a given command within a pod
```

## Securing the Kubelet: Best Practices

Implement the following security controls to protect your kubelet:

### 1. Disable Anonymous Access

Disable anonymous access to prevent unauthenticated requests:

```yaml
anonymous-auth: false
```

Unauthenticated requests will receive an error response.

### 2. Implement Kubernetes RBAC

Use Kubernetes RBAC to authenticate and authorize access to the kubelet API endpoints. Set the authorization mode to a value other than `AlwaysAllow`:

```yaml
authorization-mode: Webhook
```

This verifies that requests are properly authorized.

### 3. Use NodeRestriction Admission Controller

Include `NodeRestriction` in the API server `--admission-control` setting to restrict kubelet permissions. This only allows the kubelet to modify pods bound to its own node:

```bash
--admission-control=...,NodeRestriction,...
```

### 4. Close Read-Only Ports

Disable the read-only port by setting it to 0:

```yaml
read-only-port: 0
```

### 5. Implement Node Restrictions

Limit the node and pod objects the kubelet can access by using the `NodeRestriction` admission controller along with appropriate RBAC policies.

### 6. Enable Kubelet Client Certificate Rotation

Rotate kubelet client certificates by enabling the `RotateKubeletClientCertificate` feature gate in your kubelet configuration.

### 7. Use Firewall Rules

Restrict access to the kubelet port 10250 and read-only port 10255 using firewall rules. Limit access to only authorized components within your infrastructure.

### 8. Keep Kubelet Updated

Keep the kubelet version up-to-date to ensure latest security patches are applied.

### 9. Protect Kernel Defaults

Enable kernel default protection to prevent kernel parameters from being overridden:

```yaml
protect-kernel-defaults: true
```

### 10. Secure the Kubelet Configuration File

Secure and limit access to the kubelet config file:

- Set permissions to 644 or more restrictive
- Set ownership to `root:root`

## Verification

After implementing security measures, verify the kubelet is properly configured:

1. Check that anonymous access is disabled
2. Verify that RBAC authorization is enabled
3. Confirm that read-only ports are closed
4. Test that unauthorized curl requests receive proper error responses

Example of a properly secured kubelet response to anonymous requests:

```bash
curl -sk https://localhost:10250/pods
# Should return an authentication error, not pod data
```

## Summary

By properly configuring the kubelet and implementing these security best practices, you can significantly enhance the security posture of your Kubernetes nodes and the overall cluster. The kubelet API is a critical attack surface—always ensure it is properly authenticated, authorized, and access-controlled.
