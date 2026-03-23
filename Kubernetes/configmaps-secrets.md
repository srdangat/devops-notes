# Kubernetes ConfigMaps and Secrets

## Overview

| Feature | ConfigMap | Secret |
|---|---|---|
| Purpose | Non-sensitive config data | Sensitive data (passwords, tokens) |
| Storage | Plain text | Base64-encoded |
| Use cases | URLs, feature flags, config files | DB credentials, API keys, certs |

---

## ConfigMaps

### Create from Literals
```bash
kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --from-literal=APP_DEBUG=false \
  --from-literal=APP_PORT=8080
```

### Create from a File
```bash
kubectl create configmap nginx-config --from-file=default.conf=<your-file>
```
The key name (`default.conf`) becomes the filename when mounted into a Pod.

### Inspect
```bash
kubectl describe configmap app-config
kubectl get configmap app-config -o yaml
```

---

## Secrets

### Create from Literals
```bash
kubectl create secret generic db-credentials \
  --from-literal=DB_USER=admin \
  --from-literal=DB_PASSWORD=s3cureP@ssw0rd
```

### Inspect & Decode
```bash
kubectl get secret db-credentials -o yaml
echo '<base64-value>' | base64 --decode
```

> **Note:** base64 is encoding, not encryption. Anyone with cluster access can decode Secrets. Real security comes from RBAC, tmpfs node storage, and optional encryption at rest.

---

## Using ConfigMaps & Secrets in Pods

### Inject as Environment Variables

**All keys from a ConfigMap (`envFrom`):**
```yaml
envFrom:
  - configMapRef:
      name: app-config
```

**Single key from a Secret (`secretKeyRef`):**
```yaml
env:
  - name: DB_USER
    valueFrom:
      secretKeyRef:
        name: db-credentials
        key: DB_USER
```

### Mount as a Volume

**ConfigMap volume mount (e.g., Nginx config):**
```yaml
volumes:
  - name: nginx-config
    configMap:
      name: nginx-config
volumeMounts:
  - name: nginx-config
    mountPath: /etc/nginx/conf.d
```

**Secret volume mount (read-only):**
```yaml
volumes:
  - name: db-creds
    secret:
      secretName: db-credentials
volumeMounts:
  - name: db-creds
    mountPath: /etc/db-credentials
    readOnly: true
```

> When mounted as a volume, each key becomes a **file** and its value is **decoded plaintext**.

---

## Update Propagation

| Injection Method | Auto-updates on ConfigMap change? |
|---|---|
| Volume mount | Yes — after ~30–60 seconds |
| Environment variable | No — requires Pod restart |

### Live update example
```bash
kubectl patch configmap live-config --type merge -p '{"data":{"message":"world"}}'
```
Volume-mounted files reflect the change automatically; env vars do not.

---

- Use **ConfigMaps** for non-sensitive config (env settings, config files, URLs).
- Use **Secrets** for sensitive data (passwords, tokens, certs).
- **base64 ≠ encryption** — it's just safe YAML encoding; treat Secrets as sensitive regardless.
- **Volume mounts** auto-propagate updates; **env vars** are fixed at Pod startup.
- Secure Secrets properly with RBAC and consider enabling **encryption at rest** in the cluster.
