# Deploying a loadbalancer on K8S

Usually you will use the loadbalancer of your cloud provider for this, as we
are our own cloud provider here we need to install one ourselves. Everyone seems
to use `metalLB` for this, so we get us that one too:

We need a configuration for the service first:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 10.0.6.0/24
```
If you want the service to be accessible from outside the local machine set the
address range to something you can actually route in your network. You can use
CIDR notation (e.g. `10.0.6.0/24` for the complete `10.0.6.x` subnet) or define
a range of addresses to use (e.g. `10.0.6.1-10.0.6.20`).

Now install `metalLB`:

```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.5/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.5/manifests/metallb.yaml
kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"
```

And install the config:

```bash
kubectl apply -f <configfile>
```

If you check the service list now with `kubectl get svc` you will see the
external IP of your `kubeview` to be populated.

Further information: https://metallb.universe.tf/installation/

![With loadbalancer only](../img/without_ingress.png)
