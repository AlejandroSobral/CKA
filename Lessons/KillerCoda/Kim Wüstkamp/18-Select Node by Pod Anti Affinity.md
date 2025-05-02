
## Select Node by Pod Anti Affinity

- There is a Pod YAML provided at /root/hobby.yaml.
- That Pod should be required to be only scheduled on Nodes where no Pods with label level=restricted are running.
- For the topologyKey use kubernetes.io/hostname.
- There are no taints on any Nodes which means no tolerations are needed.

-------------------------------------

### Demo

Original hobby.yaml file
```YAML
apiVersion: v1
kind: Pod
metadata:
  labels:
    level: hobby
  name: hobby-project
spec:
  containers:
  - image: nginx:alpine
    name: c

```

&nbsp;


Modified version YAML after checking [Documentation](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/):

```YAML
apiVersion: v1
kind: Pod
metadata:
  labels:
    level: hobby
  name: hobby-project
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
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

And that's it!
