# Architecture
Worker Nodes:
- Kubelet: listens for instructions from apiserver
- Kube-proxy: enabling communication between services.
- Container runtime engine (docker, rkt)
Master node:
- ETCD cluster: stores information about the cluster
- Kube-scheduler: scheduling applications/containers on nodes
- Kube controller manager: multipe controllers like node controller.
- Kube-apiserver: orchestration all the operations in the cluster.

Container runtime interface (CRI): is so that every containerruntime can run with kubernetes if it is based on OCI. 
Open container initiative (oci): imagespec, runtimespec
Docker follows the oci now. 

Crictl: a cli for cri compatible container runtimes, used to inspect and debug container runtimes, works across different runtimes. Mostly used for debugging purposes.

Kubernets no longer need Docker but containerd instead.


# ETCD
ETCD is a distributed reliable key-value store that is simple, secure & fast
Key-value store. Port 2379.
Key value store: each object get own file or page. Example every person get own key/value table.

## ETCD in k8s
Stores information about cluster; nodes, pods, configs, secrets, accounts, roles, bindings, others. Every get command goes to etcd server. 
When cluster is setup from scratch: binary is installed and etcd.service runs as a service. In the etcd file the --advertise-client-urls is het ip waar etcd op luistert.
Setup with Kubeadm: etcd is setup as pod in kube-system.  
Kubernetes structure: registry --> minions, pods, replicasets, deployments, roles en secrets.
When HA there are more etcd instances with multiple nodes. 

# Kube api server
Api server is the primary management component. Kubectl command is reaching the api server.
Kube-api is the only component that communicates with etcd.
Kube api server:
1. authenticate user
2. validate request 
3. retrieve data from etcd 
4. update etcd 
5. scheduler 
6. kubelet  
You can request the api server by curl. 
When installing from scratch then kube-api binary server is available from kubernetes. It runs as a server. 
view api server:
- kubeadm: cat /etc/kubernetes/manifests/kube-apiserver.yaml
- scratch: cat /etc/systemd/system/kube-apiserver.service , ps -aux | grep kube-apiserver

# Kube controller manager
Watch status and remediate situation. Every controller communicates via kube-apiserver
Al the small controllers are packaged in kube-controller-manager. 
Controller manager binary is provided and is run as service. 
In the service file there is de config for al the controllers and which controller to enable.
Kubeadm is controller-manager a pod.

Kubeadm: cat /etc/kubernetes/manifests/kube-controller-manager.yaml
scratch: cat /etc/systemd/system/kube-controller-manager.service , ps -aux | grep kube-controller-manager

# Kube scheduler
Only deciding which pod goes on which node.
Depending on certain criteria. Looks at each pod and search for the best node.
1. filter nodes that do need fit the profile for the pod like resource
2. Rank nodes for the best fit, example free resources after pod is placed
This can be customized.
Config is same as controller manager etc.

# Kubelet
Is like the captain.
Kubelet is on every node. It runs the activities gets from the apiserver.
- register node
- create pods
- monitor node & pods
Kubeadm does not deploy kubelets!! Only binary from kubernetes. 

# Kube proxy
Every pod can reach every pod. This is accomplished with POD network. Internal virtual network. Many solutions for this network. IP from pods can change so using a service.
Service cannot join pod network because is not a container but virtual only in k8s memory.
Kubeproxy is a proces on each node, its job to look for new service, its create the aproperiate rule on each node to forward traffic to services and backend pods. 
One way for this is Iptables rules. 

Kubeadm make kube-proxy as deamonset.

# Pods
Containers are encapsulated by pods.
Multi-container pods: helper containers. Both containers contact via localhost and can use same volume. 

# Services
Service make it possible to make pod accessible to end user and other pods, or to external data source.
Node has ip, inside the node there is a ip range where the pod is in.
Inside the kubernetes node you can access it by the internal pod ip.
Service types:
- NodePort: Kubernetes service is a object that can listen on a port on the node ip, type node port
- ClusterIp: creates a virtual ip inside the cluster, to enable communication between different services.
- Loadbalancer: for cloud providers. 

## nodeport
Mapping a port on a node to a port on a pod.
Service has it own ip, clusterip of the service.
3 types of port:
- targetport: the port where the pod is listening
- Port: the port on the service itself.
- NodePort: the port that is open on the node (range 30000 - 32767)
Spec --> type , ports (array) --> targetport, port, nodeport.
With labels and selectors to connect to pods. This come in selector, here comes the labels from the pod. Can be multiple pods, it used random algo for loadbalancer.
When not defining a nodeport it automatically generated, and targetport would be same as port.
When pods are on different nodes, k8s automatic generate service on all the nodes. 

## Cluster ip
Connectivity between internal services.
Clusterip service combine al the pods together and provide a internal virtual ip adres to connect to the pods. You can connect to this service with name. 
spec --> type (clusterip default) , ports --> targetport, port
selector

## Loadbalancer
Single url to acces the application.
1. You can create a new vm and install loadbalancer on it. 
2. And setup to loadbalance between al the nodes
or
On supported cloudplatform, use the native loadbalanacer from the cloudplatforms. 
spec --> type (loadbalancer) --> ports targetport, port, nodeport.

# Namespace
Kubernetes namespaces: kube-system, default, kube-public (for all users accebile instances)
Dns name outside namespace example: db-service.dev.svc.cluster.local
service-namespace-service-domain
set kubectl to a namespace:
kubectl config set-context $(kubectl config current-context) --namespace=<namespace>

kubectl get pods --all-namespaces

# imperative vs delarative
Imperative: step by step instructions. Like step by step config on how to install nginx. 
commands like set, create, run are imperative commands.
kubectl edit deployment nginx
kubectl replace -f nginx.yaml
kubectl replace --force -f nginx.yaml

Declarative: only final dest. Now how but what. Tools like ansible, terraform. 
command like kubectl apply -f is declarative. Apply create if not exist. 
kubectl apply -f /path/to/config-files
update can also with apply. 

In exam use imperative commands to save time.

# Kubectl apply
1. If object not exist object is created
2. Live object configuration is created on kubernetes cluster
3. Last applied configuration is stored in json file.  
When apply with file then live object config is updated en then the last applied config. Only done with apply, create and update is not using this. 
So keep this is mind when using declarative and imperative commands.
Last applied is used; when something is removed from the local file, it can be compared to last applied to compare. 
https://kubernetes.io/docs/tasks/manage-kubernetes-objects/declarative-config
