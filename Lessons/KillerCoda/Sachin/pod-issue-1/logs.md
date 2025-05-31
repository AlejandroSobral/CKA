```bash
kubectl describe pods nginx-pod
...
..
.
Events:
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  Normal   Scheduled  2m56s                default-scheduler  Successfully assigned default/nginx-pod to node01
  Normal   Pulling    87s (x4 over 2m56s)  kubelet            Pulling image "nginx:ltest"
  Warning  Failed     86s (x4 over 2m54s)  kubelet            Failed to pull image "nginx:ltest": rpc error: code = NotFound desc = failed to pull and unpack image "docker.io/library/nginx:ltest": failed to resolve reference "docker.io/library/nginx:ltest": docker.io/library/nginx:ltest: not found
  Warning  Failed     86s (x4 over 2m54s)  kubelet            Error: ErrImagePull
  Normal   BackOff    6s (x10 over 2m53s)  kubelet            Back-off pulling image "nginx:ltest"
  Warning  Failed     6s (x10 over 2m53s)  kubelet            Error: ImagePullBackOff
```bash