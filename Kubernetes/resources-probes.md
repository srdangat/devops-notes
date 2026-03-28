# Kubernetes – Resource Requests, Limits & Probes

---

## Resource Requests vs Limits

| | Requests | Limits |
|---|---|---|
| **Used by** | Scheduler (pod placement) | Container runtime (enforcement) |
| **Purpose** | Guarantees minimum resources | Prevents exceeding defined resources |
| **Unit (CPU)** | Millicores — `100m` = 0.1 CPU | Same |
| **Unit (Memory)** | Mebibytes — `128Mi` | Same |

### QoS Classes

| Class | Condition |
|---|---|
| `Guaranteed` | Requests == Limits for all containers |
| `Burstable` | Requests < Limits (at least one container) |
| `BestEffort` | No requests or limits defined |

---

## What Happens When Limits Are Exceeded

### CPU Limit Exceeded
- Container is **throttled** (slowed down)
- Not killed — just rate-limited

### Memory Limit Exceeded
- Container is **OOMKilled** (killed immediately)
- Exit code: **137** (128 + SIGKILL)
- Kubernetes restarts the container

---

## Pending Pods — Insufficient Resources

If a Pod requests more resources than any node can provide:
- Pod stays in `Pending` state indefinitely
- Scheduler emits an event: `0/N nodes are available: Insufficient cpu, Insufficient memory`

---

## Probes

### Overview

| Probe | Purpose | Runs When | On Failure |
|---|---|---|---|
| **Startup** | Is the app done starting? | At container startup | Container restarted |
| **Liveness** | Is the app still alive? | After startup succeeds | Container restarted |
| **Readiness** | Is the app ready to serve traffic? | Throughout lifecycle | Removed from Service endpoints (NOT restarted) |

---

### Startup Probe

- Gives slow-starting containers extra time before liveness/readiness kick in
- While startup probe is running, **liveness and readiness probes are disabled**
- Budget = `periodSeconds × failureThreshold`
- Example: `periodSeconds: 5`, `failureThreshold: 12` → 60 second budget
- If the app hasn't started within the budget → container is restarted

---

### Liveness Probe

- Detects stuck or crashed containers
- Common check types: `exec` (run a command), `httpGet`, `tcpSocket`
- On failure (after `failureThreshold` consecutive failures) → container is **restarted**

---

### Readiness Probe

- Controls whether a Pod receives traffic from a Service
- On failure → Pod IP is **removed from Service endpoints**
- Container is **NOT restarted** — only traffic is cut off
- Recovers automatically once the probe passes again

---

## Probe Configuration Fields

| Field | Description |
|---|---|
| `initialDelaySeconds` | Wait before first probe |
| `periodSeconds` | How often to run the probe |
| `failureThreshold` | Consecutive failures before action |
| `successThreshold` | Consecutive successes to mark healthy |
| `timeoutSeconds` | Timeout per probe attempt |

---


- **Requests** are used at scheduling time; **limits** are enforced at runtime.
- CPU over-limit → throttled. Memory over-limit → killed (OOMKilled, exit 137).
- **Startup probe** protects slow starters; blocks liveness/readiness until it passes.
- **Liveness probe** restarts unhealthy containers.
- **Readiness probe** gates traffic without restarting containers.
- Pods with impossible resource requests stay `Pending` — the scheduler tells you why in Events.
