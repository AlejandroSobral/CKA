### 11 - Ingress Creation

1st part:
- There are two existing Deployments in Namespace "world" which should be made accessible via an Ingress.
- First: create ClusterIP Services for both Deployments for port 80. 
- The Services should have the same name as the Deployments.

2nd part:

The Nginx Ingress Controller has been installed.

- Create a new Ingress resource called "world" for domain name world.universe.mine . The domain points to the K8s Node IP via /etc/hosts .
- The Ingress resource should have two routes pointing to the existing Services:
  - http://world.universe.mine:30080/europe/ and http://world.universe.mine:30080/asia/

----------------------------------------

### Demo

### 1st Part:

To begin, let's check the existing deployments. Then, create ClusterIPs by using the -expose- option.


```bash
controlplane:~$ k get all -n world 
.
..
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/asia     2/2     2            2           2m48s
deployment.apps/europe   2/2     2            2           2m48s
..
.

controlplane:~$ k expose -n world deployment asia --port=80
service/asia exposed

controlplane:~$ k expose -n world deployment europe --port=80
service/europe exposed

controlplane:~$ k get all -n world
.
..
NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/asia     ClusterIP   10.108.196.66    <none>        80/TCP    87s
service/europe   ClusterIP   10.107.209.188   <none>        80/TCP    64s
..
.

```

### 2nd Part:


The Service will match the existing Labels to distribute the incoming traffic. 
Let's check the Pod's labels:

```bash
k describe -n world pod asia-69ff9c8458-6cqvk | grep Labels
Labels:           app=asia

k describe -n world pod europe-6c7579bb-82v6d | grep Labels
Labels:           app=europe
```

From [Docs](https://kubernetes.io/docs/concepts/services-networking/ingress/): "You must have an Ingress controller to satisfy an Ingress. Only creating an Ingress resource has no effect." According to the instructions, "The Nginx Ingress Controller has been installed". 
All good.

Check the existing ingressClass:
```bash
NAME    CONTROLLER             PARAMETERS   AGE
nginx   k8s.io/ingress-nginx   <none>       7m5s
```

ingress-europe.yaml
```YAML
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-europe
  namespace: world
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /europe
        pathType: Prefix
        backend:
          service:
            name: europe
            port:
              number: 80
```

ingress-asia.yaml
```YAML
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-asia
  namespace: world
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /asia
        pathType: Prefix
        backend:
          service:
            name: asia
            port:
              number: 80
```

Output:
```bash
controlplane:~$ k apply -f ingress-europe.yaml 
ingress.networking.k8s.io/ingress-europe created
controlplane:~$ curl http://world.universe.mine:30080/europe

hello, you reached EUROPE
controlplane:~$ curl http://world.universe.mine:30080/asia  
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx</center>
</body>
</html>
controlplane:~$ k apply -f as.yaml 
ingress.networking.k8s.io/ingress-asia created
controlplane:~$ curl http://world.universe.mine:30080/asia
hello, you reached ASIA
```

