# Lesson 11.2 — Creating a User Account in Kubernetes

> From the [Certified Kubernetes Security Specialist (CKS) Video Course](https://www.pearsonitcertification.com/store/certified-kubernetes-security-specialist-cks-video-9780138296476)

## Objective
Create a user account in Kubernetes 1.30 by generating certificates, creating a CertificateSigningRequest (CSR) resource, configuring kubectl contexts, and testing the user's permissions.

## Prerequisites
- Kubernetes 1.30 cluster running
- `kubectl` configured with admin access
- `openssl` installed
- Access to Kubernetes CA certificate and key

## Steps

### Step 1: Create the User Credentials

Generate a private key and certificate signing request for the user:

```bash
openssl genrsa -out user.key 2048
openssl req -new -key user.key -out user.csr -subj "/CN=user/O=acme"
```

### Step 2: Retrieve Kubernetes CA Certificate and Key

Use Vagrant commands to retrieve the CA certificate and key from the Vagrant virtual machine:

```bash
# SSH into the Vagrant VM and copy the CA certificate and key to a shared directory
vagrant ssh control-plane -c 'sudo cp /etc/kubernetes/pki/ca.crt /vagrant'
vagrant ssh control-plane -c 'sudo cp /etc/kubernetes/pki/ca.key /vagrant'
```

Sign the CSR with the Kubernetes CA certificate and key:

```bash
openssl x509 -req -in user.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out user.crt -days 365
```

### Step 3: Create the User in Kubernetes

Create a CertificateSigningRequest (CSR) resource in Kubernetes:

```bash
Cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: user
spec:
  request: $(cat user.csr | base64 | tr -d '\n') 
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
EOF
```

View the pending CSR:

```bash
kubectl get csr
```

Approve the CSR:

```bash
kubectl certificate approve user
```

Download the signed certificate:

```bash
kubectl get csr user -o jsonpath='{.status.certificate}' | base64 --decode > user.crt
```

### Step 4: Configure kubectl for the User

Configure `kubectl` to use the user credentials:

```bash
kubectl config set-credentials user --client-certificate=user.crt --client-key=user.key
kubectl config set-context user-context --cluster=kubernetes --user=user
kubectl config use-context user-context
```

Now `kubectl` commands will authenticate as the newly created user. The user's permissions can be further defined using Kubernetes RBAC (Role and RoleBinding resources).

## Testing Permissions

With no role bindings assigned, the user can perform only limited operations:

**Commands that will succeed:**

```bash
kubectl version
kubectl cluster-info
kubectl api-resources
kubectl api-versions
kubectl config view
```

These commands provide basic cluster information and available API resources without exposing sensitive data.

**Permission checking commands:**

```bash
kubectl auth can-i list pods
kubectl auth can-i create deployments
kubectl auth can-i delete nodes
```

The `auth can-i` command allows users to check their permissions but does not grant them.

**Commands that will fail (authorization denied):**

```bash
kubectl get pods
kubectl get nodes
kubectl create deployment nginx --image=nginx
kubectl delete namespace default
```

**Local kubeconfig configuration commands:**

```bash
kubectl config get-contexts
kubectl config set-context user-context --user=user
kubectl config use-context user-context
```

These affect only the user's local kubectl configuration and grant no additional cluster permissions.

## Verification

Without any role bindings, the user has read-only access to basic cluster information and can check their own permissions. Any attempts to view or modify actual cluster resources will be blocked by Kubernetes RBAC. Additional permissions must be granted by a cluster administrator through role bindings.

## Cleanup

Remove the user context and credentials from kubectl:

```bash
kubectl config delete-context user-context
kubectl config delete-user user
```
