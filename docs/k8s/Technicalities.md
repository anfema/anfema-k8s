# Overview over kubernetes architecture and slang

## Slang terms

Cluster
: This describes the complete group of machines working together as one black-
  box to deploy software to. It consists of multiple machines or VMs which host
  at least one master and one or multiple workers

Master
: The kube-master is the Machine/VM that the kubernetes daemon itself is running
  on. You need Networked access to this master to run any `kubectl` command

Worker
: The kube-worker is a Machine that shares it's resources to be managed by the
  master.

kubectl
: this is the management frontend for the administrator, kubectl is a command
  line tool to roll out changes into the cluster. You need a configuration file
  in `~/.kube/config` to be able to run commands on the master.

Pod
: A Pod is a container running one or multiple Services, usually you want one
  Service per Pod if you can do so. Sometimes one Pod may run multiple services
  if they are tightly coupled or need to communicate via an IPC mechanism that
  is not able to be networked (like SysV Shared Memory)

Container
: A Container is the unit that Kubernetes uses to deploy a Pod. It is usually
  the exact equivalent to a Docker container and uses Docker container images.

Service
: A service is a container definition which describes what is running in a
  container

Replica
: A Replica is the unit Kubernetes uses to describe multiple deployments of the
  same service. To be able to use a replicated service you will need a load-
  balancer to spread the requests over all replicas.

Deployment
: A deployment is the description of what services to run in which numbers. A
  deployment has a name to be able to refer to it to either update it or tear it
  down again. A deployment can describe how to scale a service and is used to
  describe how services can talk to each other. Usually you will have a YAML
  file to describe a deployment. You can change that file (and update the
  version number) and re-apply the deployment to run a rollout

Horizontal-Scaling
: Horizontal Scaling describes how kubernetes will handle load on your Pods, if
  a Pod gets overloaded you may specify how many parallel Pods (replicas) the
  management daemon may run automatically to scale up the service.

Volume
: A volume is a description to on-disk memory that is ephemeral to a running
  Pod. It will be destroyed if the Pod is torn down.

Persistent Volume
: A persisten volume is a method to retain on-disk memory over a networked
  cluster. By the nature of being networked the persistent volume has to be
  networked too. Usually you will use the cloud provider's block storage (if
  persistent to a Pod but not shared between replicas or pods) or bucket storage
  (if parallel access is needed) to host persistent volumes. If you're running
  your own cluster you can use Network Shares like NFS to facilitate that.

Ingress
: An ingress is a kind of request router to be able to host microservices under
  a shared domain. Usually you will use nginx as the reverse-proxy and router.
  An ingress only routes HTTP usually but bare TCP and UDP ingresses are
  available. An ingress may be used to terminate HTTPS connections and
  communicate only HTTP in the cluster.

Loadbalancer
: The loadbalancer is strictly needed if you want to access the cluster from
  the external network. Usually you would use the cloud provider's loadbalancer
  as they scale better than running your own in a Pod. If you're running a local
  cluster you would probably use `metalLB` in a Pod with an external IP.

## Architecture

To build a cluster you will have `Nodes` that register to the Master running the
kubernetes control-plane daemon. Usually the nodes know the master and
self-register.

### Nodes

You can control nodes with `kubectl` to mark them defective if a machine goes
down or has a hardware issue. The control plane will then migrate all running
Pods away from that node and not use it again.

```bash
kubectl cordon <Nodename>
```

If you want to get the node status run:

```bash
kubectl describe node <Nodename>
```

### Containers

On a Node you run containers which are deployed by extracting a container image
on the local filesystem and running the container from that image.
The container runtime is running the containers, in current deployments this
would be `containerd` like when you run local docker containers.

### Workloads

On the containers you are running workloads, this can be:
- A `Deployment` or `ReplicaSet` for running services
- A `StatefulSet` for providing Persistent Volumes to a service
- A `DaemonSet` which runs Node-Infrastructure, e.g. the control-plane is such
  a set.
- A `Job` or `CronJob` which is a single task that has to use the cluster
  infrastructure, e.g. a Database Migration or a periodic cleanup task

### Networking

If you have multiple containers running in a Pod you can use _loopback_
networking to communicate between services.

The next level is cluster networking, you usually use that by using common names
like `database` or `frontend` which will be DNS resolved by the container-
runtime to an cluster internal IP address.

The last level is external service IPs that will be visible on the Node`s
external IP. Usually only loadbalancers and ingress Pods actually have
externally reachable ports, but you may define services that are visible for
special cases.

If you want to communicate between Pods you will have to define Network Policies
to describe which Pod may connect to which other Pods internally, usually all
Pods are running in non-isolated mode which allows incoming traffic from all
cluster internal IPs which is not something you probably want.

### Storage

Usually you want to manage the storage that your Pods will consume to not starve
a node and ultimately crash it.

There are 2 types of storage. Ephemeral and Persistent.
Ephemeral storage is tied to a Pod and destroyed if the Pod is torn down while
Persistent storage will remain available after the Pod is gone (and may be re-
attached to another Pod). When using persistent Storage there are multiple
providers to allow for parallel access from multiple Pods or replicas and
high-speed variants that are only allowed to be attached to a single Pod (or
to multiple Pods running on the same node, but that is a special case).

### Resources

    TODO

### Quotas

    TODO

### Staggered rollouts

    TODO