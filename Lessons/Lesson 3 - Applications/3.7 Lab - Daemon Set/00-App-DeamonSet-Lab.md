## Commands

Run the following command to get a YAML file that emulates the described deployment.

```bash
kubectl create deployment deploydaemon --image=nginx --dry-run=client -o yaml > deploydaemon.yaml
```

The original file, Named 00 in the repo, will look like:
```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: deploydaemon
  name: deploydaemon
spec:
  replicas: 1
  selector:
    matchLabels:
      app: deploydaemon
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: deploydaemon
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```

Some modifications should be introduced, such as:
- Change kind into "Daemonset"
- Delete tags
    - replicas
    - strategy


Named 01 in the repo:
```YAML
apiVersion: apps/v1
kind: DaemonSet
metadata:
  creationTimestamp: null
  labels:
    app: deploydaemon
  name: deploydaemon
spec:
  selector:
    matchLabels:
      app: deploydaemon
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: deploydaemon
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```

```bash
kubectl apply -f deploydaemon.yaml
```
```bash
kubectl get all --selector app=deploydaemon
```

Expected output:
```bash
NAME                     READY   STATUS    RESTARTS   AGE
pod/deploydaemon-4pjvw   1/1     Running   0          41s

NAME                          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/deploydaemon   1         1         1       1            1           <none>          41s

```