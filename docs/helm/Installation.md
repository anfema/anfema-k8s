# Installing helm

Get it from your package repository or install it via the installer:

```bash
curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh
chmod 700 get_helm.sh
./get_helm.sh
```

On OSX it seems to be in homebrew, so just run `brew install helm`

## Running your first app

A good example is the kube visualizer which displays what is going on in your
cluster:

```bash
helm repo add kubeview https://benc-uk.github.io/kubeview/charts
helm install kubeview kubeview/kubeview
```

Ignore the last line where the app is available under, we need a loadbalancer
to be deployed to make this actually accessible

You can verify that the app is running with `kubectl get pods` and see service
info with `kubectl get svc` which should display the `kubernetes` service and the
new `kubeview` service.
