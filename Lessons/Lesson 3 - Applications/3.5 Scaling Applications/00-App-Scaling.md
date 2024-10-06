
## Application Scaling

Scaling can be done whether manually or automatically. For manual scaling just run below posted command:

``` bash
kubectl scale deployment myapp --replicas=3
```

There is also the posibility of applying HPA (Horizontal Pod Autoscaling) which will automatically set/delete pods according to pre-adjusted thresholds.
But that is out of the CKA scope.