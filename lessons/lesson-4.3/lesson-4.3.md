# Lesson 4.3 — Configuring Developer Permissions with Kubernetes RBAC

> From the [Certified Kubernetes Security Specialist (CKS) Video Course](https://www.pearsonitcertification.com/store/certified-kubernetes-security-specialist-cks-video-9780138296476)

## Objective
Create a Role defining permissions for a developer and bind it to a user account using a RoleBinding to demonstrate fine-grained permission control in Kubernetes.

## Prerequisites
- Kubernetes cluster running
- kubectl configured with appropriate context access
- Developer user account created (from previous lesson)

## Steps

### Step 1: Create the Developer Role

First, switch to the admin context:
```bash
kubectl config use-context kubernetes-admin@kubernetes
```

Create a YAML file defining the developer Role with specific resource permissions:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: developer
rules:
- apiGroups: [""] 
  resources: ["pods", "services", "persistentvolumeclaims"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets", "statefulsets"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: ["batch"]
  resources: ["jobs", "cronjobs"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```

Apply the Role:
```bash
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: developer
rules:
- apiGroups: [""] 
  resources: ["pods", "services", "persistentvolumeclaims"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets", "statefulsets"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: ["batch"]
  resources: ["jobs", "cronjobs"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
EOF
```

This Role allows the developer to perform common actions (get, list, watch, create, update, patch, delete) on core resources (pods, services, PVCs), apps resources (deployments, replicasets, statefulsets), and batch resources (jobs, cronjobs). However, it restricts access to cluster-level and sensitive resources like nodes, namespaces, roles, etc.

### Step 2: Bind the Role to the User

Create a RoleBinding to assign the developer Role to the user:
```bash
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developer-binding
  namespace: default
subjects:
- kind: User
  name: user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
EOF
```

This binds the "developer" Role to the "user" account created in the previous demo.

### Step 3: Test the Developer Permissions

Switch to the developer's context:
```bash
kubectl config use-context user-context
```

**Test 1: List pods (allowed)**
```bash
kubectl get pods
```

**Test 2: List nodes (not allowed)**
```bash
kubectl get nodes
```

This should result in a permissions error, as the developer Role does not allow access to node resources.

**Test 3: Create a deployment (allowed)**
```bash
kubectl create deployment nginx --image=nginx
```

**Test 4: Attempt to create a ClusterRole (not allowed)**
```bash
kubectl create clusterrole test --verb=get --resource=pods
```

This should fail due to insufficient permissions. ClusterRoles are cluster-scoped resources not included in the developer Role.

## Verification

- Developer can list and manage pods, deployments, and other namespaced workload resources
- Developer receives permission errors when attempting to access cluster-scoped resources (nodes, ClusterRoles, etc.)
- Permissions are properly restricted to the default namespace and specified resource types

## Summary

In this demo, we:
1. Created a "developer" Role with permissions for common namespaced resources
2. Bound the Role to the "user" account using a RoleBinding
3. Tested the developer's permissions, confirming access is restricted as expected

This illustrates how Kubernetes RBAC allows defining fine-grained permissions for different user roles. The developer can manage app workloads in their namespace but cannot impact cluster-level resources or other namespaces.
