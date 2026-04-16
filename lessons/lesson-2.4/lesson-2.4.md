# Lesson 2.4 — Getting Minikube Up and Running with Cilium

> From the [Certified Kubernetes Security Specialist (CKS) Video Course](https://www.pearsonitcertification.com/store/certified-kubernetes-security-specialist-cks-video-9780138296476)

## Objective

Start a local Minikube cluster with multiple nodes, install and configure Cilium as the CNI plugin with Hubble observability, and verify cluster functionality by deploying NGINX and testing network connectivity.

## Prerequisites

- Minikube installed and functional
- kubectl installed and configured
- Cilium CLI installed (`cilium` command available)
- Docker or appropriate container runtime running
- Kubernetes v1.29.4 support

## Steps

### Step 1: Start Minikube Cluster with Two Nodes

Initialize a multi-node Minikube cluster with a specific Kubernetes version:

```bash
minikube start --nodes 2 --kubernetes-version=v1.29.4
```

This creates a cluster with two nodes running Kubernetes v1.29.4, providing a more realistic environment for testing network policies and multi-node scenarios.

### Step 2: Enable Ingress-DNS Addon

Enable the DNS support for Ingress resources:

```bash
minikube addons enable ingress-dns
```

This addon provides DNS resolution for ingress rules, allowing you to use hostnames rather than IP addresses.

### Step 3: Check Initial Cilium Status

Verify Cilium's current status before installation:

```bash
cilium status
```

You may see that Cilium is not yet installed at this stage.

### Step 4: Install Cilium

Install Cilium as the cluster networking plugin:

```bash
cilium install
```

This installs Cilium across the cluster to provide network policies, service mesh capabilities, and observability.

### Step 5: Verify Cilium Installation

Confirm that Cilium has been successfully installed and is running:

```bash
cilium status
```

This should now show Cilium components as healthy and operational.

### Step 6: Enable Cilium Hubble

Enable Hubble, Cilium's observability component, for network visibility and debugging:

```bash
cilium hubble enable
```

Hubble provides flow visibility, DNS monitoring, and security policy debugging.

### Step 7: Check Status After Hubble Enablement

Verify the cluster status with Hubble now enabled:

```bash
cilium status
```

Confirm all components, including Hubble, are operational.

### Step 8: Port Forward for Hubble

Start a port-forward tunnel to access Hubble's observability UI:

```bash
cilium hubble port-forward&
```

This runs the port-forward in the background, making Hubble accessible on localhost.

### Step 9: Check Hubble Status

Verify that Hubble is operational and connected:

```bash
hubble status
```

### Step 10: Test Cilium Connectivity

Run Cilium's built-in connectivity test to verify network policies and communication:

```bash
cilium connectivity test
```

This tests end-to-end connectivity between pods using various network policy scenarios.

### Step 11: Deploy NGINX Application

Deploy a sample NGINX application to the cluster:

```bash
kubectl apply -f https://k8s.io/examples/application/deployment.yaml
```

This creates an NGINX deployment with multiple replicas.

### Step 12: Verify NGINX Pods Deployment

Check that all NGINX pods have been successfully created and are running:

```bash
kubectl get pods
```

Wait for all pods to reach the Running state.

### Step 13: Expose NGINX as a Service

Create a NodePort service to expose the NGINX deployment:

```bash
kubectl expose deployment/nginx-deployment --port=80 --type=NodePort
```

This makes the NGINX application accessible via a NodePort on the cluster nodes.

### Step 14: Get Service Details

List all services to see the assigned port and cluster IP:

```bash
kubectl get svc
```

Note the NodePort assigned to the nginx-deployment service.

### Step 15: View Minikube Services

List all services available through Minikube:

```bash
minikube service list
```

### Step 16: Access NGINX via Minikube

Open the NGINX service in your default browser via Minikube:

```bash
minikube service nginx-deployment
```

This automatically routes you to the NGINX deployment using the exposed NodePort.

## Verification

- All pods in the cluster should be in the `Running` state
- `cilium status` should show all Cilium components as healthy
- `hubble status` should indicate a successful connection
- `cilium connectivity test` should complete with all tests passing
- NGINX deployment pods should be accessible and serving HTTP responses
- `minikube service nginx-deployment` should successfully open the NGINX welcome page in your browser

## Cleanup

To tear down the cluster and free resources:

```bash
minikube delete
```

This stops and removes all Minikube clusters and associated data.
