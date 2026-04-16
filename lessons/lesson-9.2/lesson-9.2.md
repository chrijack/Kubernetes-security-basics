# Lesson 9.2 — Verifying Kubernetes Binary Integrity

> From the [Certified Kubernetes Security Specialist (CKS) Video Course](https://www.pearsonitcertification.com/store/certified-kubernetes-security-specialist-cks-video-9780138296476)

## Objective

Demonstrate two critical methods for verifying the integrity and authenticity of Kubernetes binaries before deployment:
1. **SHA256 checksum verification** — ensures binaries have not been corrupted or tampered with during download
2. **Cosign signature verification** — cryptographically verifies that binaries were signed by the official Kubernetes release engineering team

Understanding binary verification is essential for CKS exam success and real-world Kubernetes deployments. Supply chain security is a critical attack vector — compromised binaries can give attackers complete control of your cluster.

## Prerequisites

- Linux system with shell access
- `wget` and `curl` utilities installed
- `cosign` tool installed (`https://github.com/sigstore/cosign`)
- Network access to Kubernetes release servers (`dl.k8s.io`)
- Internet connectivity to Google accounts for OIDC verification

## Steps

### Step 1: Download and Verify Binary Using SHA256 Checksum

This method verifies that the downloaded binary matches the official release — preventing corruption from network issues or MITM attacks.

Download the Kubernetes API server binary and its checksum:

```bash
wget https://dl.k8s.io/v1.29.4/bin/linux/arm64/kube-apiserver
wget https://dl.k8s.io/v1.29.4/bin/linux/arm64/kube-apiserver.sha256
```

Verify the checksum matches:

```bash
echo "$(cat kube-apiserver.sha256) kube-apiserver" | sha256sum --check
```

**Expected output:** `kube-apiserver: OK`

If verification fails, the binary has been corrupted or tampered with. Do not use it.

### Step 2: Download Binaries for Signature Verification

This method verifies that the binary was digitally signed by the official Kubernetes release engineering team, proving authenticity.

Set variables for the release and binary:

```bash
URL=https://dl.k8s.io/release/v1.29.4/bin/linux/arm64
BINARY=kubectl
```

Define files needed for verification (binary, signature, and certificate):

```bash
FILES=(
    "$BINARY"
    "$BINARY.sig"
    "$BINARY.cert"
)
```

Download all three files:

```bash
for FILE in "${FILES[@]}"; do
    curl -sSfL --retry 3 --retry-delay 3 "$URL/$FILE" -o "$FILE"
done
```

The three files are:
- **kubectl** — the binary to verify
- **kubectl.sig** — digital signature created by the release engineering team
- **kubectl.cert** — certificate containing the public key and identity metadata

### Step 3: Verify Digital Signature Using Cosign

Use cosign to verify the binary signature against the certificate:

```bash
cosign verify-blob "$BINARY" \
  --signature "$BINARY".sig \
  --certificate "$BINARY".cert \
  --certificate-identity krel-staging@k8s-releng-prod.iam.gserviceaccount.com \
  --certificate-oidc-issuer https://accounts.google.com
```

**Key parameters:**
- `--certificate-identity` — the official service account that signed the release
- `--certificate-oidc-issuer` — Google's OpenID Connect provider (verifies the identity was authenticated by Google)

**Expected output:** Success message confirming the signature is valid and signed by the expected identity.

## Verification

A successful verification confirms:
1. ✓ The binary has not been corrupted during download (SHA256)
2. ✓ The binary was signed by the official Kubernetes release engineering team (cosign)
3. ✓ The signature was verified against Google's OIDC provider (proof of identity)
4. ✓ The identity matches the expected service account (`krel-staging@k8s-releng-prod.iam.gserviceaccount.com`)

Any failure in these steps indicates a potential security issue and the binary should not be used.

## Cleanup

Remove downloaded files:

```bash
rm -f kube-apiserver kube-apiserver.sha256 kubectl kubectl.sig kubectl.cert
```

## Supply Chain Security Context

**Why binary verification matters for CKS:**

- **Attack vector** — Compromised binaries are a high-impact supply chain attack (e.g., SolarWinds, Codecov incidents)
- **Exam focus** — CKS explicitly tests knowledge of binary verification and admission control for supply chain security
- **Production practice** — Real-world deployments must verify all binaries before running them in clusters
- **Two-layer defense** — SHA256 catches accidental corruption; cosign catches intentional tampering with cryptographic proof

Always verify binaries before deployment. Never run unverified binaries in production clusters.
