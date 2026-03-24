# Kubernetes Persistent Storage

## Why Containers Need Persistent Storage

Containers are **ephemeral** — any data written inside them is lost on restart or deletion. Persistent storage keeps data outside the container lifecycle, which is essential for databases, logs, and user files.

---

## Core Concepts

### emptyDir Volume
- A temporary volume tied to the Pod's lifetime.
- Data is lost when the Pod is deleted or restarted.
- Useful for scratch space or sharing data between containers in the same Pod.

---

### PersistentVolume (PV)
A piece of storage provisioned in the cluster, either manually by an admin or automatically via a StorageClass.

Key fields:
- `capacity` — how much storage (e.g., `1Gi`)
- `accessModes` — how the volume can be mounted
- `persistentVolumeReclaimPolicy` — what happens to the PV when the PVC is deleted
- `storageClassName` — links to a StorageClass for dynamic provisioning

---

### PersistentVolumeClaim (PVC)
A request for storage by a user/workload.

Key fields:
- `resources.requests.storage` — how much storage is needed
- `accessModes` — required access mode
- `storageClassName` — optional, targets a specific StorageClass

**Binding:** Kubernetes automatically binds a PVC to a PV that satisfies its capacity and access mode requirements.

**Relationship:**
```
PVC (request) → bound to PV (storage) → mounted in Pod
```

---

## Access Modes

| Mode | Short | Description | Common Use Case |
|------|-------|-------------|-----------------|
| ReadWriteOnce | RWO | Read-write by a **single node** | Most databases |
| ReadOnlyMany  | ROX | Read-only by **multiple nodes** | Config files, shared static data |
| ReadWriteMany | RWX | Read-write by **multiple nodes** | Shared storage for multiple Pods |

---

## Reclaim Policies

| Policy | Effect After PVC Deletion |
|--------|--------------------------|
| `Retain` | PV is kept; data persists. Must be manually cleaned up. |
| `Delete` | PV and underlying storage are automatically deleted. |
| `Recycle` | Deprecated. Basic scrub and made available again. |

---

## Static vs Dynamic Provisioning

### Static Provisioning
- Admin **manually creates** the PV before a PVC exists.
- PVC is then created and Kubernetes binds it to the matching PV.
- More control, but requires upfront admin work.

### Dynamic Provisioning
- No PV is created manually.
- A **StorageClass** defines a provisioner.
- When a PVC is created with a `storageClassName`, Kubernetes **automatically provisions** a matching PV.
- Saves time and scales well.

---

## StorageClass

Defines *how* storage is dynamically provisioned.

Key fields:
- `provisioner` — the plugin that creates the storage (e.g., `rancher.io/local-path`, `kubernetes.io/aws-ebs`)
- `reclaimPolicy` — inherited by PVs created from this class (`Delete` or `Retain`)
- `volumeBindingMode`:
  - `Immediate` — PV is provisioned as soon as PVC is created
  - `WaitForFirstConsumer` — PV is provisioned only when a Pod schedules and uses the PVC

The **default StorageClass** is used automatically when a PVC does not specify `storageClassName`.

---

## PV Lifecycle

```
Available → Bound → Released → (Deleted or Retained)
```

| State | Meaning |
|-------|---------|
| `Available` | PV exists and is not yet claimed |
| `Bound` | PV is matched to a PVC |
| `Released` | PVC was deleted; PV still exists (Retain policy) |
| `Failed` | Automatic reclamation failed |

---

## hostPath Volumes

- Maps a directory on the **host node** into the Pod.
- Simple and useful for local development/learning.
- **Not suitable for production** — data is node-local and lost if the Pod moves to another node.

---

## Quick Reference Summary

| Concept | Purpose |
|---------|---------|
| `emptyDir` | Temporary, Pod-scoped storage |
| `hostPath` | Node-local storage (dev/learning only) |
| `PersistentVolume` | Cluster-level storage resource |
| `PersistentVolumeClaim` | Pod's request for storage |
| `StorageClass` | Template for dynamic PV provisioning |
| `Retain` policy | Admin must manually clean up after PVC delete |
| `Delete` policy | Storage auto-deleted when PVC is deleted |
| `WaitForFirstConsumer` | PV only created when Pod is actually scheduled |
