# Lesson 3.2 — Installing and Running kube-bench for CIS Kubernetes Benchmark Security Scanning

> From the [Certified Kubernetes Security Specialist (CKS) Video Course](https://www.pearsonitcertification.com/store/certified-kubernetes-security-specialist-cks-video-9780138296476)

## Objective

Install kube-bench on a Kubernetes node and understand how to perform CIS Kubernetes Benchmark security compliance checks. kube-bench is an automated tool that scans your Kubernetes cluster and verifies whether it is deployed according to security best practices defined in the CIS Kubernetes Benchmark.

## Prerequisites

- A running Vagrant VM with Kubernetes already initialized
- Network access to download the kube-bench release from GitHub
- `sudo` privileges to install packages

## Steps

### Step 1: Start Your Vagrant Environment

```bash
Vagrant up
```

This command initializes and starts your Vagrant VM with Kubernetes.

### Step 2: Download kube-bench

```bash
curl -L  https://github.com/aquasecurity/kube-bench/releases/download/v0.8.0/kube-bench_0.8.0_linux_amd64.deb -o kube-bench_0.8.0_linux_amd64.deb
```

This downloads the kube-bench v0.8.0 Debian package to your local system. The `-L` flag follows any redirects, and `-o` specifies the output filename.

### Step 3: Install kube-bench

```bash
sudo dpkg -i kube-bench_0.8.0_linux_amd64.deb
```

This installs the downloaded kube-bench package using the Debian package manager.

## Verification

Once installed, you can verify the installation by running:

```bash
kube-bench --version
```

Or run a benchmark scan:

```bash
kube-bench run
```

This will output a detailed security scan report showing which CIS Kubernetes Benchmark controls pass, fail, or are not applicable to your cluster.

## Cleanup

To remove kube-bench if needed:

```bash
sudo dpkg -r kube-bench
```

To shut down your Vagrant VM:

```bash
Vagrant down
```
