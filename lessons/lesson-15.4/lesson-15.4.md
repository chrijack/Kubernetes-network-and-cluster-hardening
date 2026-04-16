# Lesson 15.4 — Pod Security Verification and Validation

> From the [Certified Kubernetes Security Specialist (CKS) Video Course](https://www.pearsonitcertification.com/store/certified-kubernetes-security-specialist-cks-video-9780138296476)

## Objective

Configure and validate UFW (Uncomplicated Firewall) to ensure all necessary ports for Kubernetes communication are open, firewall is active with proper logging, and unauthorized access is blocked.

## Prerequisites

- A Kubernetes cluster with Cilium CNI installed
- Root or sudo access on the target node
- A second machine or VM for testing connections
- Basic understanding of UFW and network connectivity

## Steps

### Step 1: Enable UFW and Configure Ports

Enable UFW:
```bash
sudo ufw enable
```

Allow outbound communication:
```bash
sudo ufw default allow outgoing
```

Allow Kubernetes and Cilium ports with logging:
```bash
sudo ufw allow log 6443/tcp && sudo ufw allow log 2379:2380/tcp && sudo ufw allow log 10250/tcp && sudo ufw allow log 30000:32767/tcp && sudo ufw allow log 4245/tcp && sudo ufw allow log 12000/tcp && sudo ufw allow log 9091/tcp
```

### Step 2: Validate Firewall Status and Rules

Check UFW status and configured rules:
```bash
sudo ufw status verbose
```

Expected output:
```
Status: active
Logging: on (low)
Default: allow (outgoing), deny (incoming), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
6443/tcp                   ALLOW IN    Anywhere
2379:2380/tcp              ALLOW IN    Anywhere
10250/tcp                  ALLOW IN    Anywhere
30000:32767/tcp            ALLOW IN    Anywhere
4245/tcp                   ALLOW IN    Anywhere
12000/tcp                  ALLOW IN    Anywhere
9091/tcp                   ALLOW IN    Anywhere
```

### Step 3: Enable Logging

Enable UFW logging:
```bash
sudo ufw logging on
```

Individual logging rules for Kubernetes API Server (TCP 6443):
```bash
sudo ufw allow log 6443/tcp
```

etcd server client API (TCP 2379-2380):
```bash
sudo ufw allow log 2379:2380/tcp
```

Kubelet API (TCP 10250):
```bash
sudo ufw allow log 10250/tcp
```

NodePort Services (TCP 30000-32767):
```bash
sudo ufw allow log 30000:32767/tcp
```

Cilium Hubble Relay (TCP 4245):
```bash
sudo ufw allow log 4245/tcp
```

Cilium Hubble UI (TCP 12000):
```bash
sudo ufw allow log 12000/tcp
```

Cilium API (TCP 9091):
```bash
sudo ufw allow log 9091/tcp
```

Restart UFW to apply all rules:
```bash
sudo ufw disable
sudo ufw enable
```

### Step 4: Verify Open Ports

Check which ports are listening on the node:
```bash
netstat -tuln
```

Expected output:
```
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
tcp6       0      0 :::6443                 :::*                    LISTEN
tcp6       0      0 :::2379                 :::*                    LISTEN
tcp6       0      0 :::2380                 :::*                    LISTEN
tcp6       0      0 :::10250                :::*                    LISTEN
tcp6       0      0 :::4245                 :::*                    LISTEN
tcp6       0      0 :::12000                :::*                    LISTEN
tcp6       0      0 :::9091                 :::*                    LISTEN
```

### Step 5: Test Firewall Rules

From a second machine, attempt to connect to allowed ports.

Test Kubernetes API Server:
```bash
curl -k https://192.168.56.10:6443
```

Test Cilium Hubble Relay:
```bash
curl http://192.168.56.10:4245
```

Block a specific port to simulate unauthorized access:
```bash
sudo ufw deny log 9999/tcp
```

Attempt to connect to the blocked port:
```bash
curl http://192.168.56.10:9999
```

Expected output:
```
curl: (7) Failed to connect to 192.168.10.56 port 9999: Connection refused
```

### Step 6: Check Logs for Connection Attempts

View UFW logs to verify logging is functioning:
```bash
sudo tail -f /var/log/ufw.log
```

Expected output:
```
Jul 26 12:00:00 ubuntu kernel: [UFW ALLOW] IN=eth0 OUT= MAC=00:0c:29:6d:8e:3a SRC=192.168.1.20 DST=192.168.10.56 LEN=60 TOS=0x00 PREC=0x00 TTL=64 ID=54321 DF PROTO=TCP SPT=12345 DPT=6443 WINDOW=65535 RES=0x00 SYN URGP=0
Jul 26 12:01:00 ubuntu kernel: [UFW BLOCK] IN=eth0 OUT= MAC=00:0c:29:6d:8e:3a SRC=192.168.1.20 DST=192.168.10.56 LEN=60 TOS=0x00 PREC=0x00 TTL=64 ID=54322 DF PROTO=TCP SPT=12345 DPT=9999 WINDOW=65535 RES=0x00 SYN URGP=0
```

## Verification

Confirm the following:

- UFW is active and shows all configured rules with `sudo ufw status verbose`
- All required Kubernetes and Cilium ports are in the LISTEN state via `netstat -tuln`
- Allowed ports accept connections from external machines
- Blocked ports refuse connections with "Connection refused" error
- UFW logs contain both ALLOW and BLOCK entries for tested connections

## Cleanup

To reset the firewall (if needed):
```bash
sudo ufw disable
sudo ufw reset
```

To remove a specific rule:
```bash
sudo ufw delete deny log 9999/tcp
```
