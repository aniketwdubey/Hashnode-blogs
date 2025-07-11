---
title: "Securing GKE with Let's Encrypt Wildcard Certificates via GoDaddy DNS and cert-manager"
seoTitle: "Secure GKE with Let's Encrypt Wildcard SSL via GoDaddy & cert-manager"
seoDescription: "Learn how to secure your GKE applications with Let's Encrypt wildcard SSL certificates using cert-manager and GoDaddy DNS automation. Full production-grade "
datePublished: Fri Jul 11 2025 21:01:36 GMT+0000 (Coordinated Universal Time)
cuid: cmczaxzvs000d02la3k7w6ygw
slug: securing-gke-with-lets-encrypt-wildcard-certificates-via-godaddy-dns-and-cert-manager
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1752267067737/f0ce4903-4852-4ebc-8c37-6bab4008c788.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1752267389302/12a56841-3090-4114-bcaa-465bfca045e5.png
tags: cloud, software-development, docker, aws, security, google-cloud, kubernetes, cloud-computing, google, devops, gke

---

Managing TLS certificates in a scalable and secure manner is crucial when deploying multi-tenant or subdomain-rich applications on Google Kubernetes Engine (GKE). This guide walks you through a production-grade setup for acquiring and managing Let's Encrypt wildcard SSL certificates using cert-manager and the **GoDaddy DNS webhook solver**.

---

## Why Wildcard Certificates?

Wildcard certificates (e.g., `*.yourdomain.com`) simplify TLS management across multiple subdomains and services, ensuring encrypted HTTPS connections without manually managing individual certificates.

---

## Prerequisites

* GKE cluster with Ingress configured (using GCE Ingress or similar)
    
* A GoDaddy domain with DNS managed via GoDaddy
    
* `kubectl`, `helm`, and cluster-admin access
    
* A GoDaddy API key/token for DNS automation
    

---

## Step 1: Install cert-manager

