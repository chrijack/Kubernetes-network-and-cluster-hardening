# Lesson 10.5 — Kubernetes API Access Verification

> From the [Certified Kubernetes Security Specialist (CKS) Video Course](https://www.pearsonitcertification.com/store/certified-kubernetes-security-specialist-cks-video-9780138296476)

## Objective
Verify the secure configuration of the Kubernetes API server, including checking API server connectivity, ensuring TLS encryption is enabled, confirming anonymous authentication is disabled, and validating API Priority and Fairness (APF) with PriorityLevelConfiguration and FlowSchema.

## Prerequisites
- Kubernetes 1.30 is installed and running
- Administrative access to Kubernetes configuration files
- kubectl command-line tool configured with cluster access
- OpenSSL installed for TLS verification

## Steps

### Step 1: Check API Server Address and Port

Retrieve the API server address and port from your kubeconfig:

```bash
kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}'
```

```bash
kubectl get endpoints -A
```

Expected output:
```
https://192.168.1.100:6443
```

This output shows the API server address (192.168.1.100) and port (6443).

### Step 2: Verify API Server Connectivity

Test connectivity to the API server:

```bash
kubectl get --raw /
```

Expected output:
```json
{
  "paths": [
    "/api",
    "/api/v1",
    "/apis",
    "/apis/",
    ...
  ]
}
```

### Step 3: Test TLS Encryption

Verify TLS 1.2 support:

```bash
openssl s_client -connect 192.168.56.10:6443 -tls1_2
```

Expected output (partial):
```
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-RSA-AES128-GCM-SHA256
```

Verify TLS 1.3 support:

```bash
openssl s_client -connect 192.168.56.10:6443 -tls1_3
```

Expected output (partial):
```
SSL-Session:
    Protocol  : TLSv1.3
    Cipher    : TLS_AES_256_GCM_SHA384
```

These commands confirm that the API server supports TLS 1.2 and 1.3.

### Step 4: Check Anonymous Auth is Forbidden

Attempt to access the API server without authentication:

```bash
curl -k https://192.168.56.10:6443
```

Expected output:
```json
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "forbidden: User \"system:anonymous\" cannot get path \"/\"",
  "reason": "Forbidden",
  "details": {},
  "code": 403
}
```

This output confirms that anonymous authentication is forbidden.

### Step 5: Inspect API Fairness and Priority

Check existing flow schemas:

```bash
kubectl get flowschemas
```

Expected output:
```
NAME                           PRIORITYLEVEL     MATCHINGPRECEDENCE   DISTINGUISHERMETHOD   AGE
exempt                         exempt            1                    <none>                24h
global-default                 global-default    1000                 <none>                24h
service-accounts               service-accounts  200                  ByNamespace           24h
...
```

Check existing priority level configurations:

```bash
kubectl get prioritylevelconfigurations
```

Expected output:
```
NAME              TYPE      ASSUREDCONCURRENCYSHARES   QUEUES   HANDSIZE   QUEUELENGTHLIMIT
exempt            Exempt    0                          0        0          0
global-default    Limited   20                         128      6          50
leader-election   Limited   10                         16       4          50
...
```

Inspect a specific flow schema:

```bash
kubectl get flowschema service-accounts -o yaml
```

Expected output (partial):
```yaml
apiVersion: flowcontrol.apiserver.k8s.io/v1beta2
kind: FlowSchema
metadata:
  name: service-accounts
spec:
  distinguisherMethod:
    type: ByNamespace
  matchingPrecedence: 200
  priorityLevelConfiguration:
    name: service-accounts
  rules:
  - resourceRules:
    - apiGroups:
      - '*'
      clusterScope: true
      namespaces:
      - '*'
      resources:
      - '*'
      verbs:
      - '*'
    subjects:
    - kind: Group
      group:
        name: system:serviceaccounts
```

Inspect a specific priority level configuration:

```bash
kubectl get PriorityLevelConfiguration workload-low -o yaml
```

### Step 6: Create PriorityLevelConfiguration

Create a restrictive priority level configuration for the kubernetes-admin user:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: flowcontrol.apiserver.k8s.io/v1
kind: PriorityLevelConfiguration
metadata:
  name: admin-priority
spec:
  type: Limited
  limited:
    nominalConcurrencyShares: 1  # Reduced to absolute minimum
    limitResponse:
      type: Reject
    lendablePercent: 0  # Prevent lending capacity to other priority levels
    borrowingLimitPercent: 0  # Prevent borrowing capacity from other priority levels
EOF
```

### Step 7: Create FlowSchema

Create a flow schema that routes the kubernetes-admin user to the admin priority level:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: flowcontrol.apiserver.k8s.io/v1
kind: FlowSchema
metadata:
  name: admin-flow
spec:
  priorityLevelConfiguration:
    name: admin-priority
  matchingPrecedence: 100
  rules:
  - resourceRules:
    - apiGroups: ["*"]
      resources: ["*"]
      clusterScope: true
      verbs: ["*"]
    subjects:
    - kind: User
      user:
        name: "kubernetes-admin"
EOF
```

### Step 8: Create a Test Script

Create a bash script that sends a large number of concurrent requests to test the APF configuration:

```bash
cat <<EOF > test-apf.sh
#!/bin/bash

# Function to send requests
send_requests() {
  for i in {1..100}
  do
    kubectl --user=kubernetes-admin get pods --all-namespaces >/dev/null 2>&1 &
  done
}

# Send requests in 10 batches
for batch in {1..10}
do
  send_requests
  sleep 0.1  # Short pause between batches
done

wait
EOF
```

Make the script executable:

```bash
chmod +x test-apf.sh
```

### Step 9: Monitor APF Metrics

In a separate terminal, run this command to monitor APF metrics in real-time:

```bash
watch -n 0.5 'kubectl get --raw /metrics | grep "apiserver_flowcontrol" | grep "flow_schema=\"admin-flow\"" | sort | head -n 10'
```

### Step 10: Run the Test

Execute the test script:

```bash
./test-apf.sh
```

Observe the metrics in the monitoring terminal. You should see:
1. Requests being rejected due to concurrency limits
2. Increased execution times for some requests
3. Fair distribution of resources within the priority level

### Step 11: Analyze the Results

After the test completes, check for rejected requests:

```bash
kubectl get --raw /metrics | grep 'apiserver_flowcontrol_rejected_requests_total{flow_schema="admin-flow"}'
```

Examine the request execution times:

```bash
kubectl get --raw /metrics | grep 'apiserver_flowcontrol_request_execution_seconds_bucket{flow_schema="admin-flow"}'
```

## Verification

The demonstration verifies secure configuration of the Kubernetes API server by:

1. **API Connectivity**: Confirming the API server is accessible at the expected address and port
2. **TLS Encryption**: Validating that TLS 1.2 and 1.3 are supported for secure communication
3. **Anonymous Auth**: Confirming that unauthenticated requests are properly rejected
4. **API Priority and Fairness**: Demonstrating that:
   - Concurrent requests for the kubernetes-admin user are limited
   - Excess requests are rejected when limits are reached
   - Resources are fairly distributed within priority levels
   - The API server is protected from being overwhelmed by a single user or client

## Cleanup

To remove the custom APF configuration and test script:

```bash
kubectl delete flowschema admin-flow
kubectl delete prioritylevelconfiguration admin-priority
rm -f test-apf.sh admin-priority.yaml admin-flow.yaml
```
