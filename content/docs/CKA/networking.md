# Networking

## Network namespaces
create network namespace: ip netns add red
Show interfaces: ip link
run ip link in namespace: ip netns exec red ip link or ip -n red link
Containers are in a network namespace. Connect two network namespaces together with a virtual cable or pipe: ip link add veth-red type veth peer name veth-blue.
ip link set veth-red netns red (connect interface / virtual cable to network namespace)
TO enable the link: ip -n red link set veth-red up. To connect the interfaces create a virtual switch with like linux bridge. To create nat on linux host: iptables -t nat -A POSTROUTING -s <ipcdr> -j MASQUERADE
Port forwarding rule: iptables -t nat -A PREROUTING --dport 80 --to-destination ip:port -j DNAT


## Docker network
Host: using host network
Bridge: internal private network is created

## CNI - container networking interface
Is a single standard approach for container networking
Container runtime must create network namespace
Identify network the container must attach to
Container runtime to invoke network plugin (bridge) when container is added
Container runtime to invoke network plugin (brdige) when container is deleted
json format of the network config

Plugin:
must support command line arguments add/del/check
must support parameters container id, network ns etc
Must manage ip address assignment to pods
must return results in a specific format
Plugins rkt and kubernetes: bridge, vlan, ipvlan, macvla, windows, dhcp, host-local

Docker doesnt implement cni but has own container network model (CNM)
Because of this k8s creates containers on the none network and then create own network. 

## Cluster networking
### Networking cluster nodes
Each node must have at least 1 interface connected to network. Each interface must have an address configured. As wel as a unique mac adress. 
The master should accept connections on 6443 fro the api server. Kubescheduler port 10250. controller manager 10252.
Worker nodes port 30000 to 32767 for external services.. ETCD on port 2379 and port 2380 for etcd clients to communicate with each other. 
kubernetes.io/docs/setup/independent/install-kubeadm/#check-required-ports
https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-implement-the-kubernetes-networking-model
https://kubernetes.io/docs/concepts/cluster-administration/addons/
Commands:
```bash
ip link
ip addr
ip addr add ip dev eth0
ip route
ip route add
cat /proc/sys/net/ipv4/ip_forward
arp
# for port listening
netstat -npl | grep -i scheduler
netstat -npa | grep -i etcd | grep -i 2379 | wc -l
```

### Pod networking
Networking model
- every pod should have an ip address
- every pod should be able to communicate with every other pod in the same node
- every pod should be able to communicate with every pod on other nodes without nat.
There are many solutions that provide this

Each node has a bridge network. And each bridge has it own subnet. And pods connect to the bridge network on the node. 
For pods to connect to pods on other nodes, make a route to the node ip. But this is a lot of a network, better is define a router in cluster. 

Get Kubernetes service like kubelet
```bash
 ps -aux | grep kubelet | grep --color container-runtime-endpoint
 ```

### Container networking interface (CNI) in K8s
- Container runtime must create network namespace
- identify network the container must attach to
- container runtime to invoke network plugin (bridge) when container is added.
- container runtime to invoke network plugin (bridge) when container is deleted
- json format of the network config
Configuring cni:
For configuring containers the container runtime is responsible like containerd en cri-o
Network plugins like flannel, cilium.
Plugins are installed at /opt/cni/bin (alle the plugins are here)
How to use the cni is in /etc/cni/net.d (alfabetic order)

#### CNI Weave
! Weave cloud is end of life new is weave net:
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml

Example of CNI plugin is Weaveworks. On each node there is a agent. They communicate with each other. Weave creates own bridge on each node. Weave networking looks like real networking with routers. 


**IPAM WEAVE**
CNI plugin is responsible to take assigning ip to pods.
Host local plugin or dhcp
In /etc/cni/net.d/net-script.conf you can specify in ipam the type for ip 

## Service networking
Clusterip Service is accessible by all pods on the cluster. Service is not bound to a node. Service gets a intern ip.
NodePort is accessible outside the cluster. It gets a intern ip and is accessible on a port on the node.
Each node runs a kubelet proces, kubelet service watches the changes through api server.
Each node runs a kube proxy. Kubeproxy watches changes by api server, every time a new service is created, kube proxy gets in action. Service dont exists on a node just on all nodes. Service gets a ip and the proxy creates ip route from service to pods.

Proxy modes: 
userspace
iptables (default)
ipvs
Proxy mode can be set:
```bash
kube-proxy --proxy-mode [userspace | iptables | ipvs]
```
```bash
ps aux | grep kube-api-server
```
```bash
iptables -L -t nat | grep db-service
```
Service ip range in kube api config
```yaml
cat /etc/kubernetes/manifests/kube-api
```

## DNS
Kubernetes has own DNS. DNS name = service name
DNS name: servicename.namespace.svc.rootdomain
Pods get also a dns name but the hostname is the ip.
DNS server in /etc/resolv.conf
CoreDNS server is a pod in the kube-system namespace.
/etc/coredns/Corefile
coredns settings are in config map coredns
in the kubelet config the dns ip is set
/var/lib/kubelet/config.yaml

## Ingress
Without ingress create proxy server that point to nodeport
Loadbalancer can be used in a cloud platform. Loadbalancer service sends a request to the cloud provider.
In kubernetes cluster deploy a proxy: nginx, haproxy, traefik. Then configure ingress resources with ingress definition file
Proxy is not deployed by default
GCP HTTPS Loadbalancer GCE, nginx, contour, haproxy, traefik, istio. GCP and nginx are maintained by kubernetes project





