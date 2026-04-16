# Lesson 12.3 — Disabling Default Settings for Service Accounts

> From the [Certified Kubernetes Security Specialist (CKS) Video Course](https://www.pearsonitcertification.com/store/certified-kubernetes-security-specialist-cks-video-9780138296476)

## Objective
Demonstrate how Kubernetes automatically mounts service account tokens into pods by default, and learn how to disable this behavior at both the pod level and namespace level for improved security.

## Prerequisites
- A running Kubernetes cluster
- `kubectl` configured to access the cluster
- Basic knowledge of Kubernetes pods and service accounts

## Steps

### Step 1: Show How Service Account Token is Shared with a Pod by Default

Create a pod that uses the default service account to demonstrate that the token is mounted by default:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: default-token-pod
spec:
  containers:
  - name: default-token-container
    image: nginx
EOF
```

Verify the token mount:

```bash
kubectl get pod default-token-pod -o yaml | grep -A 2 'volumeMounts'
```

You should see output similar to this, showing the service account token being mounted:

```yaml
volumeMounts:
- mountPath: /var/run/secrets/kubernetes.io/serviceaccount
  name: kube-api-access-<random-string>
```

Also, check the pod for the actual token:

```bash
kubectl exec default-token-pod -- ls /run/secrets/kubernetes.io/serviceaccount
kubectl exec default-token-pod -- cat /run/secrets/kubernetes.io/serviceaccount/token
kubectl exec default-token-pod -- sh -c 'cat /run/secrets/kubernetes.io/serviceaccount/token | cut -d "." -f2 | base64 --decode'
```

### Step 2: Disable Auto-mounting of Service Account Tokens

#### For a Specific Pod

Create a pod with the `automountServiceAccountToken` field set to `false`:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  automountServiceAccountToken: false
  containers:
  - name: my-container
    image: nginx
EOF
```

Verify the token is not mounted:

```bash
kubectl get pod my-pod -o yaml | grep -A 2 'volumeMounts'
kubectl exec my-pod -- ls /run/secrets/kubernetes.io/serviceaccount
```

#### For the Default Service Account in a Namespace

Create a new namespace:

```bash
kubectl create namespace secure-ns
```

Patch the default service account in this namespace to disable auto-mounting of tokens:

```bash
kubectl patch serviceaccount default -n secure-ns -p '{"automountServiceAccountToken":false}'
```

Verify the auto-mount setting for the default Service Account:

```bash
kubectl get serviceaccount default -n secure-ns -o yaml
```

### Step 3: Use the Default Service Account in a Pod within the Namespace

Create a pod in the `secure-ns` namespace using the `default` service account:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
  namespace: secure-ns
spec:
  serviceAccountName: default
  containers:
  - name: secure-container
    image: nginx
EOF
```

Verify the token is not mounted:

```bash
kubectl get pod secure-pod -n secure-ns -o yaml | grep -A 2 'volumeMounts'
kubectl exec secure-pod -n secure-ns -- ls /run/secrets/kubernetes.io/serviceaccount
```

## Verification

- The first pod (`default-token-pod`) should show the service account token mounted at `/var/run/secrets/kubernetes.io/serviceaccount`
- The second pod (`my-pod`) should have no volumeMount entry for the service account
- The `default` service account in `secure-ns` should have `automountServiceAccountToken: false`
- The pod (`secure-pod`) in the `secure-ns` namespace should not have the service account token mounted, despite using the default service account

## Cleanup

Remove all resources created during this lesson:

```bash
# Delete the pods from the default namespace
kubectl delete pod default-token-pod my-pod

# Delete the pod and namespace from the secure-ns namespace
kubectl delete namespace secure-ns
```
