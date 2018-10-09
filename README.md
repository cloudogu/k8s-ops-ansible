k8s-ops-ansible
=====

Kubernetes deployment automated using ansible.

For now, works for Ubuntu and CentOS Hosts only. 
The client has to be a Mac or a Linux 64bit machine.

Note that while this works in general, it still is **work in progress**.

# Requirements

* ansible >= 2.6

# Getting started

* For testing, you can just spawn VMs using Vagrant
  * Uses Ubuntu 18.04 image by default with 3 masters and workers (See `Vagrantfile`).
  * These can be set using env vars (e.g. `export K8S_MASTERS=1`)
    * `K8S_MASTERS=3`
    * `K8S_WORKERS=3`
    * `K8S_VM_MEMORY=1024`
    * `K8S_VM_IMAGE=bento/ubuntu-18.04` (also tested with `bento/centos-7`, you might have to comment 
      `group_vars/controllers.yml` and `group_vars/workers.yml`)
  * `vagrant up`
* However, this will also work with preconfigured VMs or bare metal machines.
  * Adapt `inventory` and 
  * `group_vars/controllers.yml` / `group_vars/workers.yml`. 
* Start Ansible rollout by calling: `ansible-playbook -K --inventory inventory playbook.yml`
* Installs `kubectl` on your local machine and adds a context called `default` to your kubeconfig (`~/.kube/config`).  
  You can then start using your cluster via `kubectl`. For example
  * `kubectl run nginx --image nginx && kubectl port-forward $(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{end}}' | grep nginx) 8080:80`   
  * Access the NGINX pod on your cluster at http://localhost:8080/

# Inventory

* Defines IPs for groups workers and controllers
* For controllers: Distinguish between keepalived master and slave and prios

# Group Variables (group_vars)
* Shared variables (all.yml)
* Defines connection details (wokers.yml und controllers.yml)
* Python interpreter Path, because not on PATH in Ubuntu 18.04 (wokers.yml und controllers.yml)

# Playbook

* Defines which roles are executed on local, worker and controllers hosts
* Are executed sequentially from top to bottom (2xlocal, 1 before and 1 after cluster setup)

# Roles

## Local

### k8s-pki

* Generate certs for all k8s components. 
  admin account, control plane components, workers (a separate one for each kubelet)
* Joins all controller IPs into certificate
* Adding worker after initial creation works
* Adding node to controllers after intitial creation does not work yet. For this to work, delete kubernetes.pem.
  On the next execution a new pem with all new IP addresses is generated and later applied to all controller nodes.
  Is this really necessary? It's done in KTHW, so we realize it here as well...

### k8s-config

* Creates kubeconfig for all components (uses embedded certs)
 
### k8s-enc

* Creates encryption key for secrets

# Shared by workers and controllers

* Sets NTP and TimeZone
* Sets hostnames
* Creates user and group

# Controllers

## etcd

Sets up etc secured with mutual certs and running as non-root.

## k8s-controlplane

Configures and sets up services for all binaries

Note: Scheduler `allocate-node-cidrs=true` and `cluster-cidr` for flannel.

## keepalived

Provides virtual IP address for connecting to API server.

#  Workers

## k8s-worker

Does it all: Installs runc, containerd, kubelet, kube-proxy. Sets up networking (CNI) and kubelet certificates.

# TODOs

* Make Cluster Name `kubernetes-the-hard-way` configurable. (k8s-config/kubelet, ...)
* Make local kubeconfig context `default` configurable. (k8s-local)
* k8s-config: All ymls execpt main could be merged into one when using parameters for certs and names (use an object in `with_items`)
* k8s-commons: Do we really need NTP (other service for time syncing will do as well) - requires ubuntu universe
* Don't run executables as root
* etcd: 
  * Don't disable firewall, configure ports
  * Restart etc if certs / or binary change (necessary for updates) realize using notify
* apiserver: Are network policies active?
* Restarts on change in k8s-controlplane (necessary for updates) realize using notify
* Expose ip 10.244. as parameter
* keepalived.conf check HTTP instead of process
* Non-idempotent tasks: 
  * `install kubectl (Linux)`: `fatal: [localhost]: FAILED! => {"changed": false, "dest": "/usr/local/bin/kubectl", "gid": 0, "group": "root", "mode": "0755", "msg": "Request failed", "owner": "root", "response": "HTTP Error 400: Bad Request", "size": 55414436, "state": "file", "status_code": 400, "uid": 0, "url": "https://storage.googleapis.com/kubernetes-release/release/v1.11.3/bin/linux/amd64/kubectl"}`
    Workaround: Delete `/usr/local/bin/kubectl` locally
  * `apply k8s resources` / `coredns.yaml`: `failed: [controller0] (item=coredns.yaml) => {"changed": false, "error": 422, "item": "coredns.yaml", "msg": "Failed to patch object: b'{\"kind\":\"Status\",\"apiVersion\":\"v1\",\"metadata\":{},\"status\":\"Failure\",\"message\":\"Deployment.apps \\\\\"coredns\\\\\" is invalid: spec.template.spec.containers[0].ports[1].name: Duplicate value: \\\\\"dns-tcp\\\\\"\",\"reason\":\"Invalid\",\"details\":{\"name\":\"coredns\",\"group\":\"apps\",\"kind\":\"Deployment\",\"causes\":[{\"reason\":\"FieldValueDuplicate\",\"message\":\"Duplicate value: \\\\\"dns-tcp\\\\\"\",\"field\":\"spec.template.spec.containers[0].ports[1].name\"}]},\"code\":422}\\n'", "reason": "Unprocessable Entity", "status": 422}`
    Workaround: `kubectl delete -f  roles/k8s-controlplane/files/coredns.yaml` 

# Use Cases

TODO

## Update K8s version

## Update certs

## ...
