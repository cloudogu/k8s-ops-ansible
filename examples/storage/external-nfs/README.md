# External NFS

This example demonstrates the usage of an external nfs server as volume provider in an on-premise k8s cluster.

## Requirements

* running k8s cluster
* working `kubectl` pointing to the cluster
* helm installed and initialized
* vagrant, virtualbox and ansible

## Installation

```bash
# start the storage vm
vagrant up

# install the nfs serer
ansible-playbook -i inventory playbook.yml

# install nfs-client-provisioner
helm install --name nfs-cp --namespace kube-system --values nfs-cp-values.yml stable/nfs-client-provisioner
```

## Smoke Test
```bash
# apply nfs client
kubectl apply -f nfs-cp-test.yml

# get name of running pod
POD_NAME=$(kubectl get pods -l app=nginx -o jsonpath="{.items[0].metadata.name}")

# forward nginx port to local machine
kubectl port-forward $POD_NAME 8080:80 >/dev/null &

# check if everything works as expected
curl http://localhost:8080/message.txt

# stop port-forward
kill %1

# remove test deployment
kubectl delete -f nfs-cp-test.yml
```

## cleanup

```bash
# remove nfs-client-provisioner
helm delete --purge nfs-cp

# remove storage vm
vagrant destroy -f
```