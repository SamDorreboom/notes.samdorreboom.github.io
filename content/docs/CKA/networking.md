# Networking

## Network namespaces
create network namespace: ip netns add red
Show interfaces: ip link
run ip link in namespace: ip netns exec red ip link or ip -n red link
Containers are in a network namespace. Connect two network namespaces together with a virtual cable or pipe: ip link add veth-red type veth peer name veth-blue.
ip link set veth-red netns red (connect interface / virtual cable to network namespace)
TO enable the link: ip -n red link set veth-red up. To connect the interfaces create a virtual switch with like linux bridge. To create nat on linux host: iptables -t nat -A POSTROUTING -s <ipcdr> -j MASQUERADE
Port forwarding rule: iptables -t nat -A PREROUTING --dport 80 --to-destination ip:port -j DNAT

## Cluster networking

