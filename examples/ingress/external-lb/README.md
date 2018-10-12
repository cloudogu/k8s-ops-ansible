# Ingress External Load-Balancer

This example demonstrates the usage of an ingress server with external load-balancer.
The load-balancer is deployed on two nodes with keepalived in order to reach ha.

## Requirements

* running k8s cluster
* working `kubectl` pointing to the cluster
* vagrant, virtualbox and ansible

## Installation

```bash
# start the storage vm
vagrant up

# install the load balancerts
ansible-playbook -i inventory playbook.yml

# install the ingress controller
kubectl apply -f k8s/ingress-contour.yml
```

## Smoke Test
```bash
# deploy demo application to test ingress
kubectl apply -f k8s/smoke-test.yml

# Open http://192.168.100.245/ in your browser, you should see the kuard application.

kubectl delete -f k8s/smoke-test.yml
```

## cleanup

```bash
# remove ingress contoller
kubectl delete -f k8s/ingress-contour.yml

# remove load balancer vms
vagrant destroy -f
```