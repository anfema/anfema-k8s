# Debugging, Monitoring and Logging

The first thing to run if you're stuck somewhere is usually:

```bash
kubectl get events
```

Which should display critical errors to look out for in the next steps.

## Debugging stuck Pods

See if a pod is stuck:

```bash
kubectl get pods
```

There are multiple options on what is happening.
- `Waiting` waiting for an external resource
- `Init:N/M` pods are initializing
- `Init:Error` the Init phase of a service failed
- `Init:CrashLoopBackOff` the Init phase has failed multiple times and k8s gave
  up
- `Pending` the pod is starting up
- `PodInitializing` the Init phase has been run successfully and the pod is
  spinning up
- `Running` the pod is running as far as k8s is concerned

You can see details on a Pod by running:

```bash
kubectl describe pod <name>
```

If your pod stays in `Pending` there may be multiple causes for that:
- Cluster is overloaded, no resources available
- You are using `hostPort`, which means you request a port on a Node, only one
  service per node can listen on the same host port, so avoid this if possible

If your pod stays in `Waiting`:
- Is the image name correct?
- Is the image in the repository?
- Network error, no connection to repository

## Debugging a failed Node

At first we need a list of all nodes:

```bash
kubectl get nodes
```

Then we can get information of a node by describing it:

```bash
kubectl describe node <name>
```

Usually, if it is not a hardware failure you should get information why the
node is down or stuck. (e.g. No RAM left, out of disk space, etc.)

## Displaying logs

To examine logs (usually stdout and stderr are redirected to the log) run

```bash
kubectl logs <pod-name> <container-name>
```

If the pod is currently running but crashed before use the `--previous` flag
to access older logs.

## Running commands in a container

You can execute commands in a running container and even get an interactive
shell if needed:

```bash
kubectl exec <pod-name> -c <container-name> -- <your command>
kubectl exec -it <pod-name> -- sh
```

The `-it` runs the command interactively, so you can interact with the shell.

For more information [Read the Kubernetes Docs](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-running-pod/)