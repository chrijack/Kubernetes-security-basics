# Lesson 1.3 — Kubernetes Cluster Setup with kops on GCE

> From the [Certified Kubernetes Security Specialist (CKS) Video Course](https://www.pearsonitcertification.com/store/certified-kubernetes-security-specialist-cks-video-9780138296476)

## Objective

Create a fully functional Kubernetes cluster on Google Cloud Engine using kops, configure it with proper state management, and validate that all nodes are running correctly. This lesson demonstrates the complete lifecycle of a Kubernetes cluster from creation through validation.

## Prerequisites

- Google Cloud Platform (GCP) account with billing enabled
- `gcloud` CLI installed and configured
- `kops` CLI installed
- `kubectl` installed
- Write permissions to create GCS buckets and Compute Engine resources
- Authentication configured for GCP

## Steps

### Step 1: Initialize gcloud and Authenticate

Set up authentication to Google Cloud by initializing gcloud and logging in with your credentials.

```bash
gcloud init
gcloud auth application-default login
```

### Step 2: Create a GCS Bucket for kops State Store

Create a Google Cloud Storage bucket that will serve as the central state store for kops. This bucket stores the cluster configuration and metadata.

```bash
gsutil mb gs://k8sdemo1337/
```

### Step 3: Set the KOPS_STATE_STORE Environment Variable

Configure the environment variable to point to the GCS bucket created in the previous step. This tells kops where to store and retrieve cluster state.

```bash
export KOPS_STATE_STORE=gs://k8sdemo1337/
```

### Step 4: Retrieve Project ID and Create the Cluster

Get your GCP project ID and use kops to create a Kubernetes cluster named `simple.k8s.local` in the `us-central1-a` zone.

```bash
PROJECT=$(gcloud config get-value project)
kops create cluster simple.k8s.local --zones us-central1-a --state ${KOPS_STATE_STORE}/ --project=${PROJECT}
```

### Step 5: Edit the Cluster Configuration

Open the cluster configuration in your default editor to make any necessary adjustments to cluster settings, node sizes, networking, or other parameters.

```bash
kops edit cluster simple.k8s.local
```

### Step 6: Update the Cluster

Apply the cluster configuration changes to create the actual infrastructure on GCE. The `--yes` flag confirms the changes without prompting.

```bash
kops update cluster --name simple.k8s.local --yes --admin
```

### Step 7: Validate the Cluster

Verify that the cluster is running correctly and all nodes are healthy. This command checks cluster health and readiness.

```bash
kops validate cluster
```

### Step 8: Get Instance Group Details

List all instance groups in the cluster to see the compute resources created.

```bash
kops get instancegroup --state ${KOPS_STATE_STORE} --name simple.k8s.local
```

### Step 9: Get Full Cluster Details

Display the complete cluster configuration in YAML format for reference and troubleshooting.

```bash
kops get cluster --state ${KOPS_STATE_STORE}/ simple.k8s.local -oyaml
```

### Step 10: Display Kubernetes Nodes

List all nodes in the Kubernetes cluster to confirm they are running and ready.

```bash
kubectl get nodes
```

## Verification

To confirm the cluster is properly set up:

1. The `kops validate cluster` command completes successfully
2. `kubectl get nodes` returns at least one node in Ready status
3. All instance groups are listed with appropriate sizes and configurations
4. The cluster configuration YAML displays with no errors

## Cleanup

When finished, delete the Kubernetes cluster and remove the GCS state store bucket.

```bash
# Delete the Kubernetes cluster
kops delete cluster simple.k8s.local --yes

# Delete the GCS bucket
gsutil rb gs://k8sdemo1337/
```

Note: The `--yes` flag on `kops delete` confirms deletion without prompting. This operation is irreversible.
