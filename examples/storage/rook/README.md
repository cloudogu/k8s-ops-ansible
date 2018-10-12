# Rook

## Install

```bash
# we use beta, because stable in not yet available
helm repo add rook-beta https://charts.rook.io/beta
helm install --name rook --namespace rook-ceph-system rook-beta/rook-ceph

# create the ceph cluster
kubectl apply -f cluster.yml

# create the block storage and iits storage class
kubectl apply -f storageclass.yml
```

## Smoke Test / Blockstore

```bash
# apply nfs client
kubectl apply -f smoke-test.yml

# get name of running pod
POD_NAME=$(kubectl get pods -l app=nginx-on-rook -o jsonpath="{.items[0].metadata.name}")

# forward nginx port to local machine
kubectl port-forward $POD_NAME 8080:80 >/dev/null &

# check if everything works as expected
curl http://localhost:8080/message.txt

# stop port-forward
kill %1

# remove test deployment
kubectl delete -f smoke-test.yml
```

## Cleanup

```bash
# remove storage class
kubectl delete -f storageclass.yml

# remove ceph cluster
kubectl delete -f cluster.yml

# remove operator
helm delete --purge rook

# remove namespace and all components
kubectl delete namespace rook-ceph-system
```
