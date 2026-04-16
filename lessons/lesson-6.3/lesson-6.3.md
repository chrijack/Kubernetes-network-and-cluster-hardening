# Lesson 6.3 — TLS Encryption for Kubernetes Ingress with Self-Signed Certificates

> From the [Certified Kubernetes Security Specialist (CKS) Video Course](https://www.pearsonitcertification.com/store/certified-kubernetes-security-specialist-cks-video-9780138296476)

## Objective
Secure Kubernetes Ingress traffic with TLS encryption using self-signed certificates in a Minikube cluster. This lesson covers two approaches: using Kubernetes CertificateSigningRequest (CSR) and creating self-signed certificates directly with OpenSSL.

## Prerequisites
- Minikube running with the Ingress addon enabled
- kubectl configured to communicate with your Minikube cluster
- openssl installed on your local machine

---

## Approach A: Kubernetes CertificateSigningRequest (CSR)

This approach leverages Kubernetes' built-in CertificateSigningRequest API to generate and manage certificates through the cluster.

### Step 1: Generate a private key and certificate signing request

```bash
openssl genrsa -out myapp.key 2048
openssl req -new -key myapp.key -out myapp.csr -subj "/CN=myapp.com" -addext "subjectAltName=DNS:myapp.com"
```

### Step 2: Create a CertificateSigningRequest in Kubernetes

Save the following manifest as `myapp-csr.yaml`:

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: myapp-csr
spec:
  request: $(cat myapp.csr | base64 | tr -d '\n')
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - digital signature
  - key encipherment
  - server auth
```

Apply the CSR:

```bash
kubectl apply -f myapp-csr.yaml
```

### Step 3: Approve the certificate signing request

```bash
kubectl certificate approve myapp-csr
```

### Step 4: Extract the signed certificate

```bash
kubectl get csr myapp-csr -o jsonpath='{.status.certificate}' | base64 -d > myapp.crt
```

### Step 5: Create a TLS Secret

```bash
kubectl create secret tls myapp-tls --cert=myapp.crt --key=myapp.key
```

---

## Approach B: Self-Signed Certificate with OpenSSL

This is the recommended approach for testing and development. It uses OpenSSL to directly generate a self-signed certificate without relying on Kubernetes CSR.

### Step 1: Generate a self-signed certificate

Use OpenSSL to create a self-signed certificate for your Ingress host:

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=myapp.com"
```

This command generates a private key (`tls.key`) and a self-signed certificate (`tls.crt`) valid for 365 days, with the common name (CN) set to `myapp.com`.

### Step 2: Create a TLS Secret

Create a Kubernetes Secret to store the TLS certificate and key:

```bash
kubectl create secret tls myapp-tls --key tls.key --cert tls.crt
```

This creates a Secret named `myapp-tls` using the generated key and certificate files.

### Step 3: Configure the Ingress to use TLS

Update your Ingress resource to use the TLS Secret and enable HTTPS. Save the following as `ingress-tls.yaml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress-tls
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - myapp.com
    secretName: myapp-tls
  rules:
  - host: myapp.com
    http:
      paths:
      - path: /app1
        pathType: Prefix
        backend:
          service:
            name: app1-svc
            port:
              number: 80
      - path: /app2
        pathType: Prefix
        backend:
          service:
            name: app2-svc
            port:
              number: 80
```

The `tls` section specifies the host to secure with TLS and references the Secret containing the certificate.

Apply the updated Ingress manifest:

```bash
kubectl apply -f ingress-tls.yaml
```

### Step 4: Test the TLS encryption

Get the Minikube IP:

```bash
minikube ip
```

Update your `/etc/hosts` file to map the hostname to the Minikube IP (replace `MINIKUBE_IP` with the actual IP):

```
MINIKUBE_IP myapp.com
```

Test the HTTPS connection using curl with the `--insecure` flag (required for self-signed certificates):

```bash
curl --insecure -H "Host: myapp.com" https://127.0.0.1/app1
curl --insecure -H "Host: myapp.com" https://127.0.0.1/app2
curl --insecure -v -H "Host: myapp.com" https://127.0.0.1/app2
```

The `--insecure` flag tells curl to ignore certificate validation errors. You can also open `https://myapp.com/app1` and `https://myapp.com/app2` in your web browser. You'll see a warning about the self-signed certificate, but you can proceed to the site.

---

## Verification

### Step 1: Verify the TLS configuration

Confirm that TLS is configured correctly for the Ingress by describing the resource:

```bash
kubectl describe ingress example-ingress
```

In the 'TLS' section, you should see the Secret name and the host it applies to.

### Step 2: Check the Ingress controller logs

Check the Ingress controller logs for any TLS-related errors:

```bash
kubectl logs -n ingress-nginx <ingress-controller-pod-name>
```

If everything is set up correctly, you should be able to access your applications over HTTPS using the specified host, with the traffic encrypted using the provided TLS certificate.

---

## Cleanup

To remove the resources created in this lesson:

```bash
kubectl delete ingress example-ingress-tls
kubectl delete secret myapp-tls
```

If using Approach A, also clean up the CSR:

```bash
kubectl delete csr myapp-csr
```

---

## Notes

- Using self-signed certificates is suitable for testing and development purposes.
- For production environments, always use certificates from a trusted Certificate Authority (CA) and ensure proper validation and renewal.
