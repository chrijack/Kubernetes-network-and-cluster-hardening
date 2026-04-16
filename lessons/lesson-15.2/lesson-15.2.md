# Lesson 15.2 — Finding Open Ports

> From the [Certified Kubernetes Security Specialist (CKS) Video Course](https://www.pearsonitcertification.com/store/certified-kubernetes-security-specialist-cks-video-9780138296476)

## Objective

Identify open ports and running services to understand the current exposure of your system.

## Prerequisites

- Access to a Linux system with administrative privileges
- Tools available: `ss`, `lsof`, and `iptables`/`ip6tables`

## Steps

### Step 1: List Open Ports and Associated Services

Run the following command to view all open ports and their associated services:

```bash
sudo ss -tulpan
```

This displays TCP and UDP sockets in listening mode with the program names and PIDs.

### Step 2: Find Services and User Accounts with Open Ports

Use this command to identify services running on open ports along with the user accounts they belong to:

```bash
sudo lsof -i -P -n | grep LISTEN
```

This shows listening sockets with the associated processes and user accounts.

### Step 3: View iptables Rules with Line Numbers

To examine the current firewall rules, run:

```bash
sudo iptables -t filter -L --line-numbers
sudo ip6tables -t filter -L --line-numbers
```

The first command displays IPv4 firewall rules and the second shows IPv6 rules, both with line numbers for easy reference.

## Verification

Walk through each command output and review:

- Which ports are open on your system
- Which services are listening on those ports
- Which user accounts own those services
- What firewall rules (iptables) are in place for those ports

This provides visibility into your system's current security posture and network exposure.

## Cleanup

No cleanup required. This lesson is informational and does not modify system configuration.
