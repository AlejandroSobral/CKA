## Scheduling Pod Affinity


- There is a Pod YAML provided at /root/hobby.yaml .
- That Pod should be *preferred* to be only scheduled on Nodes where Pods with label level=restricted are running.
- For the topologyKey use kubernetes.io/hostname .
- There are no taints on any Nodes which means no tolerations are needed
  
--------------------------------------------------------------------------

### Demo

Original YAML:
```YAML
apiVersion: v1
kind: Pod
metadata:
  labels:
    level: hobby
  name: hobby-project

  containers:
  - image: nginx:alpine
    name: c
```
&nbsp;


Modified version YAML after checking [Documentation](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/):

hobby.yaml
```YAML
apiVersion: v1
kind: Pod
metadata:
  labels:
    level: hobby
  name: hobby-project
spec:
  affinity:
    podAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 50
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: level
              operator: In
              values:
              - restricted
          topologyKey: kubernetes.io/hostname
  containers:
  - name: c
    image: nginx:alpine
```

&nbsp;

Output:
```bash
controlplane:~$ k get pods -o wide
NAME         READY   STATUS    RESTARTS   AGE     IP            NODE     NOMINATED NODE   READINESS GATES
restricted   1/1     Running   0          3m17s   192.168.1.4   node01   <none>           <none>

controlplane:~$ k get nodes
NAME           STATUS   ROLES           AGE    VERSION
controlplane   Ready    control-plane   2d9h   v1.32.1
node01         Ready    <none>          2d9h   v1.32.1

controlplane:~$ k apply -f hobby.yaml 
pod/hobby-project created

controlplane:~$ k get pods -o wide --show-labels
NAME            READY   STATUS    RESTARTS   AGE    IP            NODE     NOMINATED NODE   READINESS GATES   LABELS
hobby-project   1/1     Running   0          26s    192.168.1.5   node01   <none>           <none>            level=hobby
restricted      1/1     Running   0          4m1s   192.168.1.4   node01   <none>           <none>            level=restricted
```

&nbsp;

And that's it!.