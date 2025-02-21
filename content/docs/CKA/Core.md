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
Key-value store 
