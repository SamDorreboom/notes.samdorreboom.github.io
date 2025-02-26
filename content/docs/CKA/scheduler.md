# Scheduling



## Manual scheduling
Set nodeName in pod specification to set pod to a specific node.
Pod definition has by default a nodeName under spec. But this is normally not set but set automatically and scheduled by scheduler.
When there is no scheduler the pod keeps status pending, then can manually set nodeName in pod specification file. It is possible only to set this nodeName when creating. 
When pod already exist, then create binding object. In bindingobeject you specify the node and set it as api in json form request to api.


## labels and selectors
Labels are properties for each object. 
Selectors helps filter this objects. 
Annotations are used to specify non-identifying metadata to objects like buildversion, email etc. This is set in metadata.

## Taints and tolerations
Taints are restrictions for nodes. If a taint is set on a node, only pods with the right tolerant can be placed on that node. 
kubectl taint nodes node-name key=value:taint-effect
taint effect can be: NoSchedule, PreferNoSchedule or NoExecute.
Tolerations is a value of spec. values in tolerations must be in "" .
This not quarantee that the pod with the right tolerant be placed on the node with the taint. 
On the master node there is a taint active. 