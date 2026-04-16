# Kubernetes Network and Cluster Hardening - Demo Scripts

**Course:** Kubernetes Network and Cluster Hardening (Video Course)  
**Author:** Chris Jackson, CCIEx2 #6256 — Distinguished Architect, Cisco  
**Publisher:** Pearson IT Certification  
**Published:** April 4, 2025  
**ISBN-10:** 0-13-544637-6  
**ISBN-13:** 978-0-13-544637-9  

This repository contains demo scripts and lab exercises from the **Kubernetes Network and Cluster Hardening** video course. The course provides essential skills for securing Kubernetes environments at the network and cluster level — from setting up effective network policies and securing ingress traffic with TLS to safeguarding node metadata and restricting API access. It is part of the larger **Kubernetes Security Essentials (Video Collection)**.

For the complete CKS (Certified Kubernetes Security Specialist) course materials, see the [CKS Demo Scripts repository](https://github.com/chrijack/CKS-demo-scripts).

---

## Course Content

All lesson materials are located in the `lessons/` directory.

### Lesson 1: Network Security Policies

| Lesson | Topic | Link |
|--------|-------|------|
| 1.2 | Applying Kubernetes Network Policies with Cilium | [lessons/lesson-1.2/lesson-1.2.md](lessons/lesson-1.2/lesson-1.2.md) |
| 1.3 | Kubernetes Network Policies and Cilium | [lessons/lesson-1.3/lesson-1.3.md](lessons/lesson-1.3/lesson-1.3.md) |

### Lesson 2: Properly Setup Ingress with TLS

| Lesson | Topic | Link |
|--------|-------|------|
| 2.2 | Kubernetes Ingress with Cilium and Hubble Observability | [lessons/lesson-2.2/lesson-2.2.md](lessons/lesson-2.2/lesson-2.2.md) |
| 2.3 | TLS Encryption for Kubernetes Ingress with Self-Signed Certificates | [lessons/lesson-2.3/lesson-2.3.md](lessons/lesson-2.3/lesson-2.3.md) |
| 2.4 | Validating and Troubleshooting TLS Ingress Configuration | [lessons/lesson-2.4/lesson-2.4.md](lessons/lesson-2.4/lesson-2.4.md) |

### Lesson 3: Protect Node Metadata and Endpoints

| Lesson | Topic | Link |
|--------|-------|------|
| 3.1 | Securing the Kubelet in Kubernetes | [lessons/lesson-3.1/lesson-3.1.md](lessons/lesson-3.1/lesson-3.1.md) |
| 3.4 | Kubelet Security Verification and Hardening | [lessons/lesson-3.4/lesson-3.4.md](lessons/lesson-3.4/lesson-3.4.md) |

### Lesson 4: Kubernetes Dashboard Security

| Lesson | Topic | Link |
|--------|-------|------|
| 4.2 | Installing Metrics Server and Kubernetes Dashboard with RBAC | [lessons/lesson-4.2/lesson-4.2.md](lessons/lesson-4.2/lesson-4.2.md) |

### Lesson 5: Restrict Access to Kubernetes API

| Lesson | Topic | Link |
|--------|-------|------|
| 5.3 | Restricting Access to Kubernetes API | [lessons/lesson-5.3/lesson-5.3.md](lessons/lesson-5.3/lesson-5.3.md) |
| 5.4 | Hardening the Kubernetes API Server | [lessons/lesson-5.4/lesson-5.4.md](lessons/lesson-5.4/lesson-5.4.md) |
| 5.5 | Kubernetes API Access Verification | [lessons/lesson-5.5/lesson-5.5.md](lessons/lesson-5.5/lesson-5.5.md) |

### Lesson 6: Minimize External Access to the Network

| Lesson | Topic | Link |
|--------|-------|------|
| 6.2 | Finding Open Ports | [lessons/lesson-6.2/lesson-6.2.md](lessons/lesson-6.2/lesson-6.2.md) |
| 6.3 | Host Firewall Configuration | [lessons/lesson-6.3/lesson-6.3.md](lessons/lesson-6.3/lesson-6.3.md) |
| 6.4 | Pod Security Verification and Validation | [lessons/lesson-6.4/lesson-6.4.md](lessons/lesson-6.4/lesson-6.4.md) |

### Lesson 7: Verify Platform Binaries Before Deploying

| Lesson | Topic | Link |
|--------|-------|------|
| 7.2 | Verifying Kubernetes Binary Integrity | [lessons/lesson-7.2/lesson-7.2.md](lessons/lesson-7.2/lesson-7.2.md) |

---

## Prerequisites

- Basic understanding of Kubernetes concepts and architecture
- Kubernetes cluster access (local or cloud-based)
- kubectl command-line tool
- Docker or container runtime knowledge
- Linux/Unix command-line familiarity

---

## Repository Structure

```
.
├── README.md
├── .gitignore
├── lessons/
│   ├── lesson-1.2/
│   ├── lesson-1.3/
│   ├── lesson-2.2/
│   ├── lesson-2.3/
│   ├── lesson-2.4/
│   ├── lesson-3.1/
│   ├── lesson-3.4/
│   ├── lesson-4.2/
│   ├── lesson-5.3/
│   ├── lesson-5.4/
│   ├── lesson-5.5/
│   ├── lesson-6.2/
│   ├── lesson-6.3/
│   ├── lesson-6.4/
│   └── lesson-7.2/
```

---

## Course Information

**Course URL:** https://www.pearsonitcertification.com/store/kubernetes-network-and-cluster-hardening-video-course-9780135446379

**Description:** This course provides essential skills for securing Kubernetes environments at the network and cluster level — from setting up effective network policies and securing ingress traffic with TLS to safeguarding node metadata and restricting API access. Part of the Kubernetes Security Essentials (Video Collection).

---

## License

MIT License

Copyright (c) 2025 Chris Jackson

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

**Attribution Required:** Demo scripts from the [Kubernetes Network and Cluster Hardening (Video Course)](https://www.pearsonitcertification.com/store/kubernetes-network-and-cluster-hardening-video-course-9780135446379) by Chris Jackson, published by Pearson IT Certification (ISBN: 978-0-13-544637-9).

---

## Related Resources

- [CKS Demo Scripts Repository](https://github.com/chrijack/CKS-demo-scripts) - Complete Certified Kubernetes Security Specialist course materials
- [Kubernetes Security Essentials (Video Collection)](https://www.pearsonitcertification.com) - Full video collection by Pearson IT Certification
