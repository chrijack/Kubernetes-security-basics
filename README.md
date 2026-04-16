# Kubernetes Security Basics - Demo Scripts

**Course:** Kubernetes Security Basics (Video Course)  
**Author:** Chris Jackson, CCIEx2 #6256 — Distinguished Architect, Cisco  
**Publisher:** Pearson IT Certification  
**Published:** April 4, 2025  
**ISBN-10:** 0-13-544619-8  
**ISBN-13:** 978-0-13-544619-5  

This repository contains demo scripts and lab exercises from the **Kubernetes Security Basics** video course. The course covers foundational Kubernetes security concepts, securing clusters, implementing robust access controls, and verifying platform integrity. It is part of the larger **Kubernetes Security Essentials (Video Collection)**.

For the complete CKS (Certified Kubernetes Security Specialist) course materials, see the [CKS Demo Scripts repository](https://github.com/chrijack/CKS-demo-scripts).

---

## Course Content

All lesson materials are located in the `lessons/` directory.

### Lesson 1: Building Your K8s Home Lab

| Lesson | Topic | Link |
|--------|-------|------|
| 1.2 | Setting Up kops on Google Cloud Engine | [lessons/lesson-1.2/lesson-1.2.md](lessons/lesson-1.2/lesson-1.2.md) |
| 1.3 | Kubernetes Cluster Setup with kops on GCE | [lessons/lesson-1.3/lesson-1.3.md](lessons/lesson-1.3/lesson-1.3.md) |
| 1.4 | Getting Minikube Up and Running with Cilium | [lessons/lesson-1.4/lesson-1.4.md](lessons/lesson-1.4/lesson-1.4.md) |

### Lesson 2: Kubernetes Security Foundation

_No demo scripts — conceptual lesson only._

### Lesson 3: CIS Benchmark Review of Kubernetes Components

| Lesson | Topic | Link |
|--------|-------|------|
| 3.2 | Installing and Running kube-bench for CIS Kubernetes Benchmark Security Scanning | [lessons/lesson-3.2/lesson-3.2.md](lessons/lesson-3.2/lesson-3.2.md) |

### Lesson 4: Role Based Access Control (RBAC)

| Lesson | Topic | Link |
|--------|-------|------|
| 4.2 | Creating a User Account in Kubernetes | [lessons/lesson-4.2/lesson-4.2.md](lessons/lesson-4.2/lesson-4.2.md) |
| 4.3 | Configuring Developer Permissions with Kubernetes RBAC | [lessons/lesson-4.3/lesson-4.3.md](lessons/lesson-4.3/lesson-4.3.md) |
| 4.4 | ClusterRoles, Role Aggregation, and Privilege Validation | [lessons/lesson-4.4/lesson-4.4.md](lessons/lesson-4.4/lesson-4.4.md) |
| 4.5 | Inspecting, Validating, and Troubleshooting RBAC Privileges | [lessons/lesson-4.5/lesson-4.5.md](lessons/lesson-4.5/lesson-4.5.md) |

### Lesson 5: Protecting Service Accounts

| Lesson | Topic | Link |
|--------|-------|------|
| 5.2 | Configuring a Service Account in Kubernetes 1.30 | [lessons/lesson-5.2/lesson-5.2.md](lessons/lesson-5.2/lesson-5.2.md) |
| 5.3 | Disabling Default Settings for Service Accounts | [lessons/lesson-5.3/lesson-5.3.md](lessons/lesson-5.3/lesson-5.3.md) |
| 5.4 | Verifying Service Account Security and Ensuring Compliance | [lessons/lesson-5.4/lesson-5.4.md](lessons/lesson-5.4/lesson-5.4.md) |

### Lesson 6: Verify Platform Binaries Before Deploying

| Lesson | Topic | Link |
|--------|-------|------|
| 6.2 | Verifying Kubernetes Binary Integrity | [lessons/lesson-6.2/lesson-6.2.md](lessons/lesson-6.2/lesson-6.2.md) |

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
│   ├── lesson-1.4/
│   ├── lesson-3.2/
│   ├── lesson-4.2/
│   ├── lesson-4.3/
│   ├── lesson-4.4/
│   ├── lesson-4.5/
│   ├── lesson-5.2/
│   ├── lesson-5.3/
│   ├── lesson-5.4/
│   └── lesson-6.2/
```

---

## Course Information

**Course URL:** https://www.pearsonitcertification.com/store/kubernetes-security-basics-video-course-9780135446195

**Description:** This course covers foundational Kubernetes security concepts, securing clusters, implementing robust access controls, and verifying platform integrity. Part of the Kubernetes Security Essentials (Video Collection).

---

## License

MIT License

Copyright (c) 2025 Chris Jackson

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

**Attribution Required:** Demo scripts from the [Kubernetes Security Basics (Video Course)](https://www.pearsonitcertification.com/store/kubernetes-security-basics-video-course-9780135446195) by Chris Jackson, published by Pearson IT Certification (ISBN: 978-0-13-544619-5).

---

## Related Resources

- [CKS Demo Scripts Repository](https://github.com/chrijack/CKS-demo-scripts) - Complete Certified Kubernetes Security Specialist course materials
- [Kubernetes Security Essentials (Video Collection)](https://www.pearsonitcertification.com) - Full video collection by Pearson IT Certification
