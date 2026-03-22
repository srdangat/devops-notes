# Kubernetes Namespaces and Deployments

## Namespaces

Namespaces are **logical partitions** inside a single cluster. They separate resources to avoid conflicts and keep environments organized.

### Built-in Namespaces

| Namespace | Purpose |
|---|---|
| `default` | Where resources go when no namespace is specified |
| `kube-system` | Core Kubernetes internals — do not modify |
| `kube-public` | Publicly readable resources |
| `kube-node-lease` | Node heartbeat tracking |

### Why Use Custom Namespaces?
- Separate environments (dev, staging, production) within one cluster
- Apply resource quotas per namespace
- Control access per team using RBAC
- Deleting a namespace removes **everything** inside it

### Namespace Behavior
- `kubectl get pods` only shows the `default` namespace
- Use `-n <namespace>` to target a specific namespace
- Use `-A` to list resources across **all** namespaces

---

## Deployments

A Deployment tells Kubernetes: *"I want X replicas of this Pod running at all times."* The Deployment controller continuously reconciles the actual state with the desired state.

### Key Differences from a Standalone Pod

| Feature | Standalone Pod | Deployment |
|---|---|---|
| `kind` | `Pod` | `Deployment` |
| `apiVersion` | `v1` | `apps/v1` |
| Self-healing | No | Yes — recreates deleted pods |
| Scaling | Manual | `replicas` field |
| Rolling updates | Not supported | Built-in |

### Deployment Manifest Structure

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3                  # Desired number of Pods
  selector:
    matchLabels:
      app: nginx               # Links Deployment to its Pods
  template:                    # Pod blueprint
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.24
        ports:
        - containerPort: 80
```

| Field | Purpose |
|---|---|
| `replicas` | How many identical Pods to maintain |
| `selector.matchLabels` | Connects the Deployment to the Pods it manages |
| `template` | The Pod blueprint used to create each replica |

---

## Self-Healing

When a Pod managed by a Deployment is deleted:
- Kubernetes detects the replica count has dropped
- A new Pod is created automatically within seconds
- The new Pod gets a **different name** but the same Deployment prefix

When a **standalone Pod** is deleted:
- It is gone permanently — no controller recreates it

---

## Scaling

### Imperative (command)
```bash
kubectl scale deployment nginx-deployment --replicas=5 -n dev
```

### Declarative (manifest)
Change `replicas:` in the YAML and re-apply:
```bash
kubectl apply -f nginx-deployment.yaml
```

When scaling **down**, Kubernetes terminates excess Pods automatically to match the desired count.

---

## Rolling Updates

Kubernetes replaces Pods **one by one** — old Pods are terminated only after new ones are healthy. This ensures **zero downtime**.

```bash
# Update the image
kubectl set image deployment/nginx-deployment nginx=nginx:1.25 -n dev

# Watch the rollout
kubectl rollout status deployment/nginx-deployment -n dev

# View rollout history
kubectl rollout history deployment/nginx-deployment -n dev
```

---

## Rollbacks

Kubernetes keeps a history of previous ReplicaSets. A rollback recreates Pods from the previous revision while removing the current ones.

```bash
# Roll back to the previous version
kubectl rollout undo deployment/nginx-deployment -n dev

# Verify the image version after rollback
kubectl describe deployment nginx-deployment -n dev | grep Image
```

---

## Deployment Output Columns

| Column | Meaning |
|---|---|
| `READY` | Pods ready to serve traffic (ready/desired) |
| `UP-TO-DATE` | Pods using the latest Deployment spec |
| `AVAILABLE` | Pods that are ready and stable |
