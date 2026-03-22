# Kubernetes Architecture

## Why Kubernetes Exists

Docker allows you to build and run containers, but it focuses on running containers on a **single machine**. When applications grow and require many containers across multiple servers, managing them manually becomes complex — scaling, restarting failures, and scheduling all need automation.

**Kubernetes** solves this by acting as a **container orchestration system** that can automatically:
- Scale containers based on demand
- Restart containers when they fail
- Schedule and manage containers across a cluster of machines

> Docker runs containers. Kubernetes manages large numbers of containers across a cluster.

---

## History

- **Created by:** Google (open-sourced in 2014)
- **Inspired by:** Google's internal system called **Borg**, which managed containers at massive scale
- **Maintained by:** The Cloud Native Computing Foundation (CNCF), part of the Linux Foundation
- **Name meaning:** Greek for "helmsman" or "ship pilot" — someone who steers a ship
- **Abbreviation:** K8s (the 8 represents the eight letters between K and S)

---

## Kubernetes Architecture

![Kubernetes Architecture Diagram](k8s.png)

### Control Plane (Master Node)

| Component | Role |
|---|---|
| **API Server** | The front door to the cluster — every command goes through it |
| **etcd** | Distributed key-value store that holds all cluster state |
| **Scheduler** | Decides which node a new pod should run on |
| **Controller Manager** | Watches the cluster and ensures desired state matches reality |

### Worker Node

| Component | Role |
|---|---|
| **kubelet** | Agent on each node; talks to the API server and manages pods |
| **kube-proxy** | Manages networking rules so pods can communicate |
| **Container Runtime** | The engine that actually runs containers (e.g., containerd, CRI-O) |

---

## Request Flow: `kubectl apply -f pod.yaml`

1. `kubectl` reads the YAML file
2. Request is sent to the **API Server**
3. API Server performs **Authentication** and **Authorization**
4. If valid, the Pod object is stored in **etcd**
5. The **Scheduler** detects the unscheduled Pod and assigns it to a node
6. The **kubelet** on that node instructs the **container runtime** to start the container
7. The container starts; Pod status is updated to `Running`

---

## Failure Scenarios

**If the API Server goes down:**
- You cannot run `kubectl` commands or make changes to the cluster
- Running pods and services continue working
- No new deployments or scheduling can happen

**If a Worker Node goes down:**
- Pods on that node stop running
- Kubernetes detects the failure and reschedules pods on other healthy nodes

---

## kubeconfig

`kubeconfig` is a configuration file used by `kubectl` to connect to a Kubernetes cluster. It stores:
- Cluster connection details
- User credentials
- Contexts (named combinations of cluster + user)

**Default location:** `~/.kube/config`

---

## Key Namespaces (Built-in)

| Namespace | Purpose |
|---|---|
| `default` | Where resources go if no namespace is specified |
| `kube-system` | Core Kubernetes components (API server, scheduler, etcd, etc.) |
| `kube-public` | Publicly readable resources |
| `kube-node-lease` | Node heartbeat tracking |

---

## Common kubectl Commands

```bash
kubectl cluster-info              # Show cluster endpoint
kubectl get nodes                 # List nodes
kubectl get pods -A               # List all pods across all namespaces
kubectl get pods -n kube-system   # Pods in a specific namespace
kubectl config current-context    # Which cluster kubectl is connected to
kubectl config get-contexts       # List all available contexts
kubectl config view               # Show the full kubeconfig
```
