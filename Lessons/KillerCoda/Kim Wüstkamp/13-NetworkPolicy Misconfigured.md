### 13 - NetworkPolicy Misconfigured

- All Pods in Namespace default with label level=100x should be able to communicate with Pods with label level=100x in Namespaces level-1000, level-1001 and level-1002 .

- Fix the existing NetworkPolicy np-100x to ensure this.

------------------------------------------------------------------------------------------------------------

Get the original version:

```bash
controlplane:~$ k get networkpolicies np-100x -o yaml
```

```YAML
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  creationTimestamp: "2025-04-15T22:04:27Z"
  generation: 1
  name: np-100x
  namespace: default
  resourceVersion: "4325"
  uid: 21a57c28-4f03-4869-afd4-29431906c938
spec:
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: level-1000
      podSelector:
        matchLabels:
          level: 100x
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: level-1000
      podSelector:
        matchLabels:
          level: 100x
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: level-1002
      podSelector:
        matchLabels:
          level: 100x
  - ports:
    - port: 53
      protocol: TCP
    - port: 53
      protocol: UDP
  podSelector:
    matchLabels:
      level: 100x
  policyTypes:
  - Egress
```
&nbsp;

After confirming that no other NetworkPolicies existed in the other spaces; it is a matter of tunning the existing np-100x.

The -edited- version is very similar:


```YAML
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np-100x
  namespace: default
spec:
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: level-1000
      podSelector:
        matchLabels:
          level: 100x
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: level-1001
      podSelector:
        matchLabels:
          level: 100x
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: level-1002
      podSelector:
        matchLabels:
          level: 100x
  - ports:
    - port: 53
      protocol: TCP
    - port: 53
      protocol: UDP
  podSelector:
    matchLabels:
      level: 100x
  policyTypes:
  - Egress
```
&nbsp;

To verify, all these three should work correctly:

```bash
kubectl exec tester-0 -- curl tester.level-1000.svc.cluster.local
kubectl exec tester-0 -- curl tester.level-1001.svc.cluster.local
kubectl exec tester-0 -- curl tester.level-1002.svc.cluster.local
```