# Vagrant/KVM and Ansible setup to run the cluster

You'll have to edit some files to be able to access the cluster later on, this
is mainly hostnames and IP addresses.

All files referenced hereafter are relative to the `k8s-kvm` directory.

After setting up the configuration files below start the VM cluster by issuing
a `vagrant up` command, now vagrant will configure the worker and master VMs.

## `Vagrantfile`

```
# Net work prefix in which a single digit is appended
# ex 192.168.1.5 will have a master at 192.168.1.50 and workers starting
# from 192.168.1.51
NETWORK_PREFIX="192.168.2.24"

# base image to use for the cluster
IMAGE_NAME = "generic/ubuntu2004"
```

**Attention**: The network prefix is not an IP address, the scripts
_concatenate_ a number to that prefix (0 for the master, 1-n for the cluster
workers). Make sure this will not collide with your DHCP range if your router
dynamically allocates IP addresses in your network.

You can set for the base image what you want to use to run the containers on,
this is not a container base image, but the image that the container daemon runs
on. Best is to stick to Ubuntu LTS.

If you want more or less than 3 worker nodes you can set `NUM_NODES`, be sure
to not create more than 9 as the concatenation from above will fail.

You can change the CPU cores assigned to the nodes and the amount of RAM to
expend per node. Make sure to not overallocate the RAM. If you overallocate the
CPU cores of your machine you will get a similar effect to badly behaving
neighbors on AWS EC2 instances with VCPUs.

```
   config.vm.provider :libvirt do |libvirt|
    libvirt.cpus = 2
    libvirt.memory = 4096
```

## `hosts`

```
[kube-masters]
master1.kube.local ansible_host=192.168.2.240

[kube-nodes]
worker1.kube.local ansible_host=192.168.2.241
worker2.kube.local ansible_host=192.168.2.242
worker3.kube.local ansible_host=192.168.2.243

[ubuntu:children]
kube-masters
kube-nodes
```

Here we have to setup hostnames and the corresponding IP addresses. These are
the same IPs as we set up with the prefix in the `Vagrantfile`. The master gets
the `0`, the workers are numbered from `1`.

## `kube-master-init/tasks/main.yml`

We have one change in the cube master tasks file to allow a `kubectl` from
outside the master VM to control the kubernetes cluster:

```
  command: kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-cert-extra-sans 192.168.2.240
```

Replace the IP address with the IP you used for the master. This will ensure the
self-signed certificate generated while running the setup procedure will include
the external IP address that is visible to the network.
