## Cron Job Issue

[Sachin CronJob-Issue](https://killercoda.com/sachin/course/CKA/cronjob-issue)

&nbsp;

```bash
controlplane:~$ k get pods --show-labels
## Pod is not labeled!
NAME                         READY   STATUS             RESTARTS      AGE   LABELS
cka-pod                      1/1     Running            0             80s   <none>

## Service address with Selector-> app=cka-pod
controlplane:~$ k describe svc cka-service 
Name:                     cka-service
Namespace:                default
Labels:                   <none>
Annotations:              <none>
Selector:                 app=cka-pod
Type:                     ClusterIP
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.107.51.158
IPs:                      10.107.51.158
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
Endpoints:                
Session Affinity:         None
Internal Traffic Policy:  Cluster
Events:                   <none>

## Check Pod Logs  
controlplane:~$ k logs cka-cronjob-29148402-4rfmx 
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0curl: (6) Could not resolve host: cka-pod

## Label the pods so as to receive Service traffic:
k label pod cka-pod app=cka-pod
pod/cka-pod labeled
```

&nbsp;

Modify the CronJob YAML, set cka-service for the curl command

```YAML
apiVersion: batch/v1
kind: CronJob
metadata:
  generation: 1
  name: cka-cronjob
  namespace: default
spec:
  concurrencyPolicy: Allow
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - command:
            - curl
            - cka-service
            image: curlimages/curl:latest
            imagePullPolicy: Always
            name: curl-container
            resources: {}
            terminationMessagePath: /dev/termination-log
            terminationMessagePolicy: File
          dnsPolicy: ClusterFirst
          restartPolicy: OnFailure
          schedulerName: default-scheduler
          securityContext: {}
          terminationGracePeriodSeconds: 30
  schedule: '* * * * *'
  successfulJobsHistoryLimit: 3
  suspend: false
```

&nbsp;

Check that now it is working fine:
```bash
Every 1.0s: kubectl get pods                                                                                                                         controlplane: Mon Jun  2 22:48:04 2025

NAME                         READY   STATUS      RESTARTS   AGE
cka-cronjob-29148407-nq7cj   0/1     Completed   0          64s
cka-cronjob-29148408-jkl2p   0/1     Completed   0          4s
cka-pod                      1/1     Running     0          6m44s
```

It is working even since the Scenario can't be checked:

```bash
controlplane:~$ k logs cka-cronjob-29151204-jshp8
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
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
100   615  100   615    0     0   138k      0 --:--:-- --:--:-- --:--:--  150k
```