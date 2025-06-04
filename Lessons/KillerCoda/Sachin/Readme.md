### Info

This is useful to have handy to compare:
- https://github.com/anishbista60/CKA-KILLERKODA-SOLUTION/tree/main
- https://www.diguage.com/post/killercoda-cka-architecture-installation-maintenance/
- https://www.diguage.com/post/killercoda-cka-services-networking/

# Pod
k run nginx-pod-cka --image=nginx


# Deployment

```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: busybox
  name: busybox
spec:
  replicas: 1
  selector:
    matchLabels:
      app: busybox
  strategy: {}
  template:
    metadata:
      labels:
        app: busybox
    spec:
      containers:
      - image: busybox:1.28
        name: busybox
        command:
        - "sleep"
        - "3600"
        resources: {}
status: {}
```


# Service

k expose pod nginx-pod-cka --name=nginx-service-cka --port=80 --targetport=80

# nslookup
k exec -it busybox-6b7bd556f9-9rv72 -- nslookup nslookup nginx-service-cka