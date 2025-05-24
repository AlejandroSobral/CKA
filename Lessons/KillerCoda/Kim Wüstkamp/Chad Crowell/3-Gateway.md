### Create a Gateway and HTTPRoute

- 1) The Gateway API CRDs are already installed in your cluster. View them using the kubectl command line.
- 2) Install a basic Gateway resource named my-gateway in the default namespace. 
  - The gateway should be based on the gateway class nginx. You can view the gatewayClass with the command kubectl get gatewayclass
- 3) Deploy a simple web app in Kubernetes and a ClusterIP type service exposing the deployment on port 80 internally.
    - The name of the web app should be web and the image used should be nginx . Expose the container on port 80.
    - The name of the service should also be web, and the service should be exposed on port 80 targeting port 80 in the pod as well.
    - Use ONLY the kubectl command line arguments to get the web deployment and service up and running.


-------------------------------------------------


### 2 - Install basic gateway.

Having this [Docs](https://kubernetes.io/docs/concepts/services-networking/gateway/) handy:

Let's check the existing Gateway-Classes:
```bash
controlplane:~$ k get gatewayclass
NAME    CONTROLLER                                   ACCEPTED   AGE
nginx   gateway.nginx.org/nginx-gateway-controller   True       16m
```

```YAML
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: my-gateway
spec:
  gatewayClassName: nginx
  listeners:
  - name: http
    protocol: HTTP
    port: 80
```

```bash
controlplane:~$ k get gateway
NAME         CLASS   ADDRESS   PROGRAMMED   AGE
my-gateway   nginx             True         25s
```