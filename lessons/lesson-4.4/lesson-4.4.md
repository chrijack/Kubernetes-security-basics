# Lesson 4.4 — ClusterRoles, Role Aggregation, and Privilege Validation

> From the [Certified Kubernetes Security Specialist (CKS) Video Course](https://www.pearsonitcertification.com/store/certified-kubernetes-security-specialist-cks-video-9780138296476)

## Objective

Demonstrate how to create ClusterRoles, implement Role Aggregation to combine permissions, and validate privilege levels in Kubernetes 1.30. This lesson emphasizes the principle of least privilege when managing cluster-wide permissions.

## Prerequisites

- Kubernetes 1.30 cluster setup
- `kubectl` configured and authenticated
- Understanding of RBAC fundamentals from previous lessons

## Steps

### Step 1: Create and Validate Limited Cluster Admin Role

Create the limited cluster admin role:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: limited-cluster-admin
rules:
- apiGroups: [""]
  resources: ["pods", "services", "persistentvolumeclaims"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
EOF
```

Validate the limited cluster admin role:

```bash
kubectl describe clusterrole limited-cluster-admin
kubectl get clusterrole limited-cluster-admin -o yaml
```

This command will show the permissions granted by the limited-cluster-admin role.

### Step 2: Create and Validate Individual Roles for Aggregation

Create individual roles with aggregation labels:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-reader
  labels:
    rbac.example.com/aggregate-to-limited-admin: "true"
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
  labels:
    rbac.example.com/aggregate-to-limited-admin: "true"
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list"]
EOF
```

Validate these individual roles:

```bash
kubectl describe clusterrole node-reader
kubectl describe clusterrole pod-reader
```

### Step 3: Create Aggregated Role

Create an aggregated role that combines roles based on matching labels:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: aggregated-limited-admin
aggregationRule:
  clusterRoleSelectors:
  - matchLabels:
      rbac.example.com/aggregate-to-limited-admin: "true"
rules: []
EOF
```

Verify the aggregated role was created:

```bash
kubectl describe clusterrole aggregated-limited-admin
```

### Step 4: Bind the Aggregated Role to a User

Create a ClusterRoleBinding to assign the aggregated role to a user:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: aggregated-limited-admin-binding
subjects:
- kind: User
  name: user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: aggregated-limited-admin
  apiGroup: rbac.authorization.k8s.io
EOF
```

### Step 5: Validate Aggregated Permissions

Check the permissions of the aggregated role:

```bash
kubectl auth can-i --list --as=user
```

This command will show all permissions granted by the aggregated-limited-admin role, which should include permissions from limited-cluster-admin, pod-reader, and node-reader.

### Step 6: Test Specific Permissions

Test allowed actions:

```bash
kubectl auth can-i get pods --as=user
kubectl auth can-i list nodes --as=user
kubectl auth can-i create deployments --as=user
```

Test disallowed actions:

```bash
kubectl auth can-i delete nodes --as=user
kubectl auth can-i create clusterroles --as=user
```

## Verification

After completing the steps, verify the configuration:

1. Confirm all ClusterRoles were created successfully
2. Verify the aggregation rule is functioning by checking that permissions from all labeled roles are included
3. Validate that the user has the expected permissions and restrictions

## Cleanup

After completing the demonstration, clean up the resources:

```bash
kubectl delete clusterrole limited-cluster-admin
kubectl delete clusterrole pod-reader
kubectl delete clusterrole node-reader
kubectl delete clusterrole aggregated-limited-admin
kubectl delete clusterrolebinding aggregated-limited-admin-binding
```

## Key Takeaways

This demonstration shows how to:

- Create and validate a limited cluster admin role
- Create and validate individual roles for aggregation
- Use Role Aggregation to combine multiple roles into a single aggregated role
- Bind the aggregated role to a user
- Validate the aggregated permissions
- Test specific allowed and disallowed actions

Role aggregation simplifies permission management by allowing you to combine simpler, focused roles into more complex permission sets. Always carefully review and test RBAC configurations in a safe environment before applying them to production clusters.
