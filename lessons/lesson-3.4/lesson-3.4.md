# Lesson 3.4 — Kubelet Security Verification & Hardening

> From the [Certified Kubernetes Security Specialist (CKS) Video Course](https://www.pearsonitcertification.com/store/certified-kubernetes-security-specialist-cks-video-9780138296476)

## Objective

Understand and verify the kubelet's role as the primary node agent in Kubernetes, then configure and secure it using authentication, authorization, and access controls. This lesson focuses on hands-on verification and hardening techniques for the kubelet API.

## Prerequisites

- Access to a Kubernetes cluster with kubectl configured
- SSH access to at least one worker node
- `curl` and `jq` utilities installed on the control plane or worker node
- Understanding of Kubernetes authentication and RBAC (lesson 7.1 recommended)
- sudo/root access on the node

## How the Kubelet Works

The kubelet is the primary "node agent" that runs on each Kubernetes node. It is responsible for:

1. **Node registration**: Registers the node with the Kubernetes API server using the hostname or cloud provider-specific logic
2. **Pod management**: Works with PodSpecs (YAML/JSON objects describing pods) obtained primarily from the API server
3. **Container health**: Ensures containers described in PodSpecs are running and healthy (does not manage containers created outside Kubernetes)
4. **Additional manifest sources**: Can retrieve container manifests via:
   - File path passed as a command-line flag (monitored for periodic updates)
   - HTTP endpoint passed as a command-line parameter (checked every 20 seconds)

## Steps

### Step 1: Verify Kubelet Process and Configuration Files

**On a worker node**, check the running kubelet process:

```bash
ps -aux | grep kubelet
```

Examine the kubelet configuration file (typically at `/var/lib/kubelet/config.yaml`):

```bash
sudo nano /var/lib/kubelet/config.yaml
```

Review the API server configuration to understand how the kubelet interacts with the control plane:

```bash
sudo nano /etc/kubernetes/manifests/kube-apiserver.yaml
```

### Step 2: Configure Kubelet Settings

The kubelet can be configured via a configuration file, which is the recommended approach for simplified node deployment and management.

Create or edit the kubelet configuration YAML file at `/var/lib/kubelet/config.yaml`. This file should contain parameters to configure kubelet behavior.

Add a line in the kubelet systemd service file (typically at `/etc/systemd/system/kubelet.service.d/`) to point to this config file:

```
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
```

After making configuration changes, reload and restart the kubelet service:

```bash
systemctl daemon-reload
systemctl restart kubelet
```

### Step 3: Verify Kubelet API Access — Read-Only Port

Test unauthenticated access to the kubelet's read-only port (10255) on localhost:

```bash
curl -sk http://localhost:10250/pods
```

**Note**: Port 10250 is the primary kubelet API server port. The read-only port (10255) is typically disabled in modern Kubernetes clusters for security reasons.

### Step 4: Verify Kubelet API Access — Secure Port

Test access to the kubelet's secure port (10250) using HTTPS:

```bash
curl --insecure -sk https://localhost:10250/pods
```

Query the kubelet configuration via the secure port:

```bash
curl -sk https://localhost:10250/configz | jq .
```

Get information about running pods:

```bash
curl -sk https://localhost:10250/runningpods/ | jq .
```

### Step 5: Verify Kubelet Access from Remote Nodes

First, list your cluster nodes to identify IP addresses:

```bash
kubectl get nodes -o wide
```

Query the kubelet on a remote node (substitute the actual node IP):

```bash
curl -sk https://192.168.56.11:10250/runningpods/ | jq .
```

**Generic reference commands:**

```bash
# Lists information about running pods on a specific node
curl -k "https://<node_ip>:10250/runningpods"

# Returns kubelet configuration info
curl -k "https://<node_ip>:10250/configz"

# Executes a given command within a pod (requires authorization)
curl -k -XPOST "https://<node_ip>:10250/run/<namespace>/<pod>/<container>" -d "cmd=<command>"
```

### Step 6: Implement Kubelet Security Hardening

Apply these security best practices to harden the kubelet:

**1. Disable anonymous access:**
   - Set `--anonymous-auth=false` in the kubelet configuration so unauthenticated requests receive an error response

**2. Enable authentication and authorization:**
   - Use Kubernetes RBAC to authenticate and authorize access to kubelet API endpoints
   - Set `--authorization-mode` to a value other than `AlwaysAllow` to verify requests are authorized

**3. Restrict kubelet permissions at the API server:**
   - Include `NodeRestriction` in the API server's `--admission-control` setting
   - This allows the kubelet to modify only pods bound to its own node

**4. Disable read-only ports:**
   - Set `--read-only-port=0` to close read-only ports and prevent unauthenticated access

**5. Limit node and pod access:**
   - Use the `NodeRestriction` admission controller to restrict which node and pod objects the kubelet can access

**6. Enable kubelet certificate rotation:**
   - Enable the `RotateKubeletClientCertificate` feature gate to automatically rotate kubelet client certificates

**7. Restrict network access:**
   - Use firewall rules to limit access to kubelet port 10250 and read-only port 10255
   - Only allow access from authorized components (API server, monitoring systems, etc.)

**8. Keep kubelet updated:**
   - Ensure kubelet version is up-to-date to apply the latest security patches

**9. Protect kernel defaults:**
   - Set `--protect-kernel-defaults=true` to prevent kernel parameters from being overridden

**10. Secure the kubelet config file:**
   - Set file permissions to 644 or more restrictive
   - Ensure ownership is `root:root`

## Verification

To verify kubelet security hardening:

1. **Check anonymous access is disabled:**
   ```bash
   curl -sk https://localhost:10250/pods
   ```
   Should return an authentication error (not a list of pods)

2. **Verify authorization is enforced:**
   - Attempt to access kubelet APIs without proper credentials
   - Should receive authorization denied responses

3. **Confirm read-only ports are closed:**
   ```bash
   curl -sk http://localhost:10255/pods
   ```
   Should fail to connect (connection refused)

4. **Check firewall rules are in place:**
   - Verify that only authorized systems can reach port 10250
   - Use `ss -tlnp | grep 10250` or `netstat` to check listening ports

5. **Inspect running kubelet configuration:**
   ```bash
   curl -sk https://localhost:10250/configz | jq '.kubeletconfig.authentication'
   ```
   Verify that `anonymous: false` is set

## Cleanup

After completing this lesson, optionally restore the kubelet to its original configuration:

```bash
# If changes were made to the config file, restore the original
sudo nano /var/lib/kubelet/config.yaml

# Restart kubelet to apply any reverted changes
systemctl restart kubelet

# Verify the kubelet is running
systemctl status kubelet
```

---

**Key Takeaway**: The kubelet is a critical component that must be properly secured. By disabling anonymous access, enforcing authentication and authorization, restricting permissions, and using firewall rules, you significantly enhance the security posture of your Kubernetes nodes and overall cluster.
