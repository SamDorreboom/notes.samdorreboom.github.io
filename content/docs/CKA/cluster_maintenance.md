

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
Versions: v1.11.3
1 = major, 11 = minor, 3 
Alpha > beta > release

## Cluster upgrade 
K8s components can have different versions.
None component should be at a version higher then kube-apiserver.
Controller manager and kube-scheduler can be 1 version lower. Kubelet and kube-proxy can be two versions lower.
Kubectl can be 1 higher or lower.
Kubernetes supports only the three highest versions.
Upgrade should be done one minor version at the time. v1.10 > v1.11 > v1.12

First upgrade master node and then the worker nodes. When master node goes down the workers nodes keeps working.
Strategy: one node at the time. Add node with newer version and remove old node.

With kubeadm:
Gives the current version and available version. And the manually upgraded tasks after upgrade.
```bash
kubeadm upgrade plan
```
First upgrade kubeadm then the kubernetes components.
```bash
kubeadm upgrade apply
```
The kubectl get nodes gives the version of kubelet, not api server. 
```bash
apt-get upgrade -y kubeadm=1.12.0-00
```
```bash
apt-get upgrade -y kubelet=1.12.0-00
```
```bash
kubeadm upgrade node config --kubelet-version v1.12.0
```
```bash
systemctl restart kubelet
```
Get distribution version:
```bash
cat /etc/*release*
```

## Backup and restore methods
Backup candidates: resource configuration, etcd cluster, persistent volumes.
Resource configs:
```bash
kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml
```
Velero is a backup tool voor kubernetes

ETCD cluster:
All the data from etcd is stored this is set in etcd.service:
--data-dir=/var/lib/etcd. This folder can be used for backup.
ETCD also has snapshot function:
```bash
ETCDCTL_API=3 etcdctl \
    snapshot save snapshot.db
```
```bash
ETCDCTL_API=3 etcdctl \
    snapshot status snapshot.db
```
Restore:
Stop the kube-apiserver
```bash
service kube-apiserver stop
```
restore snapshot
```bash
ETCDCTL_API=3 etcdctl \
    snapshot restore snapshot.db \
    --data-dir /var/lib/etcd-from-backup
```
This create a new data directory, this must been changed in etcd.service
```bash
vi /etc/kubernetes/manifests/etcd.yaml 
```
Give the new data directory the right owner:
```bash
chown -R etcd:etcd /var/lib/etcd-data-new
```


Now reload service daemon
```bash
systemctl daemon-reload
```
And restart etcd
```bash
service etcd restart
service kube-apiserver start
```

```bash
 ETCDCTL_API=3 etcdctl snapshot save /opt/snapshot-pre-boot.db --endpoints 127.0.0.1:2379 \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt
```
Find etcd:
```bash
kubectl get pods -n kube-system  | grep etcd

ls /etc/kubernetes/manifests/ | grep -i etcd

ps -ef | grep etcd

kubectl -n kube-system describe pod kube-apiserver-cluster2-controlplane 

vi /etc/systemd/system/etcd.service 
```
You must verify your work yourself. For example, if the question is to create a pod with a specific image, you must run the kubectl describe pod command to verify the pod is created with the correct name and correct image.



