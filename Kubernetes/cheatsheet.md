# Kubernetes (K8s) Cheat Sheet
> Quick Reference for kubectl, Resources, Configs & Troubleshooting

---

## Cluster & Context

| Command | Description |
|---------|-------------|
| `kubectl version` | Client & server version |
| `kubectl cluster-info` | Cluster endpoint info |
| `kubectl config get-contexts` | List all contexts |
| `kubectl config current-context` | Show active context |
| `kubectl config set-context --current --namespace=<ns>` | Set default namespace |
| `kubectl get nodes` | List all nodes |
| `kubectl get nodes -o wide` | Nodes with IP, OS, roles |
| `kubectl describe node <n>` | Node detail & capacity |
| `kubectl top nodes` | Node CPU/memory usage |

---

## Namespaces

| Command | Description |
|---------|-------------|
| `kubectl get namespaces` | List namespaces |
| `kubectl create namespace <n>` | Create namespace |
| `kubectl delete namespace <n>` | Delete namespace |
| `kubectl get all -n <ns>` | All resources in namespace |
| `-n <ns>` / `--namespace=<ns>` | Target specific namespace |
| `--all-namespaces` / `-A` | Across all namespaces |

---

## Pods

| Command | Description |
|---------|-------------|
| `kubectl get pods` | List pods (current ns) |
| `kubectl get pods -A` | List pods (all ns) |
| `kubectl get pods -o wide` | Pods with node & IP |
| `kubectl get pods -w` | Watch pods live |
| `kubectl describe pod <n>` | Full pod detail & events |
| `kubectl logs <pod>` | Container logs |
| `kubectl logs <pod> -c <container>` | Logs from specific container |
| `kubectl logs <pod> --previous` | Logs from crashed container |
| `kubectl logs -f <pod>` | Follow/stream logs |
| `kubectl exec -it <pod> -- /bin/sh` | Shell into container |
| `kubectl exec -it <pod> -c <c> -- bash` | Shell into specific container |
| `kubectl delete pod <n>` | Delete pod (recreated if managed) |
| `kubectl delete pod <n> --force` | Force delete stuck pod |
| `kubectl run <n> --image=<img>` | Run a quick pod |
| `kubectl run -it --rm debug --image=busybox -- sh` | Ephemeral debug pod |
| `kubectl get pod <n> -o yaml` | Pod spec as YAML |
| `kubectl top pods` | Pod CPU/memory usage |

---

## Deployments

| Command | Description |
|---------|-------------|
| `kubectl get deployments` | List deployments |
| `kubectl describe deployment <n>` | Deployment detail |
| `kubectl create deployment <n> --image=<img>` | Create deployment |
| `kubectl scale deployment <n> --replicas=3` | Scale replicas |
| `kubectl set image deployment/<n> <c>=<img>:v2` | Update image (rolling) |
| `kubectl rollout status deployment/<n>` | Watch rollout progress |
| `kubectl rollout history deployment/<n>` | Rollout history |
| `kubectl rollout undo deployment/<n>` | Rollback to previous |
| `kubectl rollout undo deployment/<n> --to-revision=2` | Rollback to revision N |
| `kubectl rollout pause deployment/<n>` | Pause rolling update |
| `kubectl rollout resume deployment/<n>` | Resume rolling update |
| `kubectl edit deployment <n>` | Edit deployment live |
| `kubectl delete deployment <n>` | Delete deployment |
| `kubectl get replicasets` | List ReplicaSets |

---

## Services & Networking

| Command | Description |
|---------|-------------|
| `kubectl get services` | List services |
| `kubectl get svc -A` | Services in all namespaces |
| `kubectl describe svc <n>` | Service detail & endpoints |
| `kubectl expose deployment <n> --port=80 --type=ClusterIP` | Expose as ClusterIP |
| `kubectl expose deployment <n> --port=80 --type=NodePort` | Expose as NodePort |
| `kubectl expose deployment <n> --port=80 --type=LoadBalancer` | Expose as LoadBalancer |
| `kubectl port-forward pod/<n> 8080:80` | Local port forward to pod |
| `kubectl port-forward svc/<n> 8080:80` | Local port forward to service |
| `kubectl get endpoints <svc>` | View service endpoints |


---

## ConfigMaps & Secrets

| Command | Description |
|---------|-------------|
| `kubectl get configmaps` | List ConfigMaps |
| `kubectl describe configmap <n>` | ConfigMap contents |
| `kubectl create configmap <n> --from-literal=k=v` | Create from literal |
| `kubectl create configmap <n> --from-file=<file>` | Create from file |
| `kubectl get secrets` | List secrets |
| `kubectl describe secret <n>` | Secret metadata (values hidden) |
| `kubectl get secret <n> -o jsonpath='{.data.<key>}'` | Get secret value (base64) |
| `kubectl get secret <n> -o jsonpath=... \| base64 -d` | Decode secret value |
| `kubectl create secret generic <n> --from-literal=k=v` | Create generic secret |
| `kubectl delete secret <n>` | Delete secret |

---

## Storage

| Command | Description |
|---------|-------------|
| `kubectl get pv` | List PersistentVolumes |
| `kubectl get pvc` | List PersistentVolumeClaims |
| `kubectl describe pvc <n>` | PVC detail & binding |
| `kubectl get storageclass` | List StorageClasses |
| `kubectl delete pvc <n>` | Delete PVC (may delete PV) |

---

## Apply & Manage Manifests

| Command | Description |
|---------|-------------|
| `kubectl apply -f <file.yaml>` | Apply manifest (create/update) |
| `kubectl apply -f <directory>/` | Apply all YAMLs in dir |
| `kubectl delete -f <file.yaml>` | Delete resources from manifest |
| `kubectl create -f <file.yaml>` | Create only (fails if exists) |
---

## Output, Filtering & Labels

| Flag / Command | Description |
|----------------|-------------|
| `-o yaml` | Output as YAML |
| `-o wide` | Extra columns (node, IP) |
| `-l app=myapp` | Filter by label selector |
| `-l 'env in (prod,staging)'` | Multi-value label filter |
| `kubectl label pod <n> env=prod` | Add/update label on pod |
| `kubectl get all -l app=myapp` | All resources with label |

---

## Troubleshooting

| Command | Description |
|---------|-------------|
| `kubectl describe pod <n>` | Events, state, resources |
| `kubectl logs <pod> --previous` | Logs from last crashed container |
| `kubectl get events -n <ns>` | Events in namespace |
| `kubectl exec -it <pod> -- nslookup <svc>` | DNS lookup inside pod |
| `kubectl exec -it <pod> -- curl <svc>:<port>` | HTTP test inside pod |

---

## Resource Short Names

| Short | Full Resource | Short | Full Resource |
|-------|--------------|-------|--------------|
| `po` | pods | `deploy` | deployments |
| `svc` | services | `ds` | daemonsets |
| `ns` | namespaces | `sts` | statefulsets |
| `no` | nodes | `rs` | replicasets |
| `cm` | configmaps | `pv` | persistentvolumes |
| `sa` | serviceaccounts | `pvc` | persistentvolumeclaims |
| `ing` | ingresses | `sc` | storageclasses |
| `ep` | endpoints | `ev` | events |

---

## Pro Tips

```bash
# Alias kubectl
alias k=kubectl

# Generate YAML without applying
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml

# Decode a secret value
kubectl get secret <n> -o jsonpath='{.data.password}' | base64 -d

# Explore API fields without docs
kubectl explain pod.spec.containers

# Watch a rollout finish
kubectl rollout status deployment/<n> -w

# Check your RBAC permissions
kubectl auth can-i --list
```
---