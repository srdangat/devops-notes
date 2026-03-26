# Kubernetes StatefulSets

## What is a StatefulSet?

StatefulSet is a Kubernetes workload API object used to manage stateful applications. Unlike Deployments, StatefulSets maintain a sticky identity for each pod, making them ideal for applications that require stable network identities, stable persistent storage, and ordered deployment/scaling.

---

## When to Use StatefulSets vs Deployments

### Use StatefulSet When:
- Running databases (MySQL, PostgreSQL, MongoDB, Cassandra)
- Distributed systems (Kafka, Zookeeper, etcd)
- Applications requiring data persistence with pod identity
- Ordered deployment and startup is critical
- Each instance needs unique configuration or data

### Use Deployment When:
- Web applications (frontend, APIs)
- Stateless microservices
- Pods are interchangeable and share no state
- No persistent storage required per pod

---

## Key Differences: Deployment vs StatefulSet

| Feature | Deployment | StatefulSet |
|---------|-----------|-------------|
| Pod names | Random (e.g., `app-xyz-abc`) | Stable, ordered (e.g., `app-0`, `app-1`, `app-2`) |
| Startup order | All pods start simultaneously | Ordered: `pod-0` → `pod-1` → `pod-2` |
| Storage | Shared PVC or no storage | Each pod gets its own PVC |
| Network identity | No stable hostname | Stable DNS per pod |
| Scaling | Random pod deletion | Reverse-ordered termination |
| Use case | Stateless applications | Stateful applications |

---

## Core Components

### 1. Headless Service

A Headless Service (with `clusterIP: None`) is required for StatefulSets to provide stable network identities.

**Purpose:**
- Creates individual DNS entries for each pod
- No load balancing (unlike regular Services)
- Enables direct pod-to-pod communication
- Provides stable network identity for each pod

### 2. StatefulSet Controller

**Key Responsibilities:**
- Manages pod lifecycle with ordering guarantees
- Ensures stable pod identities
- Coordinates with volumeClaimTemplates for storage
- Enforces ordered scaling operations

### 3. VolumeClaimTemplates

**Purpose:**
- Template for creating PersistentVolumeClaims (PVCs)
- Each pod gets its own PVC based on this template
- Provides stable, persistent storage per pod

---

## How StatefulSets Work

### 1. Stable Pod Identity

**Pod Naming Convention:**
- Pods get predictable names following the pattern: `<statefulset-name>-<ordinal>`
- Ordinal (numeric index) starts at 0 and increments (0, 1, 2, 3, ...)
- Names persist across pod restarts and rescheduling
- Pod identity remains constant throughout its lifecycle

### 2. Stable Network Identity (DNS)

**DNS Pattern:**
Each pod gets a unique DNS entry following this format:
`<pod-name>.<service-name>.<namespace>.svc.cluster.local`

**How it Works:**
- Headless Service creates individual DNS A records for each pod
- DNS name resolves directly to the pod's IP address
- DNS name persists even if pod IP changes
- Other pods can reliably connect using the DNS name

### 3. Stable Persistent Storage

**PVC Naming Pattern:**
`<volumeClaimTemplate-name>-<pod-name>`

**Key Behavior:**
- Each pod gets its own dedicated PVC
- PVCs persist even if pods are deleted
- When a pod is recreated, it reconnects to the same PVC
- Data survives pod deletion and recreation
- Storage is tied to pod identity, not pod instance

---

## Deployment and Scaling Behavior

### Ordered Pod Creation

**Sequential Startup:**
- Pods are created one at a time in order
- Each pod must be Running and Ready before the next pod starts
- Order follows the numeric index : 0, 1, 2, 3, ...
- Ensures dependencies are met before proceeding

### Ordered Pod Termination

**Reverse-Order Deletion:**
- When scaling down or deleting, pods terminate in reverse order
- Highest numeric index deleted first
- Each pod fully terminates before the next deletion begins
- Ensures graceful shutdown of dependencies

### Scaling Operations

**Scale Up:**
- New pods are created sequentially with incrementing numeric indices (ordinals)
- Each waits for previous pod to be Ready

**Scale Down:**
- Pods are deleted in reverse order (highest numeric index first)
- PVCs remain intact even after scaling down
- Data preserved for potential future scale-up

---

## Storage Management

### PVC Lifecycle

**Creation:**
- PVCs are automatically created from volumeClaimTemplates when pods are created
- One PVC per pod
- PVC name is deterministic based on pod name

**Deletion Behavior:**
- StatefulSet deletion does NOT automatically delete PVCs (safety feature)
- Scaling down does NOT delete PVCs (data preservation)
- PVCs must be manually deleted if no longer needed
- Prevents accidental data loss


---

## Architecture Flow

```
┌─────────────────┐
│   StatefulSet   │
└────────┬────────┘
         │ Creates pods in order
         ├─────────────────────┬─────────────────────┐
         │                     │                     │
    ┌────▼────┐          ┌─────▼────┐          ┌─────▼────┐
    │  pod-0  │          │  pod-1   │          │  pod-2   │
    └────┬────┘          └─────┬────┘          └─────┬────┘
         │                     │                     │
         │ Mounts              │ Mounts              │ Mounts
         │                     │                     │
    ┌────▼───────┐       ┌─────▼────────┐      ┌─────▼────────┐
    │ PVC for    │       │ PVC for      │      │ PVC for      │
    │ pod-0      │       │ pod-1        │      │ pod-2        │
    └────────────┘       └──────────────┘      └──────────────┘

┌──────────────────────────────────────────────────────────┐
│              Headless Service                            │
│  - No ClusterIP (None)                                   │
│  - Creates stable DNS for each pod                       │
│  - Enables direct pod-to-pod communication               │
└──────────────────────────────────────────────────────────┘
```
---

## Common Use Cases

### Database Clusters
- MySQL with primary-replica setup
- PostgreSQL with streaming replication
- MongoDB replica sets
- Cassandra clusters

### Distributed Systems
- Kafka brokers (require stable broker IDs)
- Zookeeper ensemble (require stable server IDs)
- etcd clusters (require stable member IDs)
- Elasticsearch clusters (require stable node identities)

### Message Queues
- RabbitMQ clusters
- Redis with persistence

---