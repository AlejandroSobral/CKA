## Pod Initializaton

How to initialize something before the application itself gets started.


After applying the manifest, and running:

```bash
kubectl get pods
```

The expected output would look like:
```bash
NAME        READY   STATUS     RESTARTS   AGE
init-demo   0/1     Init:0/1   0          11s
```

Where the "Init:0/1" indicates that the init container is ongoing. Once it gets finished, it will start the next phase:

```bash
NAME        READY   STATUS          RESTARTS    AGE
init-demo   1/1     PodInitializing   0          1m13s
```
Finally:

```bash
NAME        READY   STATUS    RESTARTS   AGE
init-demo   1/1     Running   0          2m43s
```
