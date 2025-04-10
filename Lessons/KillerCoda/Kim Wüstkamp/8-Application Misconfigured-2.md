### Application Misconfigured - 2

A Deployment has been imported from another Kubernetes cluster. But it's seems like the Pods are not running. Fix the Deployment so that it would work in any Kubernetes cluster.

-------------------------------------

### Demo

Let's gain some insight of the status:
```bash
controlplane:~$ k get all
NAME                                       READY   STATUS    RESTARTS   AGE
pod/management-frontend-7b897f454f-c7ndb   0/1     Pending   0          43s
pod/management-frontend-7b897f454f-dzql4   0/1     Pending   0          43s
pod/management-frontend-7b897f454f-prmnc   0/1     Pending   0          43s
pod/management-frontend-7b897f454f-qk4jn   0/1     Pending   0          43s
pod/management-frontend-7b897f454f-tbq8k   0/1     Pending   0          43s
```
&nbsp;

As per checking the details of the Pod, the most important line is "Node" and "Status".
```bash
controlplane:~$ k describe pods management-frontend-7b897f454f-qb8b8 
Name:             management-frontend-7b897f454f-qb8b8
Namespace:        default
Priority:         0
Service Account:  default
Node:             staging-node1/
Labels:           app=management-frontend
                  pod-template-hash=7b897f454f
Annotations:      <none>
Status:           Pending
IP:               
IPs:              <none>
Controlled By:    ReplicaSet/management-frontend-7b897f454f
Containers:
  nginx:
    Image:        nginx:alpine
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-8vxxp (ro)
Volumes:
  kube-api-access-8vxxp:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:                      <none>
```

&nbsp;

But that Node, does not exist!.
```bash
controlplane:~$ k get nodes
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   18d   v1.32.1
```

&nbsp;

Let's check the Pod features:

```bash
controlplane:~$ k get pods management-frontend-7b897f454f-pnfwg -o yaml | grep nodeName
  nodeName: staging-node1
```
&nbsp;

That states that the Pod will run only in a node named "staging-node1". Since there exist no Node with that name, Pods aren't able to schedule.

```bash
controlplane:~$ k edit deployments.apps management-frontend 
deployment.apps/management-frontend edited
```

<details>

<summary> Entire YAML of the deployment prior to its modification </summary>


```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2025-04-10T02:17:37Z"
  generation: 1
  name: management-frontend
  namespace: default
  resourceVersion: "5651"
  uid: 4af3c304-4882-4d8b-bbcd-4b7341317b8f
spec:
  progressDeadlineSeconds: 600
  replicas: 5
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: management-frontend
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: management-frontend
    spec:
      containers:
      - image: nginx:alpine
        imagePullPolicy: IfNotPresent
        name: nginx
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      nodeName: staging-node1
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  conditions:
  - lastTransitionTime: "2025-04-10T02:17:37Z"
    lastUpdateTime: "2025-04-10T02:17:37Z"
    message: Deployment does not have minimum availability.
    reason: MinimumReplicasUnavailable
    status: "False"
    type: Available
  - lastTransitionTime: "2025-04-10T02:17:37Z"
    lastUpdateTime: "2025-04-10T02:17:37Z"
    message: ReplicaSet "management-frontend-7b897f454f" is progressing.
    reason: ReplicaSetUpdated
    status: "True"
    type: Progressing
  observedGeneration: 1
  replicas: 5
  unavailableReplicas: 5
  updatedReplicas: 5
```

</details>

&nbsp;

Therefore, erasing the line that stated that the nodeName should be a particular one. 

It would be like this:

<details>

<summary> Entire YAML after the modification </summary>


```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2025-04-10T02:17:37Z"
  generation: 1
  name: management-frontend
  namespace: default
  resourceVersion: "5651"
  uid: 4af3c304-4882-4d8b-bbcd-4b7341317b8f
spec:
  progressDeadlineSeconds: 600
  replicas: 5
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: management-frontend
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: management-frontend
    spec:
      containers:
      - image: nginx:alpine
        imagePullPolicy: IfNotPresent
        name: nginx
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  conditions:
  - lastTransitionTime: "2025-04-10T02:17:37Z"
    lastUpdateTime: "2025-04-10T02:17:37Z"
    message: Deployment does not have minimum availability.
    reason: MinimumReplicasUnavailable
    status: "False"
    type: Available
  - lastTransitionTime: "2025-04-10T02:17:37Z"
    lastUpdateTime: "2025-04-10T02:17:37Z"
    message: ReplicaSet "management-frontend-7b897f454f" is progressing.
    reason: ReplicaSetUpdated
    status: "True"
    type: Progressing
  observedGeneration: 1
  replicas: 5
  unavailableReplicas: 5
  updatedReplicas: 5
```

</details>

Let's check the status:


```bash
controlplane:~$ k get pods
NAME                                   READY   STATUS    RESTARTS   AGE
management-frontend-6467b9fc4d-ct6qp   1/1     Running   0          68s
management-frontend-6467b9fc4d-j9gdc   1/1     Running   0          67s
management-frontend-6467b9fc4d-mjbgh   1/1     Running   0          75s
management-frontend-6467b9fc4d-tbwr5   1/1     Running   0          75s
management-frontend-6467b9fc4d-vkz8n   1/1     Running   0          74s
```

&nbsp;
That's it!.