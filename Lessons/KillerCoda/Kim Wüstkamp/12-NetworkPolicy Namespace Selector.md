### 12 - NetworkPolicy Namespace Selector:

- There are existing Pods in Namespace space1 and space2 .
- We need a new NetworkPolicy named np that restricts all Pods in Namespace space1 to only have outgoing traffic to Pods in Namespace space2. Incoming traffic not affected.
- We also need a new NetworkPolicy named np that restricts all Pods in Namespace space2 to only have incoming traffic from Pods in Namespace space1 . Outgoing traffic not affected.
- The NetworkPolicies should still allow outgoing DNS traffic on port 53 TCP and UDP.
  
--------------------------------------------------------------------------------------------

### Demo

Let's get the picture of the situation:

```bash
controlplane:~$ k get namespaces 
NAME                 STATUS   AGE
..
space1               Active   81s
space2               Active   81s
..

controlplane:~$ k get pods -n space1
NAME     READY   STATUS    RESTARTS   AGE
app1-0   1/1     Running   0          88s
controlplane:~$ k get pods -n space2
NAME              READY   STATUS    RESTARTS   AGE
microservice1-0   1/1     Running   0          91s
microservice2-0   1/1     Running   0          91s
```
There exist two namespaces; space1 & space2.
Each have a couple of pods. Considering the given tasks instructions, two Network Policies should be set in place. A NP per each namespace.
It will have an egress and ingress policies.

&nbsp;

Since the Egress Traffic is being modified, the TPC/UDP outbound traffic should be set appropietly.
space1-np.yaml
```YAML
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np
  namespace: space1
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
            kubernetes.io/metadata.name: space2
  - ports:
    - port: 53
      protocol: TCP
    - port: 53
      protocol: UDP
```
&nbsp;

For Space2, outbout policy is not address, therefore, no further settins are required.
space2-np.yaml
```YAML
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np
  namespace: space2
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
            kubernetes.io/metadata.name: space1
```