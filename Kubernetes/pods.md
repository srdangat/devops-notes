# Kubernetes Manifests and Pods

## The Anatomy of a Kubernetes Manifest

Every Kubernetes resource is defined using a YAML manifest with four required top-level fields:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: my-app
spec:
  containers:
  - name: my-container
    image: nginx:latest
    ports:
    - containerPort: 80
```

| Field | Purpose |
|---|---|
| `apiVersion` | Which API group to use. For Pods: `v1` |
| `kind` | The resource type (Pod, Deployment, Service, etc.) |
| `metadata` | Identity of your resource — `name` is required; `labels` are key-value pairs for organization |
| `spec` | The desired state — which containers to run, images, ports, commands, etc. |

---

## Declarative vs Imperative

### Declarative (`kubectl apply -f`)
- Uses a YAML file to define the desired state
- Versionable, repeatable, preferred for production
- Example: `kubectl apply -f nginx-pod.yaml`

### Imperative (`kubectl run`)
- Creates resources immediately via a command
- Quick, good for testing, not stored as a file
- Example: `kubectl run redis-pod --image=redis:latest`

**Tip:** Use `--dry-run=client -o yaml` to scaffold a manifest without creating anything:
```bash
kubectl run test-pod --image=nginx --dry-run=client -o yaml
```

---

## Pod Lifecycle

A standalone Pod has **no controller** watching over it. If deleted, it is gone permanently. This is why production workloads use **Deployments** instead of bare Pods.

**States:**
- `Pending` — Pod accepted but container not yet started
- `Running` — Container is running
- `CrashLoopBackOff` — Container keeps crashing (often missing a long-lived command)

---

## Labels

Labels are key-value pairs attached to resources. They are used for **organization and selection**.

```bash
kubectl get pods --show-labels          # Show all labels
kubectl get pods -l app=nginx           # Filter by label
kubectl label pod nginx-pod env=prod    # Add a label
kubectl label pod nginx-pod env-        # Remove a label (trailing dash)
```

Labels connect resources — for example, a Service uses a label selector to find which Pods to route traffic to.

---

## Validation Before Applying

```bash
# Client-side validation (no cluster needed)
kubectl apply -f pod.yaml --dry-run=client

# Server-side validation (checks against the cluster's API)
kubectl apply -f pod.yaml --dry-run=server
```

---

## Useful Pod Commands

```bash
kubectl get pods                        # List pods in default namespace
kubectl get pods -o wide                # Include node and IP info
kubectl describe pod <name>             # Detailed info: events, conditions, volumes
kubectl logs <pod-name>                 # Read container logs
kubectl exec -it <pod-name> -- /bin/bash  # Shell into the container
kubectl delete pod <name>               # Delete a pod
kubectl delete -f pod.yaml              # Delete using manifest file
```

---

## The `command` Field

Without a long-running command, a container exits immediately and the Pod enters `CrashLoopBackOff`. Always specify a command for containers that don't run a persistent server:

```yaml
command: ["sh", "-c", "echo Hello && sleep 3600"]
```

---

## Extra Metadata Kubernetes Adds Automatically

When Kubernetes creates a resource, it adds fields not present in hand-written manifests:
- `uid`, `resourceVersion`, `creationTimestamp`
- `namespace`, `annotations`
- `imagePullPolicy`, `terminationMessagePath`
- `dnsPolicy`, `restartPolicy`, `tolerations`
- `serviceAccount`, `schedulerName`, `nodeName`
