## Pod Issue 8

```bash
controlplane:~$ curl http://localhost:8080
curl: (7) Failed to connect to localhost port 8080 after 0 ms: Couldn't connect to server'

## Service is redirecting traffic to pod which Label app=nginx-pod
controlplane:~$ k describe svc nginx-service 
Name:                     nginx-service
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=nginx-pod
Type:                     ClusterIP
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.98.64.116
IPs:                      10.98.64.116
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
Endpoints:                
Session Affinity:         None
Internal Traffic Policy:  Cluster
Events:                   <none>

## Labels is NONE
controlplane:~$ k get pods --show-labels
NAME        READY   STATUS    RESTARTS   AGE   LABELS
nginx-pod   1/1     Running   0          30s   <none>

## Label the Pod
controlplane:~$ k label pod nginx-pod app=nginx-pod
pod/nginx-pod labeled


## Start Port-Forwading and re-try
controlplane:~$ kubectl port-forward svc/nginx-service 8080:80
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80

## Now working!
controlplane:~$ curl http://localhost:8080

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

