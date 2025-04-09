### Application Misconfigured

There is a Deployment in Namespace application1 which seems to have issues and is not getting ready.

Fix it by only editing the Deployment itself and no other resources.

-----------------------------------------

### Demo

Let's gain insight of the deployment status:

```bash
controlplane:~$ k get all -n application1
NAME                       READY   STATUS                       RESTARTS   AGE
pod/api-6768cbb9cc-2v7w7   0/1     CreateContainerConfigError   0          34s
pod/api-6768cbb9cc-f75r9   0/1     CreateContainerConfigError   0          34s
pod/api-6768cbb9cc-z8fjm   0/1     CreateContainerConfigError   0          34s

NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/api   0/3     3            0           34s
```

&nbsp;

Considering that Pods are *not* running, let's investigate further:

```bash
k describe -n application1 pod api-6768cbb9cc-s7pcw 
Events:
  Type     Reason     Age               From               Message
  ----     ------     ----              ----               -------
  ..
  Warning  Failed     5s (x5 over 44s)  kubelet            Error: configmap "category" not found
  ..
```

```bash
journalctl | grep pod
Apr 09 21:46:36 controlplane kubelet[1599]: E0409 21:46:36.213382    1599 pod_workers.go:1301] "Error syncing pod, skipping" err="failed to \"StartContainer\" for \"httpd\" with CreateContainerConfigError: \"configmap \\\"category\\\" not found\"" pod="application1/api-6768cbb9cc-zc5ht" podUID="7fa2da13-b96c-4017-bad1-0ae34dc071b9"
```

&nbsp;

As per the logs, there exist some problem with the configmaps configuration of the deployment.


```bash
controlplane:~$ k get -n application1 deployment api -o yaml
```

&nbsp;

<details>

<summary>Complete YAML output will look like:</summary>

```YAML
    matchLabels:
      app: api
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: api
    spec:
      containers:
      - env:
        - name: CATEGORY
          valueFrom:
            configMapKeyRef:
              key: category
              name: category
        image: httpd:2.4.52-alpine
        imagePullPolicy: IfNotPresent
        name: httpd
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
  - lastTransitionTime: "2025-04-09T21:39:24Z"
    lastUpdateTime: "2025-04-09T21:39:24Z"
    message: Deployment does not have minimum availability.
    reason: MinimumReplicasUnavailable
    status: "False"
    type: Available
  - lastTransitionTime: "2025-04-09T21:39:23Z"
    lastUpdateTime: "2025-04-09T21:39:24Z"
    message: ReplicaSet "api-6768cbb9cc" is progressing.
    reason: ReplicaSetUpdated
    status: "True"
    type: Progressing
  observedGeneration: 1
  replicas: 3
  unavailableReplicas: 3
  updatedReplicas: 3
```

</details>

&nbsp;

However, the issue appears to be around:

```YAML
spec:
  containers:
  - env:
    - name: CATEGORY
      valueFrom:
        configMapKeyRef:
          key: category
          name: category
```

&nbsp;

Checking the existing config maps, none matches the name "category". This make sense since the logs show something similar.

```bash
controlplane:~$ k get configmap -n application1
NAME                 DATA   AGE
configmap-category   1      9m48s
kube-root-ca.crt     1      9m49s
```

Let's edit the deployment:

```bash
controlplane:~$ k edit deployments.apps api -n application1
deployment.apps/api edited
```

```YAML
spec:
  containers:
  - env:
    - name: CATEGORY
      valueFrom:
        configMapKeyRef:
          key: category
          name: configmap-category
```

&nbsp;

Check again the Deployment status:
```bash
controlplane:~$ k get all -n application1
NAME                       READY   STATUS    RESTARTS   AGE
pod/api-77bf88fd44-hmfr2   1/1     Running   0          42s
pod/api-77bf88fd44-sk2h8   1/1     Running   0          40s
pod/api-77bf88fd44-xsr2t   1/1     Running   0          44s

NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/api   3/3     3            3           11m
```

&nbsp;

And that's it! Up & Running.