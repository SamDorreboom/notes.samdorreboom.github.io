

## OS upgrades
If a node goes down and Immediately goes up again: kubelet starts up and pods get started
If the node was down for more then 5 minutes then the pods are terminated from that node and pods are dead. En recreated on other nodes. When node comes back online the node is seen as a blank node. 
To terminate pods and recreate it on a other node then drain a node. Drain sets a node on unschedable 
```bash
kubectl drain node-1
```
To enable the node again for scheduling
```bash
kubectl uncordon node-1
```
The old pods not getting back

The cordon command set a node unschedable but not terminate pods.
```bash
kubectl cordon node-2
```
If a pod is not in a replicaset, drain would not work on that node.

## Kubernetes releases


## Cluster upgrade introduction

