# Helm — Kubernetes Package Manager

Helm is the package manager for Kubernetes. It bundles Kubernetes manifests (Deployments, Services, ConfigMaps, etc.) into reusable **charts**, handles templating via Go templates, and manages the full lifecycle of an application — install, upgrade, rollback, and uninstall.

One `helm install` replaces writing a Deployment, Service, and ConfigMap by hand.

---

## Three Core Concepts

| Concept | Description |
|---|---|
| **Chart** | A package of Kubernetes manifest templates — the shareable, versioned unit of Helm |
| **Release** | A specific running installation of a chart inside your cluster. Each install gets its own release name |
| **Repository** | A remote collection of charts, like a package registry (e.g. Bitnami, Artifact Hub) |

---

## Installation

To install Helm, follow the official guide:  
[Helm Installation Guide](https://helm.sh/docs/intro/install/)

---

## Managing Repositories

```bash
# Add a repository
helm repo add bitnami https://charts.bitnami.com/bitnami

# Fetch latest chart list
helm repo update

# Search within repos
helm search repo nginx
helm search repo bitnami

# List configured repos
helm repo list
```

---

## Lifecycle — Install · Upgrade · Rollback · Uninstall

```bash
# Install from a public chart
helm install my-nginx bitnami/nginx

# Install from a local chart with a values file
helm install my-app ./my-chart -f custom-values.yaml

# Upgrade (Helm applies only the diff)
helm upgrade my-nginx bitnami/nginx --set replicaCount=5

# Rollback to revision 1
helm rollback my-nginx 1
# Note: rollback creates a NEW revision — it does not overwrite history

# Uninstall (retaining history for audit)
helm uninstall my-nginx --keep-history
```

### Inspect Commands

| Command | Description |
|---|---|
| `helm list` | Lists all releases in the current namespace |
| `helm status <name>` | Shows current status (deployed, failed, etc.) of a release |
| `helm history <name>` | Shows full revision history for a release |
| `helm get manifest <name>` | Displays the rendered Kubernetes YAML for a release |
| `helm get values <name>` | Shows the values that were applied to a release |

---

## Customizing with Values

Helm merges values in this order (last wins):  
**chart defaults** → **`-f` values file(s)** → **`--set` CLI flags**

### Method 1 — Values File

```yaml
# custom-values.yaml
replicaCount: 3

service:
  type: NodePort
  port: 80

resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "250m"
    memory: "256Mi"
```

```bash
helm install my-app ./my-chart -f custom-values.yaml
```

### Method 2 — CLI Flags

```bash
helm install my-app ./my-chart \
  --set replicaCount=5 \
  --set image.tag=latest \
  --set service.type=NodePort
```

```bash
# View a chart's default values
helm show values bitnami/nginx
```

---

## Chart Directory Structure

Scaffold a new chart with `helm create my-app`:

```
my-app/
├── Chart.yaml          # Chart metadata (name, version, appVersion)
├── values.yaml         # Default configuration values
├── charts/             # Dependencies / subcharts
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── _helpers.tpl    # Reusable template helpers
│   └── ingress.yaml
└── .helmignore         # Files to exclude from the chart package
```

---

## Go Templating

Helm uses Go-based templates. Placeholders inside `{{ ... }}` are replaced with actual values at render time. Configuration is sourced from `values.yaml`, override files (`-f`), and `--set` flags.

```yaml
# templates/deployment.yaml
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Chart.Name }}
  template:
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

This allows the same chart to be reused across environments with different configurations.

---

## Development Workflow

```bash
# Lint — catch YAML / template errors before deploying
helm lint my-app

# Dry-run render — see the final YAML without deploying
helm template my-release ./my-app

# Install local chart
helm install my-release ./my-app

# Upgrade local chart
helm upgrade my-release ./my-app --set replicaCount=5
```

---

- Helm separates **configuration** (`values.yaml`) from **structure** (`templates/`) — the same chart deploys differently across environments.
- Every `helm install` or `helm upgrade` creates a new **revision**. Rollbacks also create a new revision rather than overwriting history.
- `helm template` is your best friend for debugging — always preview before deploying.
- Use `--keep-history` on uninstall if you want to retain revision history for auditing.
