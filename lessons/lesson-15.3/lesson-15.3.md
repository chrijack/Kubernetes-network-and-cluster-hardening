# Lesson 15.3 — Host Firewall Configuration

> From the [Certified Kubernetes Security Specialist (CKS) Video Course](https://www.pearsonitcertification.com/store/certified-kubernetes-security-specialist-cks-video-9780138296476)

## Objective

Configure UFW to ensure all necessary ports for Kubernetes communication, including those required by Cilium CNI, are open while maintaining security.

## Prerequisites

- A Kubernetes cluster or second machine for connection testing
- Administrative access to configure UFW
- netstat utility for port inspection

## Steps

### Step 1: Enable UFW

Enable UFW to start the firewall.

```bash
sudo ufw enable
```

### Step 2: Allow Outbound Communication

Allow all outbound communication from the host.

```bash
sudo ufw default allow outgoing
```

### Step 3: Allow Essential Kubernetes Ports

Allow the following ports required for Kubernetes communication:

**API Server (TCP 6443)**
```bash
sudo ufw allow 6443/tcp
```

**etcd server client API (TCP 2379-2380)**
```bash
sudo ufw allow 2379:2380/tcp
```

**Kubelet API (TCP 10250)**
```bash
sudo ufw allow 10250/tcp
```

**NodePort Services (TCP 30000-32767)**
```bash
sudo ufw allow 30000:32767/tcp
```

### Step 4: Allow Cilium CNI Ports

Allow the following ports for Cilium communication:

**Cilium Hubble Relay (TCP 4245)**
```bash
sudo ufw allow 4245/tcp
```

**Cilium Hubble UI (TCP 12000)**
```bash
sudo ufw allow 12000/tcp
```

**Cilium API (TCP 9091)**
```bash
sudo ufw allow 9091/tcp
```

### Step 5: Review Firewall Rules

List the current UFW rules to confirm the configuration.

```bash
sudo ufw status verbose
```

Example output:
```
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    192.168.1.0/24
80/tcp                     ALLOW IN    Anywhere
443/tcp                    ALLOW IN    Anywhere
6443/tcp                   ALLOW IN    Anywhere
2379:2380/tcp              ALLOW IN    Anywhere
10250/tcp                  ALLOW IN    Anywhere
30000:32767/tcp            ALLOW IN    Anywhere
4245/tcp                   ALLOW IN    Anywhere
12000/tcp                  ALLOW IN    Anywhere
9091/tcp                   ALLOW IN    Anywhere
```

### Step 6: Check Open Ports

Use `netstat` to view open ports and the processes using them.

```bash
netstat -tuln
```

Example output:
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

### Step 7: Test Specific Rules

Attempt connections to verify the firewall rules. For example, try to connect to the Kubernetes API server from a node:

```bash
curl https://<HOST_IP_ADDRESS>:6443 --insecure
```

Ensure connections to other ports are successful as required.

### Step 8: Enable UFW Logging

Enable UFW logging to monitor allowed and denied requests:

```bash
sudo ufw logging on
```

View the logs:

```bash
sudo tail -f /var/log/ufw.log
```

Example output:
```
Jul 26 12:00:00 ubuntu kernel: [UFW ALLOW] IN=eth0 OUT= MAC=00:0c:29:6d:8e:3a SRC=192.168.1.20 DST=192.168.1.10 LEN=60 TOS=0x00 PREC=0x00 TTL=64 ID=54321 DF PROTO=TCP SPT=12345 DPT=6443 WINDOW=65535 RES=0x00 SYN URGP=0
```

## Verification

- Confirm UFW is active and all required ports are listed
- Verify connections to Kubernetes API server succeed
- Confirm firewall logs show allowed traffic for configured ports
- Periodically audit firewall rules to ensure continued security compliance

## Cleanup

To reset UFW to default settings:

```bash
sudo ufw reset
```

To disable UFW:

```bash
sudo ufw disable
```