cert-manager automates TLS certificate issuance and renewal via ACME (Let's Encrypt).

```bash
helm repo add jetstack https://charts.jetstack.io --force-update
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.17.2 \
  --set crds.enabled=true
```

*cert-manager installs its CRDs (Custom Resource Definitions) like* `Certificate`*,* `ClusterIssuer`*,* `Issuer`*, and sets up controllers to watch and reconcile these resources.*

---

## Step 2: Install GoDaddy DNS Webhook Solver

This plugin allows cert-manager to fulfill DNS-01 challenges via GoDaddy's DNS API.

```bash
helm repo add godaddy-webhook https://snowdrop.github.io/godaddy-webhook
helm install acme-webhook godaddy-webhook/godaddy-webhook \
  -n cert-manager \
  --set groupName=acme.example.com
```

*The webhook solver pod is deployed. It allows cert-manager to communicate with the GoDaddy DNS API for DNS-01 challenges.*

---

## Step 3: Store GoDaddy API Token Securely

This secret is used to authenticate webhook requests to GoDaddy.

```bash
kubectl create secret generic godaddy-api-key \
  --from-literal=token=<GODADDY_API_KEY>:<GODADDY_SECRET_KEY> \
  --namespace cert-manager
```

*This secret is later referenced in the ClusterIssuer and used by the webhook to authenticate with GoDaddy's API.*

---

## Step 4: Create the ClusterIssuer

Save this as `clusterissuer.yaml`:

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: devops@example.com
    privateKeySecretRef:
      name: letsencrypt-staging
    solvers:
    - dns01:
        webhook:
          groupName: acme.example.io
          solverName: godaddy
          config:
            apiKeySecretRef:
              name: godaddy-api-key
              key: token
            production: true
            ttl: 600
```

Apply it:

```bash
kubectl apply -f clusterissuer.yaml
```

---

*cert-manager registers the ClusterIssuer and attempts to validate connectivity and the provided ACME configuration. It will not yet perform any challenges.*

### Verify ClusterIssuer

```bash
kubectl describe clusterissuer letsencrypt-staging
```

Ensure you see: `Type: Ready` and `Status: True`

---

## Step 5: Request a Wildcard Certificate

Save the following as `wildcard-certificate.yaml`:

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: example-staging-wildcard-cert
  namespace: example
spec:
  secretName: example-staging-wildcard-tls
  issuerRef:
    name: letsencrypt-staging
    kind: ClusterIssuer
  commonName: '*.staging.yourdomain.com'
  dnsNames:
  - '*.staging.yourdomain.com'
  - 'staging.yourdomain.com'
```

Apply it:

```bash
kubectl apply -f wildcard-certificate.yaml
```

***What happens:*** *cert-manager triggers the ACME flow. It creates a* `Challenge` *and a* `Order` *resource. The GoDaddy webhook adds a TXT record to verify domain ownership. Once validated, the certificate is requested and stored as a Kubernetes TLS secret.*

---

### Verify Certificate Status

```bash
kubectl describe certificate example-staging-wildcard-cert -n example
```

Check for: `Status: Ready=True`

---

## Step 6: Add DNS A Record(s) in GoDaddy

To ensure your domain and subdomains point to your GKE Ingress:

### Option A: Wildcard record (recommended)

```bash
*.yourdomain.com     A     <GKE LoadBalancer IP>
```

### Option B: Specific record (e.g., for `example.yourdomain.com`)

```bash
example.yourdomain.com     A     <GKE LoadBalancer IP>
```

*You can add multiple such DNS records depending on your application structure. These entries must be configured in GoDaddy's DNS dashboard. This step is not required for certificate issuance but is essential to serve traffic via Ingress.*

## Step 7: Use TLS in Ingress Resource

Update your GKE Ingress with wildcard TLS:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  namespace: example
  annotations:
    kubernetes.io/ingress.class: "gce"
    kubernetes.io/ingress.global-static-ip-name: "example-lb-ip"
    kubernetes.io/ingress.allow-http: "true"
    networking.gke.io/v1beta1.FrontendConfig: "http-redirect"
spec:
  tls:
    - hosts:
        - '*.yourdomain.com'
      secretName: example-wildcard-tls
  rules:
  - host: '*.yourdomain.com'
    http:
      paths:
        - path: /*
          pathType: ImplementationSpecific
          backend:
            service:
              name: frontend-lb
              port:
                number: 80
```

---

## 🔒 Debugging Secrets (Avoid in Prod)

```bash
kubectl get secret letsencrypt-staging -n cert-manager -o jsonpath='{.data.tls\.key}' | base64 -d
kubectl get secret example-staging-wildcard-tls -n example -o jsonpath='{.data.tls\.crt}' | base64 -d
kubectl get secret example-staging-wildcard-tls -n example -o jsonpath='{.data.tls\.key}' | base64 -d
```

* `letsencrypt-staging`: your ACME account key
    
* `example-staging-wildcard-tls`: your actual TLS cert + key
    

**NOTE**: Never use ACME account keys in your services.

---

## You're Done!

You can now serve HTTPS traffic on any subdomain under `staging.yourdomain.com` securely, backed by Let's Encrypt certificates managed automatically by cert-manager.

This setup ensures:

* Zero-downtime TLS renewal
    
* Wildcard coverage
    
* Secure integration with GoDaddy
    

---

## Resources

* cert-manager: [https://cert-manager.io/](https://cert-manager.io/)
    
* Let's Encrypt ACME protocol: [https://letsencrypt.org/docs/acme-protocol/](https://letsencrypt.org/docs/acme-protocol/)
    
* GoDaddy API: [https://developer.godaddy.com/](https://developer.godaddy.com/)
    
* GKE Ingress: [https://cloud.google.com/kubernetes-engine/docs/how-to/load-balance-ingress](https://cloud.google.com/kubernetes-engine/docs/how-to/load-balance-ingress)
    
* DNS-01 Webhook Solver Docs: [https://cert-manager.io/docs/configuration/acme/dns01/webhook/](https://cert-manager.io/docs/configuration/acme/dns01/webhook/)
    

For platform engineers looking to scale HTTPS across environments and clients, this pattern is extensible, secure, and production-proven.