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

## Node selectors
In the pod definition file nodeSelector: in the spec sector. In nodeSelector set size: Large. 
Size is in this case a label that can be set on a node. 
Label nodes:
```bash
kubectl label nodes <node name> <label-key>=<label-value>
```
With node selectors it is not possible to like say only on medium and large or not on small, here fore node affinity.

## node affinity
PODS are hosted on particulary node.
affinity is also set in de spec section. Node affinity is set in the affinity section.
There are different types: duringscheduling (pod not exist), DuringExecution (pod already exist)
If there are no nodes with the nodeaffinity expression; the behaviour is done by the type described above. 

## Resource limits
Resource requests can be set when creating. To specify the amount of resources a pod need.
CPU 1 = 1 aws vcpu, 1 gcp core, 1 azure core, 1 hyperthread.

It is possible to set a resource limit to a pod. under resource --> limits
When exceed limit: for cpu it is not possible it throttle.
But for memory it is possible to use more than limit, when that happend a lot then the pod terminate (oom (out of memory)).

Default behavior:
CPU:
- is no requests / limits. 
- When there is a limit set but no requests, default the requests is set to limit.
- Requests set and limit not. When possible pod can use a many cpu. This is useful because if limit set and cpu is available it is not used.
Memory:
- No requests, no limits: one pod can use all
- limit is set; requests = limit (default)
- requests and limit set
- requests but no limit: when available pod can use as much memory available. Memory can't be throttled only the pod can be killed.

### Limit range
Default values for resources at namespace level.
kind: LimitRange. Here it is possible to sepcify default limit and request.
Limitrange only affect new created pods, not exisiting pods.
#### Resource Quotas
Namespace level object, to set hard limit cpu and memory for all the pods together. 
kind: ResourceQuota. 

Pod cannot edit for this types. But deployment can.
kubectl create -f /tmp/kubectl-edit-ccvrq.yaml
and then kubectl replace --force -f 
kubectl get pod webapp -o yaml >&nbsp;my-new-pod.yaml


## Deamonsets
Deamsets is to run a copy of a pod on every node.
Use case: monitoring solution, logs viewer.
Kube-proxy is a good use case for a deamon sets, or network pods.
Kind: deamonset

## Static pods
The only thing a kubelet knows on a node is how to create a pod. If there isnt a cluster only a worker node, then it is possible to give te kubelet a pod definition file. Location for this is /etc/kubernetes/manifests, pod definition get created if it is in that directory. Kubelet reads that directory to create pods.
To enable this in kubelet.service; --pod-manifest=path=/etc/Kubernetes/manifests; or set in kubelet.service config=kubeconfig.yaml and set in kubeconfig.yaml staticPodPath: ..

Then to see the containers run docker ps. WHen there is a api server, api server knows of the static pods. 
To create master node you can set controller manager, apiserver and etcd pod in the /etc/kubernetes/manifests folder. Kubeadm does this.
Difference between deamonset: deamonsets are created by api server, static pod by kubelet; static pods for deploy control plane components, deamonsets for monitoring etc.
```bash
cat /var/lib/kubelet/config.yaml | grep static
```

## Multiple schedulers
It is possible to create own scheduler. You can specifie the scheduler defined in the pod. 
schedulername

To see which scheduler has the work, run get events -o wide and source is the scheduler.
### deploy scheduler as pod
In namespace: kube-system
use image from kube scheduler and pass in command sections the scheduler config file.
Kube scheduler config file is a yaml file with kind KubeSchedulerConfiguration. In this file you can specify the leaderElection. 

## Configuring scheduler profile
You can specify priorityclassname to scheduling queu.
priorityclassname: high-priority
Steps scheduling: scheduling queue --> filtering (filter the nodes that do not fit) --> scoring (nodes are scored based on free space) --> binding (pod is bound to a node) 
Every step has a plugin; queue = prioritysort, filtering = noderesourcefit nodename nodeunschedulable, scoring = noderesourcefit imagelocality, binding =  defaultbinder
It is possible to custom the plugins or own plugins. Each stage a a stagingpoint where a plugin can be connected to: queuesort, pre filter, filter, postfilter, prescore, score, reserve, permit, prebind, bind, postbind
In a kubeschedulerconfiguration you can define multiple schedulername to specify more scheduler with different configs. Under each schedulername profile you can set plugins settings disable or enable specific plugins. 
https://github.com/kubernetes/enhancements/blob/master/keps/sig-scheduling/1451-multi-scheduling-profiles/README.md
https://github.com/kubernetes/community/blob/master/contributors/devel/sig-scheduling/scheduling_code_hierarchy_overview.md

## Admission controllers
Authentication is repsonsible to authenticate request from kubectl. Kubectl stores keys in ~/.kube/config.
After authentication it goes to authorization if user is authorized to perform the action. 
Authorization can be done by Role. Role is based on the k8s api. 
Admission controllers comes after authorization and can do more based on rules. Like image can only come from certain repos.
Controllers: alwayspullimages, defaultstorageclass, eventratelimit, namespaceExist, NamespaceAutoProvision many more.
kube-apiserver -h | grep enable-admission-plugins
enable: in service or /etc/kubernetes/manifests/kube-apiserver.yaml
ps -ef | grep kube-apiserver | grep admission-plugins (see enabled and disabled if it is a pod)

(get it when it is a pod) kubectl exec -it kube-apiserver-controlplane -n kube-system -- kube-apiserver -h | grep 'enable-admission-plugins'

The NamespaceLifecycle admission controller will make sure that requests
to a non-existent namespace is rejected and that the default namespaces such as
default, kube-system and kube-public cannot be deleted.
This admission controller observes creation of PersistentVolumeClaim objects that do not request any specific storage class and automatically adds a default storage class to them. This way, users that do not request any special storage class do not need to care about them at all and they will get the default one.


### Validating & mutating admission controllers
Validating admission controller (validate rquest and allow/denie) is for example the namespaceExists controller.
Mutating admission controller (change request) is for example the defaultStorageClass, when a claim is created without storageclass it gets default.
Controller can have both like NamespaceAutoProvision. But first mutating then validating

Create own admission controller:
MutatingAdmissionwebhook, ValidatingAdmissionwebhook this controller can point to a admission webhook server. And the server has our own admission code and logic. 
1. Admission webhook server https://github.com/kubernetes/kubernetes/blob/v1.13.0/test/images/webhook/main.go must receive and response with the right things. can be deployed inside or outside the cluster. 
2. Create webhook admission: validatingwebhookconfiguration, this include caBundle (certificates), under rules specify when de admission should be contacted.

See al the default settings for a pod
```bash
kubectl get pod pod-with-default -o yaml
```