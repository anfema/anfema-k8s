# Deploying k8s

**Attention**: Make sure that you have the current version of kubeadm and
kubelet defined in `roles/kube-dependencies/tasks/main.yml`. Find the latest
supported version in https://packages.cloud.google.com/apt/dists/kubernetes-xenial/main/binary-amd64/Packages

To deploy kubernetes to the cluster run the corresponding ansible scripts.

```bash
ansible-playbook -i hosts bootstrap.yml
ansible-playbook -i hosts kube-masters.yml
ansible-playbook -i hosts kube-workers.yml
```

- `bootstrap` disables any automatic `apt` upgrades, disabled swap and creates a
  `kube` user in all VMs
- `kube-masters` sets up the k8s master node
- `kube-workers` sets up the k8s worker nodes

**Attention**: while running bootstrap you'll have to accept the ssh-keys of the
VM installations, so you have to type `yes<enter>` for each of the VMs

If the three playbooks did run without an error we have to tell the master node
how to find the worker nodes, for that we ssh into the master node:

```bash
ssh kube@<master_ip>
```

The password is `kubernetes`
If you run `kubectl get nodes` now, all nodes should display as `not ready`, so
let's tell kubernetes how to find the workers by installing a plugin:

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.13.0/Documentation/kube-flannel.yml
```

## Creating a comfortable interface from your local machine

SSHing into the master node is a bit inconvenient so we can copy the `kubectl`
config of the master to our local machine and run `kubectl` from there:

```bash
cd $HOME
mkdir -p .kube
scp kube@<master_ip>:.kube/config .kube/config
chmod 600 .kube/config
kubectl get nodes
```
