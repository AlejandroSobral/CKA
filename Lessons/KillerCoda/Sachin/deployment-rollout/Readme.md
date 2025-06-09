## Deployment Rollout

Create and apply deploy.yaml. Then rollout the version to the old one.

```bash
controlplane:~$ kubectl set image deployment/cache-deployment redis=redis:7.2.1
deployment.apps/cache-deployment image updated
controlplane:~$ k rollout status deployment cache-deployment 
deployment "cache-deployment" successfully rolled out

controlplane:~$ k describe deployments.apps cache-deployment | grep Image
    Image:         redis:7.2.1
```