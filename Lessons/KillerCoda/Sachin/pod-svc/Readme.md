## Pod-Svc

Console output:

```bash
# Before CURL
controlplane:~$ kubectl port-forward app-pod 80 80
Forwarding from 127.0.0.1:80 -> 80
Forwarding from [::1]:80 -> 80
Unable to listen on port 80: Listeners failed to create with the following errors: [unable to create listener: Error listen tcp4 127.0.0.1:80: bind: address already in use unable to create listener: 

#CURL
controlplane:~$ curl localhost:80
<html><body><h1>It works!</h1></body></html>


# After CURL
controlplane:~$ kubectl port-forward app-pod 80 80
Forwarding from 127.0.0.1:80 -> 80
Forwarding from [::1]:80 -> 80
Unable to listen on port 80: Listeners failed to create with the following errors: [unable to create listener: Error listen tcp4 127.0.0.1:80: bind: address already in use unable to create listener: 

```