NETWORK_PREFIX = 10.0.5.1
MASTER_IP = $(NETWORK_PREFIX)0
NUM_NODES = 3
CPUS = 2
MEMORY = 4096
BRIDGE = br0
DOCKER_REGISTRY ?= docker-registry
DOCKER_REGISTRY_PORT ?= 5000
DOCKER_REGISTRY_IP ?= `getent hosts $(DOCKER_REGISTRY) | cut -d " " -f 1`

SSH_PUBKEY=`cat ~/.ssh/id_ed25519.pub || cat ~/.ssh/id_rsa.pub`

help:
	@echo "This builds VM cluster for kubernetes"
	@echo ""
	@echo "The following targets are available:"
	@echo " - cluster: Create the VMs and provision them, run a local registry"
	@echo " - cluster_no_registry: Create the VMs and provision them, no registry"
	@echo " - destroy/clean: Destroy the cluster completely"
	@echo " - shutdown: Shutdown all VMs of the cluster"
	@echo " - start: Start previously shutdown cluster"
	@echo " - status: get node status"
	@echo
	@echo "Sub-Targets of cluster:"
	@echo " - vms: initialize vagrant VMs"
	@echo " - ansible: run ansible to provision VMs"
	@echo " - registry: set up local container registry with docker-compose"
	@echo " - loadbalancer: set up a local loadbalancer"
	@echo " - ingresscontroller: set up a local ingress controller based on nginx"
	@echo " - minio: S3 Storage API and Backend"
	@echo
	@echo "Variables:"
	@echo " - NETWORK_PREFIX: Prefix of all IPs used in the cluster, default '$(NETWORK_PREFIX)'"
	@echo " - MASTER_IP: IP address of the master, default '$(MASTER_IP)'"
	@echo " - NUM_NODES: Number of nodes to spin up, default '$(NUM_NODES)'"
	@echo " - CPUS: Number of CPU cores per node, default '$(CPUS)'"
	@echo " - MEMORY: Amount of RAM for each node, default '$(MEMORY)'"
	@echo " - BRIDGE: Name of the bridge network device, default '$(BRIDGE)'"
	@echo " - DOCKER_REGISTRY_IP: IP address of the local registry, default '$(DOCKER_REGISTRY_IP)'"
	@echo " - DOCKER_REGISTRY_HOST: Hostname of the local registry, default '$(DOCKER_REGISTRY_HOST)'"
	@echo " - DOCKER_REGISTRY_PORT: Port of the local registry, default '$(DOCKER_REGISTRY_PORT)'"
	@echo " - SSH_PUBKEY: Public key to add to all nodes as user 'kube', default your key from ~/.ssh/"


.PHONY: clean
.NOTPARALLEL:

cluster: vms registry ansible loadbalancer ingresscontroller minio

cluster_no_registry: vms ansible loadbalancer minio

ansible: hosts group_vars/all.yml
	ansible-playbook -i hosts bootstrap.yml
	ansible-playbook -i hosts kube-masters.yml
	ansible-playbook -i hosts kube-workers.yml

	@echo 'Updating ssh known hosts...'
	@ssh-keygen -R $(MASTER_IP)
	@for i in `seq $(NUM_NODES)` ; do ssh-keygen -R $(NETWORK_PREFIX)$$i ; done
	@ssh-keyscan -t ed25519 $(MASTER_IP) >> ~/.ssh/known_hosts
	@for i in `seq $(NUM_NODES)` ; do ssh-keyscan -t ed25519 $(NETWORK_PREFIX)$$i >>~/.ssh/known_hosts ; done

	scp kube@$(MASTER_IP):.kube/config ~/.kube/config
	chmod 600 ~/.kube/config
	kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/v0.13.0/Documentation/kube-flannel.yml

vms: Vagrantfile
	vagrant up
	# ensure we get an IP address
	vagrant halt
	vagrant up

