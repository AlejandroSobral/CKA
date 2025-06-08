controlplane:~$ k describe deployments.apps redis-deployment | grep Image
## Workloads & Scheduling
Due to a missing feature in the current version. To resolve this issue, perform a rollback of the deployment redis-deployment to the previous version. After rolling back the deployment, save the image currently in use to the rolling-back-image.txt file, and finally increase the replica count to 3 ."
    

```bash
controlplane:~$ k describe deployments.apps redis-deployment | grep Image
    Image:         redis:7.2.1

controlplane:~$ k rollout undo deployment redis-deployment 
deployment.apps/redis-deployment rolled back

controlplane:~$ k describe deployments.apps redis-deployment | grep Image
    Image:         redis:7.0.13

controlplane:~$ k scale deployment redis-deployment --replicas=3
deployment.apps/redis-deployment scaled
```

