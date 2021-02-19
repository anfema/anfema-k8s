# Replication

## Manual replication

To replicate a service you have to just add a `replicas` key to the `spec` of
your deployment. To scale up or down, just re-apply the deployment.

The other way is to run a `scale` command:

```bash
kubectl scale deployment.v1.apps/<deployment-name> --replicas=10
```

## Horizontal Autoscaler

Kubernetes can automatically scale up and down your services when a resource-
limit is hit. Usually your services will be scaled by observing the Node's CPU
load. Which means a Pod that sits idle on a node may be scaled up because another
Pod on the same node hits the CPU heavy. Because of this it is not immediately
clear what exactly happens. To mitigate that you can define health- and load-
checks yourself. This is a bit too much for this introduction so here's a
[Link to the Kubernetes Documentation of the Horizontal Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
