k8sthw
=====

Kubernetes deployment automated using ansible.

For now, works for Ubuntu and CentOS Hosts only. 
The client has to be a Mac (Check for `when: ansible_distribution == "MacOSX"` and add additional tasks for installing 
tool on e.g. ubuntu)

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

* Make Cluster Name `kubernetes-the-hard-way ` configurable. (k8s-config/kubelet, ...)
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

# Use Cases

TODO

## Update K8s version

## Update certs

## ...
