## Lesson 3 - Applications

### Deployments

Standard way to for running containers in K8s. It manages the ReplicaSet (which takes care of the # or running containers), ensuring scalability. Offers RollingUpdate feature for zero-downtime application update.

It is always advisable to get familiar with help documentation.
<b> kubectl create deploy -h | less</b>