Vagrantfile: Vagrantfile.template
	@echo "Creating Vagrantfile..."
	@echo "s/%NETWORK_PREFIX%/$(NETWORK_PREFIX)/g" > Vagrantfile.sed
	@echo "s/%NUM_NODES%/$(NUM_NODES)/g"           >> Vagrantfile.sed
	@echo "s/%CPUS%/$(CPUS)/g"                     >> Vagrantfile.sed
	@echo "s/%MEMORY%/$(MEMORY)/g"                 >> Vagrantfile.sed
	@echo "s/%BRIDGE%/$(BRIDGE)/g"                 >> Vagrantfile.sed
	@sed -f Vagrantfile.sed Vagrantfile.template > Vagrantfile
	@rm Vagrantfile.sed

hosts:
	@echo "Creating ansible hosts file..."
	@echo "[kube-masters]"     > hosts
	@echo "master1.kube.local ansible_host=$(MASTER_IP)" >> hosts
	@echo                                                >> hosts
	@echo "[kube-nodes]"                                 >> hosts
	@for i in `seq $(NUM_NODES)` ; do echo "worker$$i.kube.local ansible_host=$(NETWORK_PREFIX)$$i" >> hosts ; done
	@echo                                                >> hosts
	@echo "[ubuntu:children]"                            >> hosts
	@echo "kube-masters"                                 >> hosts
	@echo "kube-nodes"                                   >> hosts

group_vars/all.yml:
	@echo "Configuring ansible..."
	@echo "master_ip: $(MASTER_IP)"          > group_vars/all.yml
	@echo "registry_ip: $(DOCKER_REGISTRY_IP)"      >> group_vars/all.yml
	@echo "registry_host: $(DOCKER_REGISTRY_HOST)"  >> group_vars/all.yml
	@echo "registry_port: $(DOCKER_REGISTRY_PORT)"  >> group_vars/all.yml
	@echo "ssh_pubkey: $(SSH_PUBKEY)"        >> group_vars/all.yml


status:
	@echo "Virtual Machine status:"
	@echo "--------------------------------------------------------------------"
	@virsh -c qemu:///system list --all
	@echo
	@echo "Node Status:"
	@echo "--------------------------------------------------------------------"
	@kubectl get nodes || true
	@echo
	@echo "Pod Status:"
	@echo "--------------------------------------------------------------------"
	@kubectl get pods || true
	@echo
	@echo "Registry Status:"
	@echo "--------------------------------------------------------------------"
	@docker-compose -f ../scripts/Docker-Registry.yaml ps

registry:
	@mkdir -p data/registry
	DATA_PATH=`pwd`/data/registry docker-compose -f ../scripts/Docker-Registry.yaml up -d
	@echo "--------------------------------------------------------------------"
	@echo "Access docker registry UI at http://localhost:8080"
	@echo "Access docker registry API at http://localhost:5000"
	@echo "--------------------------------------------------------------------"

minio:
	@mkdir -p data/minio
	docker run -d --name minio -p 9000:9000 -v `pwd`/data/minio:/data minio/minio server /data
	@echo "--------------------------------------------------------------------"
	@echo "Minio access credentials: minioadmin // minioadmin"
	@echo " -> data will be stored in `pwd`/data/minio/"
	@echo "Access minio on http://`docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' minio`:9000"
	@echo "--------------------------------------------------------------------"

loadbalancer: loadbalancer.yml
	kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.5/manifests/namespace.yaml
	kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.5/manifests/metallb.yaml
	kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"
	kubectl apply -f loadbalancer.yml

loadbalancer.yml: loadbalancer.template
	@echo "Creating Loadbalancer config..."
	@echo "s/%START_IP%/$(NETWORK_PREFIX)00/g" > loadbalancer.sed
	@echo "s/%END_IP%/$(NETWORK_PREFIX)99/g"   >> loadbalancer.sed
	@sed -f loadbalancer.sed loadbalancer.template > loadbalancer.yml
	@rm loadbalancer.sed

ingresscontroller:
	helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
	helm install nginx ingress-nginx/ingress-nginx

clean: destroy
	rm -f Vagrantfile hosts group_vars/all.yml

destroy:
	vagrant destroy -f || true
	docker-compose -f ../scripts/Docker-Registry.yaml down
	docker rm -f minio

shutdown:
	vagrant halt
	docker-compose -f ../scripts/Docker-Registry.yaml stop
	docker stop minio

start:
	vagrant up
	docker-compose -f ../scripts/Docker-Registry.yaml start
	docker start minio