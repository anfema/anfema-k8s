# Deployments

A deployment is primarily a `YAML` file which defines what to deploy and its
settings.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

Let's go over the keys in here:
- `apiVersion` just tells `kubectl` the version of the file we use
- `kind` always `Deployment`
- `metadata` -> `name` is the name of the deployment so we can manage it later

In `spec` we find the definition of the deployment itself. At first we need
a `selector` which matches a `label` we defined previously or a new one. This
is used to select the correct `template` to override when we later on update a
deployment so we do not have to update all of it all the time.

The `template` defines which containers to run and which ports to expose to the
cluster network. Here we run a container named `nginx` with the docker image
named `nginx` in version `1.14.2` and expose port 80.

To run this deployment we tell `kubectl` to apply it:

```bash
kubecl apply -f nginx-deployment.yaml
```

You can even use HTTP URLs instead of a filename to run a deployment from.

To fetch information for a deployment run these commands:

```bash
kubectl describe deployment nginx-deployment
kubectl get pods -l app=nginx
kubectl describe pod <pod-name>
```

## Updating a deployment

To update a deployment just update the `YAML` file keeping the `labels` and
`matchLabels` the same and change any other value, then re-apply it with
`kubectl`.

## Deleting a deployment

When you're deleting a complete deployment all associated resources are
destroyed too, so be careful when destroying Persistent Volumes.

**ATTENTION**: Best practice for deploying Persistent Volumes is to run them in
a seperate deployment so you will not accidentally tear the complete volume down
by deleting e.g. your database server deployment!

```yaml
kubectl delete deployment <deployment-name>
```
