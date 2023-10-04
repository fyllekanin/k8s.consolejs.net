# Cert-manager Setup

## Pre-requirements
- Setup K3S

## Install reflector
This is used to reflect secrets to all namespaces

```
kubectl -n kube-system apply -f https://github.com/emberstack/kubernetes-reflector/releases/latest/download/reflector.yaml
```

## Installing cert-manager

Go to https://github.com/cert-manager/cert-manager/releases and find the latest release

Install it
```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/<version>/cert-manager.yaml
```

### Variables to update in all snippets
{email}
{token}
{namespace}
{secret-name}
{domain}.{ext}
  
### Setup secret with cloudflare token
Token needs:
Zone.Zone:Read
Zone.DNS:Edit

```
apiVersion: v1
kind: Secret
metadata:
  name: cloudflare-api-token-secret
  namespace: cert-manager
type: Opaque
stringData:
  api-token: {token}
```

### Setup issues
```
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
  namespace: cert-manager
spec:
  acme:
    email: {e-mail}
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - dns01:
        cloudflare:
          email: {e-mail}
          apiTokenSecretRef:
            name: cloudflare-api-token-secret
            key: api-token
```

### Setup certificate
```
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: {secret-name}
  namespace: cert-manager
spec:
  secretName: {secret-name}
  issuerRef:
    name: letsencrypt-prod
  secretTemplate:
    annotations:
      reflector.v1.k8s.emberstack.com/reflection-allowed: "true"
      reflector.v1.k8s.emberstack.com/reflection-auto-enabled: "true"
  duration: 2160h # 90d
  renewBefore: 720h # 30d before SSL will expire, renew it
  dnsNames:
    - "{domain}.{ext}"
    - "*.{domain}.{ext}"
```
