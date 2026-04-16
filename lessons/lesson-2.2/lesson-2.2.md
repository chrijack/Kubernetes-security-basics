# Lesson 2.2 — Setting Up kops on Google Cloud Engine

> From the [Certified Kubernetes Security Specialist (CKS) Video Course](https://www.pearsonitcertification.com/store/certified-kubernetes-security-specialist-cks-video-9780138296476)

## Objective

Set up kops (Kubernetes Operations) on Google Cloud Engine (GCE) by configuring authentication, APIs, DNS, service accounts, and storage infrastructure needed to deploy and manage Kubernetes clusters on GCP.

## Prerequisites

- Google Cloud account with billing enabled
- `gcloud` CLI tool installed and configured
- `gsutil` CLI tool installed (included with Google Cloud SDK)
- Project ID where resources will be created
- Domain/cluster name for Kubernetes cluster

## Steps

### Step 1: Authenticate with Google Cloud

Authenticate the gcloud CLI with your Google Cloud account. This will open a browser for you to sign in.

```bash
gcloud auth login
```

### Step 2: Set the Project ID

Configure the default project ID where the Kubernetes cluster will be deployed. Replace `YOUR_PROJECT_ID` with your actual GCP project ID.

```bash
gcloud config set project YOUR_PROJECT_ID
```

### Step 3: Enable Required APIs

Enable the Compute Engine and Cloud DNS APIs, which are necessary for kops to manage Kubernetes infrastructure and DNS records on GCP.

```bash
gcloud services enable compute.googleapis.com
gcloud services enable dns.googleapis.com
```

### Step 4: Create DNS Zone

Create a managed DNS zone for your Kubernetes cluster. This will be used by kops to manage cluster DNS records. Replace `YOURCLUSTER` with your desired cluster name.

```bash
gcloud dns managed-zones create kubernetes-dns-zone --dns-name YOURCLUSTER.k8s.local. --description "DNS zone for Kubernetes"
```

### Step 5: Configure IAM Permissions

Create a service account for kops and assign the necessary roles (Compute Admin and Service Account User) to allow kops to manage cloud resources.

```bash
gcloud iam service-accounts create kops --display-name "kops service account"
gcloud projects add-iam-policy-binding YOUR_PROJECT_ID --member "serviceAccount:kops@YOUR_PROJECT_ID.iam.gserviceaccount.com" --role "roles/compute.admin"
gcloud projects add-iam-policy-binding YOUR_PROJECT_ID --member "serviceAccount:kops@YOUR_PROJECT_ID.iam.gserviceaccount.com" --role "roles/iam.serviceAccountUser"
```

### Step 6: Generate and Download Service Account Key

Generate a JSON key for the kops service account. This key will be used for authentication when deploying the cluster.

```bash
gcloud iam service-accounts keys create kops-key.json --iam-account kops@YOUR_PROJECT_ID.iam.gserviceaccount.com
```

### Step 7: Create GCS Bucket for kops State

Create a Google Cloud Storage bucket where kops will store the cluster state and configuration. Replace `YOUR_KOPS_STATE_STORE` with a unique bucket name.

```bash
gsutil mb -l us-central1 gs://YOUR_KOPS_STATE_STORE
```

## Verification

To verify the setup is complete:

1. Confirm the DNS zone exists:
   ```bash
   gcloud dns managed-zones list
   ```

2. Confirm the service account was created:
   ```bash
   gcloud iam service-accounts list
   ```

3. Confirm the GCS bucket was created:
   ```bash
   gsutil ls
   ```

4. Verify the service account key file exists in your current directory:
   ```bash
   ls -la kops-key.json
   ```

## Cleanup

To remove resources created during this setup:

```bash
# Delete the GCS bucket and its contents
gsutil -m rm -r gs://YOUR_KOPS_STATE_STORE

# Delete the DNS zone
gcloud dns managed-zones delete kubernetes-dns-zone

# Delete the service account
gcloud iam service-accounts delete kops@YOUR_PROJECT_ID.iam.gserviceaccount.com

# Remove the service account key file from your local machine
rm kops-key.json
```

**Note:** Ensure you replace `YOUR_PROJECT_ID`, `YOURCLUSTER`, and `YOUR_KOPS_STATE_STORE` with actual values before running any commands.
