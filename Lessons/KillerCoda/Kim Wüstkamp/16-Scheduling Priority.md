## 16 - Scheduling Priority

1st Part:
- Find the Pod with the highest priority in Namespace management and delete it.

2nd Part:

- In Namespace lion there is one existing Pod which requests 1Gi of memory resources.
- That Pod has a specific priority because of its PriorityClass.
- Create new Pod named important of image nginx:1.21.6-alpine in the same Namespace. It should request 1Gi memory resources.
- Assign a higher priority to the new Pod so it's scheduled instead of the existing one.
- Both Pods won't fit in the cluster.



----------------------------------------------

### Demo

### 1st Part
Having this [Documentation](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-priority-preemption/) handy, let's proceed with the scenario.
I'll first get the Priority numbers for each pod; and eliminate it.

```bash
controlplane:~$ k get pods -n management 
NAME       READY   STATUS    RESTARTS   AGE
runner     1/1     Running   0          19m
sprinter   1/1     Running   0          19m

controlplane:~$ k describe pods runner -n management | grep Priority
Priority:             200000000
Priority Class Name:  level2

controlplane:~$ k describe pods sprinter -n management | grep Priority
Priority:             300000000
Priority Class Name:  level3

controlplane:~$ k -n management delete pod sprinter 
pod "sprinter" deleted
```
&nbsp;

### 2nd Part

```bash

k get pods -n lion 
NAME       READY   STATUS    RESTARTS   AGE
4d37006c   1/1     Running   0          116s

controlplane:~$ k describe pods 4d37006c -n lion | grep Priority
Priority:             200000000
Priority Class Name:  level2

## Create the Priority Class to assign it into the Pod
kubectl create priorityclass high-priority --value=200000001 --description="high priority"
priorityclass.scheduling.k8s.io/high-priority created
```
&nbsp;

Apply the Pod YAML:
```YAML
apiVersion: v1
kind: Pod
metadata:
  name: important
  namespace: lion
spec:
  containers:
  - name: nginx
    image: nginx:1.21.6-alpine
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "1Gi"
  priorityClassName: high-priority
```
&nbsp;

Verify the existing Pods:

```bash
controlplane:~$ k get pods -n lion 
NAME        READY   STATUS    RESTARTS   AGE
important   1/1     Running   0          4s
```
&nbsp;

And that's it!

