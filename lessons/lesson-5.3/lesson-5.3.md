# Lesson 5.3 — Restricting Access to Kubernetes API

> From the [Certified Kubernetes Security Specialist (CKS) Video Course](https://www.pearsonitcertification.com/store/certified-kubernetes-security-specialist-cks-video-9780138296476)

## Objective

Demonstrate the configuration of Kubernetes API server settings to ensure secure access using version 1.30 configuration files and connecting to a Vagrant box named `control-plane`.

## Prerequisites

- Kubernetes 1.30 is installed and running on your system
- Administrative access to Kubernetes configuration files and permissions to restart the API server
- Vagrant is installed and a Vagrant box named `control-plane` is set up

## Steps

### Step 1: Connect to the Vagrant Box

**Command**:
```bash
vagrant ssh control-plane
```

This command connects to the Vagrant box named `control-plane`.

---

### Step 2: Secure the API Server Port

**Configuration File**: `/etc/kubernetes/manifests/kube-apiserver.yaml`

Ensure the following line is included in the `spec.containers.command` section:

```yaml
- --secure-port=6443
```

**Explanation**:
- `--secure-port=6443`: Specifies the secure HTTPS port for the API server.
- The deprecated `--insecure-port` flag has been removed in 1.24+. Only use the secure port.

---

### Step 3: Enable TLS Encryption

**Configuration File**: `/etc/kubernetes/manifests/kube-apiserver.yaml`

Ensure the following lines are included in the `spec.containers.command` section:

```yaml
- --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
- --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
```

**Explanation**:
- `--tls-cert-file`: Specifies the file containing the API server's certificate.
- `--tls-private-key-file`: Specifies the file containing the API server's private key.

---

### Step 4: Bind API Server to Secure IP

**Configuration File**: `/etc/kubernetes/manifests/kube-apiserver.yaml`

Ensure the following line is included in the `spec.containers.command` section:

```yaml
- --bind-address=10.0.0.1
```

**Explanation**:
- `--bind-address=10.0.0.1`: Binds the API server to a specific secure IP address to restrict access.

---

### Step 5: Set Advertise Address for Clients

**Configuration File**: `/etc/kubernetes/manifests/kube-apiserver.yaml`

Ensure the following line is included in the `spec.containers.command` section:

```yaml
- --advertise-address=k8s-api.example.com
```

**Explanation**:
- `--advertise-address=k8s-api.example.com`: Sets the IP address or DNS name that the API server advertises to clients.

---

### Step 6: Configure Authorization Mode

**Configuration File**: `/etc/kubernetes/manifests/kube-apiserver.yaml`

Ensure the following line is included in the `spec.containers.command` section:

```yaml
- --authorization-mode=Node,RBAC
```

**Explanation**:
- `--authorization-mode=Node,RBAC`: Uses Node and RBAC for authorization, which is a best practice for security.

---

### Step 7: Secure Etcd Communication

**Configuration File**: `/etc/kubernetes/manifests/kube-apiserver.yaml`

Ensure the following lines are included in the `spec.containers.command` section:

```yaml
- --etcd-certfile=/etc/kubernetes/pki/etcd.crt
- --etcd-keyfile=/etc/kubernetes/pki/etcd.key
- --etcd-cafile=/etc/kubernetes/pki/ca.crt
```

**Explanation**:
- `--etcd-certfile`: Specifies the certificate file for etcd.
- `--etcd-keyfile`: Specifies the key file for etcd.
- `--etcd-cafile`: Specifies the CA file for etcd.

---

### Additional Security Recommendations

- Enable audit logging by setting `--audit-log-path` and `--audit-policy-file` flags.
- Use secure TLS certificates for the API server by setting `--tls-cert-file` and `--tls-private-key-file` flags.
- Regularly rotate and update TLS certificates and keys.
- Enable encryption at rest for secrets using the `--encryption-provider-config` flag.

## Verification

Use the following commands to verify each configuration setting:

**Check Secure Port**:
```bash
netstat -tnlp | grep 6443
```

**Verify TLS Encryption**:
```bash
openssl s_client -connect <api-server-ip>:6443
```

**Verify Bind Address**:
```bash
kubectl describe pod kube-apiserver -n kube-system | grep 'Node:'
```

**Check Advertise Address**:
```bash
kubectl get endpoints -o wide | grep k8s-api.example.com
```

**Verify Etcd Communication**:
```bash
etcdctl --endpoints=https://<etcd-endpoint>:2379 --cert=/etc/kubernetes/pki/etcd.crt --key=/etc/kubernetes/pki/etcd.key --cacert=/etc/kubernetes/pki/ca.crt endpoint health
```

## Cleanup

After completing the demonstration, ensure the kube-apiserver pod is running with the updated configuration. Monitor the pod status and verify all API server endpoints are responding correctly.
