

## DaemonSet

[DaemonSet Doc](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)

Briefly, Daemons starts applications on each cluster node. Commonly used to start agents like kube-proxy that needs to be running on all cluster nodes. Should the DaemonSet required to be running on control-plane nodes, a toleration must be set to allow in there.

Typical uses of a DaemonSet includes <b> monitoring daemons</b> on very node.

## Commands

Get DaemonSets for default namespace (Probably at this point prompts nothing).
```bash
kubectl get ds
```
Get DaemonSets for *ALL* namespaces. (This never will prompt nothing).
```bash
kubectl get ds -A
```
Prompt current configuration out, YAML formatted.
```bash
kubectl get ds -n kube-system kube-proxy -o yaml | less
```

```bash
kubectl create deploy mydaemon --image=nginx --dry-run=client -o yaml > mydeamon.yaml
```

