# Lesson 5.2 — Configuring a Service Account in Kubernetes 1.30

> From the [Certified Kubernetes Security Specialist (CKS) Video Course](https://www.pearsonitcertification.com/store/certified-kubernetes-security-specialist-cks-video-9780138296476)

## Objective
Learn how to create and configure Service Accounts in Kubernetes, including setting up Roles and RoleBindings to control permissions. This lesson demonstrates the complete workflow of creating a Service Account, defining access rules, binding them together, and testing the resulting permissions.

## Prerequisites
- A running Kubernetes 1.30 cluster
- kubectl configured with cluster access
- Basic understanding of Kubernetes namespaces and RBAC concepts

## Steps

### Step 1: Create a Namespace (Optional)
Create an isolated namespace for this exercise:
```bash
kubectl create namespace demo-ns
```

### Step 2: Create a Service Account
Create a Service Account named `demo-sa` in the `demo-ns` namespace:
```bash
kubectl create serviceaccount demo-sa -n demo-ns
```

Verify the service account creation:
```bash
kubectl get serviceaccount demo-sa -n demo-ns -o yaml
```

### Step 3: Create a Role
Define a Role that grants read-only permissions on pods. This role will allow the Service Account to get, watch, and list pods:
```bash
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: demo-ns
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
EOF
```

### Step 4: Bind the Role to the Service Account
Create a RoleBinding to associate the `pod-reader` Role with the `demo-sa` Service Account:
```bash
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: demo-ns
subjects:
- kind: ServiceAccount
  name: demo-sa
  namespace: demo-ns
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
EOF
```

Verify the RoleBinding was created:
```bash
kubectl get rolebinding read-pods -n demo-ns -o yaml
```

### Step 5: Create a Pod Using the Service Account
Deploy a pod that uses the `demo-sa` Service Account:
```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: demo-pod
  namespace: demo-ns
spec:
  serviceAccountName: demo-sa
  containers:
  - name: demo-container
    image: nginx:1.21
EOF
```

Verify the pod was created and the Service Account is associated:
```bash
kubectl get pod demo-pod -n demo-ns -o yaml | grep serviceAccount
```

### Step 6: Test Service Account Permissions
Test the permissions granted to the Service Account. The first command should succeed (getting pods is allowed), and the second should fail (creating pods is not allowed):
```bash
kubectl auth can-i get pods --as=system:serviceaccount:demo-ns:demo-sa -n demo-ns
kubectl auth can-i create pods --as=system:serviceaccount:demo-ns:demo-sa -n demo-ns
```

## Verification
- Service Account `demo-sa` exists in the `demo-ns` namespace
- Role `pod-reader` grants read permissions on pods
- RoleBinding `read-pods` connects the Service Account to the Role
- Pod `demo-pod` is running with the Service Account assigned
- Authorization test confirms: read access is granted, write access is denied

## Cleanup
Remove all resources created during this lesson:
```bash
kubectl delete namespace demo-ns
```
