# Kubernetes – Metrics Server & Horizontal Pod Autoscaler (HPA)

---

## Metrics Server

- Collects **real-time CPU and memory usage** from nodes and pods via kubelets
- Polls kubelets every **15 seconds**
- Required for `kubectl top` and HPA to function
- Data shown by `kubectl top` = **actual usage**, not requests or limits

### Common Commands

```bash
kubectl top nodes                        # Node-level CPU and memory
kubectl top pods -A                      # All pods across namespaces
kubectl top pods -A --sort-by=cpu        # Sort by CPU usage
```

> On local clusters, the `--kubelet-insecure-tls` flag may be needed — never use in production.

---

## Horizontal Pod Autoscaler (HPA)

Automatically scales the number of Pod replicas in a Deployment based on observed metrics.

### How HPA Calculates Desired Replicas

```
desiredReplicas = ceil(currentReplicas × (currentUsage / targetUsage))
```

### Prerequisites

- Metrics Server must be installed and running
- **Pods must have `resources.requests.cpu` set** — HPA uses this to calculate utilization percentages
- Without CPU requests, HPA cannot compute utilization → most common setup mistake

- `Install metrics-server:`

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

`Kind uses self-signed certs, so metrics-server can't verify kubelet TLS — patch it:`
```bash
kubectl patch deployment metrics-server -n kube-system \
  --type='json' \
  -p='[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'
```
`Wait for it to be ready:`
```bash
kubectl rollout status deployment/metrics-server -n kube-system --timeout=120s
```

---

## autoscaling/v1 vs autoscaling/v2

| Feature | `autoscaling/v1` | `autoscaling/v2` |
|---|---|---|
| CPU scaling | ✅ | ✅ |
| Memory scaling | ❌ | ✅ |
| Custom metrics | ❌ | ✅ |
| Behavior control | ❌ | ✅ |

Always prefer `autoscaling/v2` for production use.

---

## HPA Behavior Section (`autoscaling/v2`)

Controls how aggressively HPA scales up and down.

| Field | Description |
|---|---|
| `stabilizationWindowSeconds` | How long to wait before acting on scale up/down decisions |
| `policies` | Rules limiting how many pods can be added or removed per action |
| `type: Percent` | Scale by a percentage of current replicas |
| `type: Pods` | Scale by a fixed number of pods |
| `periodSeconds` | Minimum time between scaling actions |

### Typical Behavior Config

- **Scale-up:** no stabilization window (react immediately to load)
- **Scale-down:** 300 second stabilization window (avoid premature scale-down)

---

## Scaling Behavior Summary

| Direction | Default Behavior | Why |
|---|---|---|
| Scale-up | Reacts within ~1–3 minutes | Metrics need time to accumulate |
| Scale-down | 5-minute stabilization window | Prevents flapping under brief load drops |

---

- **Metrics Server** is a lightweight in-cluster component — not a full monitoring solution.
- `kubectl top` shows live usage; requests/limits are separate scheduler/runtime concepts.
- **HPA requires CPU requests** on pods to calculate percentage-based utilization.
- `autoscaling/v2` supports multiple metrics and fine-grained `behavior` tuning.
- Scale-up is fast; scale-down is intentionally slow to avoid instability.
- The `behavior` section is only configurable via YAML (`autoscaling/v2`), not the imperative `kubectl autoscale` command.
