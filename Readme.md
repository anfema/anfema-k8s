# K8S setup

1. [Network Bridge](docs/Network_Bridge.md)
2. [Libvirt setup](docs/Libvirt.md)
3. [Virtual machine cluster](docs/VM_Cluster.md)
4. [Deploying k8s](docs/K8S_Deployment.md)
5. [Installing Helm and running your first app](docs/helm/Installation.md)

You can use the provided `Makefile` in `cluster` to automate steps 3 and 4 and
set up a local docker registry. Just run `make` in the `cluster` subdir to get
a help page

## Kubernetes

1. [Technicalities](docs/k8s/Technicalities.md)
2. TODO: [Creating a Service](docs/k8s/Service.md)
3. [Creating a deployment](docs/k8s/Deployments.md)
4. [Deploying a Loadbalancer](docs/k8s/Loadbalancer.md)
5. [Deploying an Ingress](docs/k8s/Ingress.md)
6. [Replicating a Service](docs/k8s/Replication.md)
7. [Rolling updates](docs/k8s/Rolling_Updates.md)
8. [Running one off Jobs](docs/k8s/Jobs.md)
9. [Running Cronjobs](docs/k8s/CronJobs.md)
10. [Monitoring, Logging, Debugging](docs/k8s/Debug.md)
11. TODO: [Shared Persistent Storage](docs/k8s/Persistent_Storage.md)
12. TODO: [Network Policies](docs/k8s/Network_Policies.md)
13. TODO: [Service Accounts](docs/k8s/Service_Accounts.md)
14. [Configuration and Secrets](docs/k8s/Configuration.md)

## Helm

1. [Creating a Helm Chart](docs/helm/Chart.md)
2. [Required files in the Templates Directory](docs/helm/Required_Templates_in_a_Chart.md)
2. TODO: [Service Management](docs/helm/Management.md)

## Docker Images

1. [Creating a Registry](docs/docker/Registry.md)
2. [Create Docker Images](docs/docker/Images.md)
3. [Little helpers](docs/docker/Little_Helpers.md)