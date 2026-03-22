# Kubernetes Services

## The Problem Services Solve

Pods are **ephemeral** — they get new IP addresses every time they restart and are created or destroyed dynamically by a Deployment. You cannot reliably hardcode a Pod IP.

**Services** solve this by providing:
- A **stable IP address** (ClusterIP) that never changes
- A **stable DNS name** for service discovery
- **Load balancing** across all healthy Pods

---

## How Services Find Pods

Services use **label selectors** to discover their Pods. Any Pod with a matching label is included as a backend:

```yaml
selector:
  app: web-app   # Routes traffic to all Pods with this label
```

This is the same label defined in the Deployment's `template.metadata.labels`.

---

## Service Types

### 1. ClusterIP (Default)

Exposes the Service on a stable internal IP — **only reachable from within the cluster**.

```yaml
spec:
  type: ClusterIP
  selector:
    app: web-app
  ports:
  - port: 80        # Port the Service listens on
    targetPort: 80  # Port on the Pod to forward to
```

**Use case:** Internal communication between microservices.

---

### 2. NodePort

Opens a port on **every node** in the cluster, making the Service reachable from outside.

```yaml
spec:
  type: NodePort
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080   # Must be in range 30000–32767
```

**Traffic flow:** `<NodeIP>:30080` → Service → Pod:80

**Use case:** Development, testing, direct node access.

---

### 3. LoadBalancer

Provisions a **cloud load balancer** (AWS, GCP, Azure) with a public IP. On local clusters, `EXTERNAL-IP` stays `<pending>` because there is no cloud provider to assign one.

```yaml
spec:
  type: LoadBalancer
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
```

**Use case:** Production traffic in cloud environments.

---

## Service Type Hierarchy

Each type **builds on** the previous:

```
LoadBalancer
  └── includes NodePort
        └── includes ClusterIP
```

A LoadBalancer Service always has both a ClusterIP and a NodePort assigned.

---

## Kubernetes DNS (CoreDNS)

Every Service gets a DNS entry automatically. The full DNS format is:

```
<service-name>.<namespace>.svc.cluster.local
```

**How it works:**
1. A Pod makes a request using the Service name
2. The request goes to **CoreDNS** for resolution
3. CoreDNS resolves the Service name → ClusterIP
4. The Service load-balances the request across healthy Pods via Endpoints
5. A Pod processes the request and responds

Within the **same namespace**, you can use the short name:
```
wget http://web-app-clusterip
```

Across **different namespaces**, use the full DNS name:
```
wget http://web-app-clusterip.default.svc.cluster.local
```

---

## Endpoints

Endpoints are the **actual Pod IPs** behind a Service. Kubernetes manages them automatically — the Endpoints list updates whenever Pods start, stop, or restart.

```bash
kubectl get endpoints
kubectl describe endpoints web-app-clusterip
```

Example Endpoints for a 3-replica Deployment:
```
10.244.0.5:80
10.244.0.6:80
10.244.0.7:80
```

---

## Service Types: Side-by-Side

| Type | Accessible From | Use Case |
|---|---|---|
| ClusterIP | Inside the cluster only | Internal microservice communication |
| NodePort | Outside via `<NodeIP>:<NodePort>` | Dev/testing, direct access |
| LoadBalancer | Outside via cloud load balancer | Production cloud deployments |

---

## Relationship: Deployment → Service → Client

```
Client
  └── Service (stable IP + DNS + load balancing)
        └── Pods (matched by label selector)
              └── Managed by Deployment
```

---

## Common Service Commands

```bash
kubectl get services                         # List all services
kubectl get services -o wide                 # Include selector and endpoints
kubectl describe service <name>              # Detailed info
kubectl get endpoints                        # Show Pod IPs behind each service
```
