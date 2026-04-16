# Lesson 10.4 — Hardening the Kubernetes API Server

> From the [Certified Kubernetes Security Specialist (CKS) Video Course](https://www.pearsonitcertification.com/store/certified-kubernetes-security-specialist-cks-video-9780138296476)

## Objective

Demonstrate the configuration of Kubernetes API server settings to ensure secure access and prevent attacks using Kubernetes 1.30. This lesson covers TLS 1.3 enforcement, secure cipher suites, rate limiting, and API Priority and Fairness (APF).

## Prerequisites

- Kubernetes 1.30 is installed and running
- Administrative access to Kubernetes configuration files and permissions to restart the API server
- Vagrant installed with a Vagrant box named `control-plane` set up
- `openssl`, `nmap`, and `kubectl` tools available
- Access to `/etc/kubernetes/manifests/` directory

## Steps

### Step 1: Connect to the Vagrant Box

```bash
vagrant ssh control-plane
```

This connects to the Vagrant box named `control-plane` where the Kubernetes API server is running.

### Step 2: Configure TLS 1.3 and Secure Cipher Suites

**Edit the API server manifest:**

```bash
sudo nano /etc/kubernetes/manifests/kube-apiserver.yaml
```

**Add or modify the following flags in the command section:**

```yaml
spec:
  containers:
  - command:
    - kube-apiserver
    - --tls-min-version=VersionTLS13
    - --tls-cipher-suites=TLS_AES_128_GCM_SHA256,TLS_AES_256_GCM_SHA384,TLS_CHACHA20_POLY1305_SHA256
```

Save the file and exit. The API server will automatically restart.

**Verify the process is running:**

```bash
ps aux | grep kube-apiserver
```

### Step 3: Implement Rate Limiting

**Edit the API server manifest:**

```bash
sudo nano /etc/kubernetes/manifests/kube-apiserver.yaml
```

**Ensure the following lines are included in the `spec.containers.command` section:**

```yaml
- --max-requests-inflight=400
- --max-mutating-requests-inflight=200
```

**Explanation:**
- `--max-requests-inflight=400`: Limits the number of non-mutating requests the server can handle simultaneously.
- `--max-mutating-requests-inflight=200`: Limits the number of mutating requests the server can handle simultaneously.

Save the file and exit.

### Step 4: Enable API Priority and Fairness

**Edit the API server manifest:**

```bash
sudo nano /etc/kubernetes/manifests/kube-apiserver.yaml
```

**Ensure the following line is included in the `spec.containers.command` section:**

```yaml
- --enable-priority-and-fairness=true
```

**Explanation:**
- `--enable-priority-and-fairness=true`: Enables API Priority and Fairness to provide more granular control over API requests, preventing high-priority system components from being starved by low-priority user requests.

Save the file and exit.

### Step 5: Secure etcd Communication

**Edit the etcd manifest:**

```bash
sudo nano /etc/kubernetes/manifests/etcd.yaml
```

**Add to the container command:**

```yaml
- --tls-min-version=TLS1.3
```

Save the file and exit.

### Step 6: Update kubelet Configuration

**Add TLS 1.3 requirement to kubelet config:**

```bash
echo "tlsMinVersion: VersionTLS13" | sudo tee -a /var/lib/kubelet/config.yaml
sudo systemctl restart kubelet
```

## Verification

### Verify TLS 1.3 Configuration

**Test successful TLS 1.3 connection:**

```bash
openssl s_client -connect 192.168.56.10:6443 -tls1_3 -servername kube-apiserver
```

You should see output indicating a successful TLS 1.3 connection.

**Test that older TLS versions are rejected:**

```bash
# TLS 1.2 (should fail)
openssl s_client -connect 192.168.56.10:6443 -tls1_2 -servername kube-apiserver

# TLS 1.1 (should fail)
openssl s_client -connect 192.168.56.10:6443 -tls1_1 -servername kube-apiserver

# TLS 1.0 (should fail)
openssl s_client -connect 192.168.56.10:6443 -tls1 -servername kube-apiserver
```

These should all fail, confirming that only TLS 1.3 is accepted.

### Verify Cipher Suites

**Enumerate configured cipher suites:**

```bash
nmap --script ssl-enum-ciphers -p 6443 192.168.56.10
```

This should only list the specified TLS 1.3 cipher suites:
- TLS_AES_128_GCM_SHA256
- TLS_AES_256_GCM_SHA384
- TLS_CHACHA20_POLY1305_SHA256

### Verify API Server Functionality

**Check that the API server is functioning:**

```bash
kubectl get nodes
```

**Check the API server logs:**

```bash
sudo journalctl -u kubelet | grep apiserver
```

### Verify Rate Limiting

**Check rate limiting metrics:**

```bash
kubectl get --raw /metrics | grep apiserver_flowcontrol_request_concurrency_limit
```

### Verify API Priority and Fairness

**View Priority Level configurations:**

```bash
kubectl get flowcontrolprioritylevelconfigurations
```

**View Flow Schema configurations:**

```bash
kubectl get flowcontrolflowschemas
```

### Verify kubelet TLS Configuration

**Test kubelet with TLS 1.3:**

```bash
openssl s_client -connect 192.168.56.10:10250 -tls1_3 -servername kubelet
```

### Verify etcd TLS Configuration

**Test etcd with TLS 1.3:**

```bash
openssl s_client -connect 192.168.56.10:2379 -tls1_3 -servername etcd
```

## Cleanup

To revert to default settings (if needed for testing):

```bash
# Restore API server defaults (remove TLS 1.3 enforcement)
sudo nano /etc/kubernetes/manifests/kube-apiserver.yaml
# Remove or comment out the TLS and rate limiting flags

# Restore kubelet defaults
sudo nano /var/lib/kubelet/config.yaml
# Remove the tlsMinVersion line

# Restart services
sudo systemctl restart kubelet
```

## Summary

This lesson demonstrates hardening the Kubernetes API server through:

1. **TLS 1.3 Enforcement**: Only allows the most secure TLS version
2. **Cipher Suite Configuration**: Restricts to modern, secure cipher suites
3. **Rate Limiting**: Protects against request flooding via inflight request limits
4. **API Priority and Fairness**: Ensures critical system requests are not starved by user requests
5. **Kubelet & etcd Hardening**: Applies the same security standards to related components

These settings work together to create a secure, resilient API server that resists both attacks and resource exhaustion scenarios.
