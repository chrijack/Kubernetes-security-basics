# Lesson 5.4 — Verifying Service Account Security and Ensuring Compliance

> From the [Certified Kubernetes Security Specialist (CKS) Video Course](https://www.pearsonitcertification.com/store/certified-kubernetes-security-specialist-cks-video-9780138296476)

## Objective

Learn how to verify and audit service account security configurations in Kubernetes, ensuring compliance with the principle of least privilege. This lesson covers checking auto-mounting behavior, validating permissions, restricting namespace access, preventing impersonation, and auditing token settings.

## Prerequisites

- A Kubernetes cluster with kubectl access
- An existing namespace named `secure-ns` with a service account named `default`
- Familiarity with RBAC (Role-Based Access Control) and Kubernetes service accounts
- Understanding of least privilege security principles

## Steps

### Step 1: Check Auto-mounting of Service Account Tokens

Verify the auto-mounting setting for the service account to ensure it is disabled:

```bash
kubectl get serviceaccount default -n secure-ns -o jsonpath='{.automountServiceAccountToken}'
```

This command checks whether the service account automatically mounts the token into pods. Should return `false` for enhanced security.

### Step 2: Verify Least Privilege Principle

Check the permissions assigned to the service account to ensure they follow the principle of least privilege:

```bash
kubectl auth can-i --as=system:serviceaccount:secure-ns:default --list -n secure-ns
```

This lists all permissions currently granted to the service account, helping you verify that only necessary permissions are assigned.

### Step 3: Check for Critical Permissions

Verify if the service account has any critical permissions that could pose security risks.

**Check if the service account can create pods:**

```bash
kubectl auth can-i create pods --as=system:serviceaccount:secure-ns:default -n secure-ns
```

**Check if the service account can create deployments:**

```bash
kubectl auth can-i create deployments --as=system:serviceaccount:secure-ns:default -n secure-ns
```

**Check if the service account can get secrets:**

```bash
kubectl auth can-i get secrets --as=system:serviceaccount:secure-ns:default -n secure-ns
```

**Check if the service account can list nodes:**

```bash
kubectl auth can-i list nodes --as=system:serviceaccount:secure-ns:default -n secure-ns
```

**Check if the service account can create service accounts:**

```bash
kubectl auth can-i create serviceaccounts --as=system:serviceaccount:secure-ns:default -n secure-ns
```

### Step 4: Validate Namespace Restrictions

Ensure the service account's permissions are limited to its own namespace:

```bash
kubectl auth can-i list pods --as=system:serviceaccount:secure-ns:default --all-namespaces
```

This command verifies that the service account cannot access resources in other namespaces, maintaining proper isolation.

### Step 5: Check for Impersonation Permissions

Verify that the service account cannot impersonate other users or groups:

```bash
kubectl auth can-i impersonate user --as=system:serviceaccount:secure-ns:default
kubectl auth can-i impersonate group --as=system:serviceaccount:secure-ns:default
```

Impersonation permissions are dangerous and should be carefully restricted to prevent privilege escalation.

### Step 6: Audit Role and RoleBinding

Review the roles and role bindings associated with the service account to ensure they are appropriate and follow the principle of least privilege:

```bash
kubectl get roles,rolebindings --all-namespaces -o wide | grep default
```

This command helps you identify what roles are bound to the service account and verify they align with security policies.

### Step 7: Validate Service Account Token Settings

Check if the service account is using the newer, more secure token request API for Kubernetes 1.24+:

```bash
kubectl get pods -n secure-ns -o jsonpath='{.items[*].spec.containers[*].volumeMounts}' | grep service-account-token
```

This verifies the service account token mounting configuration in running pods.

## Verification

By following these steps, you can verify that your service account adheres to security best practices, including:

- The principle of least privilege (only necessary permissions assigned)
- Namespace isolation (access restricted to the appropriate namespace)
- Proper token management (secure token handling and auto-mounting disabled)
- Prevention of privilege escalation (impersonation permissions denied)
- Audit compliance (all roles and bindings reviewed and documented)

## Cleanup

Remove the test configuration when completed:

```bash
kubectl delete namespace secure-ns
```
