# Deploying an ingress (proxy frontend)

Sometimes we want to deploy microservices that run single endpoints that should
be routed under the same domain name, usually you'd use a reverse proxy like
nginx to route the URLs to the microservices. In k8s world this is called an
`Ingress` and typically actually uses `nginx`.

At first we have to install the ingress pod:

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install nginx ingress-nginx/ingress-nginx
```

You'll need an ingress definition to be able to tell the router where to route:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kubeview-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  - http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: kubeview
            port:
              number: 80
```

Save that to a YAML file and run `kubectl create -f <filename>` to create
the ingress.

**Attention**: Even when using an ingress you still need a loadbalancer, else
the `EXTERNAL-IP` field of the ingress service will be `<pending>`

To try this with `kubeview`:
1. remove the service with `helm del kubeview`
2. re-install with `helm install kubeview kubeview/kubeview --set ingress.enabled=true`

Further documentation: https://kubernetes.io/docs/concepts/services-networking/ingress/

![With ingress](../img/with_ingress.png)

## Default backend (catchall)

If you're just exposing a single service or want to have a catchall to deliver
a _Bad request_ site to unknown paths you can specify a default backend like
this:

```yaml
spec:
  defaultBackend:
    service:
      name: foobar
      port:
        number: 80
```

Pay attention to define this one last in your routing table as the table is
processed top to bottom, first match wins.

## Splitting for multiple hostnames

If you want to provide data to more than one hostname (e.g. Subdomains) you
absolutely can do that by adding a `host` entry to the `rules` at the same level
as the `http` entry:

``` yaml
...
spec:
  rules:
  - host: subdomain.example.com
    http:
      paths:
...
```

You can even use wildcards like `*.example.com`

## Routing sub-paths to different microservices

There are two path-types `Prefix` and `Exact`, you may define multiple routes
in your ingress configuration to allow routing sub-paths to different
microservices running more than one service under one domain:

```yaml
spec:
  rules:
  - http:
      paths:
      - path: "/bla"
        pathType: Prefix
        backend:
          service:
            name: bla
            port:
              number: 80
      - path: "/status"
        pathType: Exact
        backend:
          service:
            name: status
            port:
              number: 80
```

The `Exact` rule will only match the exact URL, any continuations will not match
that rule. If you have a prefix and an exact rule the exact rule takes
precendence. If you have multiple matches for a prefix rule the longest one is
choosen first and if there are multiple with the same match length, the first
one wins.

You can even use Regular expressions in your paths if you enable it in the
annotations:

```yaml
metadata:
  name: my-ingress-nginx
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/use-regex: "true"
```

## Resource backends (static data)

To serve static content you will need to run a service that just runs a
webserver to supply the static content, for example another `nginx` pod.

You may let the ingress rewrite the URLs to be able to just dump the static
files into the web-root though.

```yaml
metadata:
  name: my-ingress-nginx
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/add-base-url: "true"
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  rules:
  - host: subdomain.example.com
    http:
      paths:
      - path: "/static(/|$)(.*)"
        pathType: Prefix
        backend:
          service:
            name: bla
            port:
              number: 80
```

So all URLs that look like `http://subdomain.example.com/static/main.css` will
be rewritten to show up at the service as `http://subdomain.example.com/main.css`
