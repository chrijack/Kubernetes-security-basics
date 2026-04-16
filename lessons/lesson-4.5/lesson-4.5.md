# Lesson 4.5 — Inspecting, Validating, and Troubleshooting RBAC Privileges

> From the [Certified Kubernetes Security Specialist (CKS) Video Course](https://www.pearsonitcertification.com/store/certified-kubernetes-security-specialist-cks-video-9780138296476)

## Objective

Demonstrate how to inspect, validate, and troubleshoot the privileges for the limited-admin-user in Kubernetes 1.30, ensuring proper RBAC configuration and identifying any discrepancies in role-based access control.

## Prerequisites

- Kubernetes cluster with version 1.30 or later
- `kubectl` command-line tool configured with appropriate access
- Previously created ClusterRole `limited-cluster-admin`
- Previously created ClusterRoleBinding `aggregated-limited-admin-binding`
- limited-admin-user configured in the cluster

## Steps

### Step 1: Inspect the ClusterRoleBinding

First, let's examine all ClusterRoles and ClusterRoleBindings, then check the specific binding for the limited-admin-user:

```bash
# List all ClusterRoles
kubectl get clusterroles

# View details of our previously created ClusterRole
kubectl describe clusterrole limited-cluster-admin

# List all ClusterRoleBindings
kubectl get clusterrolebindings

# View details of our previously created ClusterRoleBinding
kubectl describe clusterrolebinding aggregated-limited-admin-binding

# Get the full YAML of the ClusterRoleBinding
kubectl get clusterrolebinding aggregated-limited-admin-binding -o yaml
```

Verify that the output includes:
```yaml
subjects:
- kind: User
  name: limited-admin-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: aggregated-limited-admin
  apiGroup: rbac.authorization.k8s.io
```

### Step 2: Validate User Permissions

Now, let's check the permissions granted to limited-admin-user:

```bash
kubectl auth can-i --list --as=user
```

This command will show all the permissions granted to the user across all API groups and resources.

### Step 3: Inspect the Aggregated ClusterRole

Let's examine the aggregated ClusterRole to understand what permissions it grants:

```bash
kubectl get clusterrole aggregated-limited-admin -o yaml
```

Look for the `aggregationRule` section and the `rules` section to understand how permissions are being aggregated and what specific permissions are granted.

### Step 4: Verify Specific Permissions

Test some specific permissions that the limited-admin-user should have:

```bash
kubectl auth can-i get pods --as=user
kubectl auth can-i create deployments --as=user
kubectl auth can-i delete nodes --as=user
```

### Step 5: Troubleshoot Any Discrepancies

If any of the above checks reveal unexpected permissions or lack thereof:

1. Review the individual ClusterRoles that are being aggregated:
   ```bash
   kubectl get clusterroles --show-labels | grep rbac.example.com/aggregate-to-limited-admin=true
   ```
   Look for roles with the label `rbac.example.com/aggregate-to-limited-admin: "true"`.

2. Inspect each of these roles:
   ```bash
   kubectl get clusterrole limited-cluster-admin -o yaml
   ```

3. Remove aggregation for limited-cluster-admin:
   ```bash
   kubectl edit clusterrole limited-cluster-admin
   ```

4. If unexpected permissions are present, check if there are any additional ClusterRoleBindings for the user:
   ```bash
   kubectl get clusterrolebindings --all-namespaces -o yaml | grep user
   ```

### Step 6: Test Real-world Scenarios

Try to perform actual operations as the limited-admin-user:

```bash
kubectl --as=user get pods --all-namespaces
kubectl --as=user create deployment nginx --image=nginx
kubectl --as=user delete node <node-name>
```

### Step 7: Review and Adjust Permissions

Based on the results of your tests:

1. If necessary, modify the aggregated ClusterRole or its component roles:
   ```bash
   kubectl edit clusterrole aggregated-limited-admin
   ```

2. Or create a new ClusterRole with adjusted permissions and update the ClusterRoleBinding:
   ```bash
   kubectl create clusterrole adjusted-limited-admin --verb=get,list,watch --resource=pods,deployments
   kubectl edit clusterrolebinding aggregated-limited-admin-binding
   ```

## Verification

This process allows you to thoroughly inspect, validate, and troubleshoot the RBAC privileges for the limited-admin-user. By following these steps, you can ensure that the user has the intended permissions, identify any discrepancies, and make necessary adjustments to maintain the principle of least privilege.

## Cleanup

No specific cleanup is required for this lesson. The ClusterRoles and ClusterRoleBindings created in previous lessons will remain in place for reference and testing purposes.
