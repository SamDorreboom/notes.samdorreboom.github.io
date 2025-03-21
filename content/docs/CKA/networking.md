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
