# Kubernetes security

## authentication
Two different types;
- user: for user accounts
- service accounts: for bots other processes or services
K8s not manage accounts but rely on external source.
K8s manage serviceaccounts.
All access is managed by the api server. Kube api server authenticate each request.
Auth mechanisms:
- static password file
- static token file
- certificates
- identity services
Basic:
store users and passwords in a csv file. password,username,userid,group
Specify this csv file in the kube-apiserver.service: --basic-auth-file=user-details.csv
Other option is a static token file. --token-auth-file=user-token-details.csv

## TLS
Certificate (public key) | *.crt *.pem | server.crt, server.pem, client.crt, client.pem
Private key: *.key, *-key.pem | server.key, server-key.pem, client.key, client-key.pem

Communications between nodes and services is encrypted. 
Server certificates for servers: each server has it own set of key/certificates. This certificates are that the api server can talk to the server.
Client certificates for clients, this are everything that talks to the api server, also kubelet has client certificate if it talks to the api server. is for the users that interact with the kube api server. Also kube-scheduler, kube-controllermanager, kube-proxy.
Al communication goes by the api server.

In Kubernetes there must be at least 1 certificate authority (CA). But can be more than one. 

### Certificate creation
Openssl to generate certificates.
Generate key:
```bash
openssl genrsa -out ca.key 2048
```
Generate certificate signing request
```bash
openssl req -new -key ca.key subj "/CN=KUBERNETES-CA" -out ca.csr
```
Sign certificates (done by the CA)
```bash
openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
```
The name in subj CN= is the name that you see in the audit from kube api. 
Sign certificates for clients
```bash
openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt
```
You must specifie group in the certificate. System components needs to called SYSTEM:Name SYSTEM:KUBE-PROXY
The kubelet certificates are named to their nodes name.
Kubectl nodes client cert name: system:node:node01

### View certificates details
First it is important to know how the cluster is setup: hard way or kubeadm
Hard way then all the certificates are self created:
```bash
cat /etc/systemd/system/kube-apiserver.service
```
Kubeadm takes care to generate the certificates for you:
```bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```
Kubeadm deploys everything as pod.

See the details of a certificate:
```bash
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
```
See logs:
```bash
journalctl -u etcd.service -l
```
Kubeadm logs:
```bash
kubectl logs etcd-master
```
If kubectl nog working use docker commands:
```bash
docker logs <container id>
```
https://kubernetes.io/docs/setup/best-practices/certificates/
https://github.com/mmumshad/kubernetes-the-hard-way/tree/master/tools

### Certificates API
Create a ca server that is secure. Signing a certificate can only be done now from a ca. Master node can be ca server.
K8s has builtin certificates api to sign certificates and rotate. Now if the admin gets a certificate signing request (CSR), it can send a api request instead of logging in to the server. 
1. The admin create a certificatesigningrequest object. This object can be seen by everyone 
2. Review requests
3. approve requests
4. share certs to users

All certificate related operations is done by the controller manager. Controller manager has multiple managers for this. Signing a certificate is ca.crt and ca.key

Steps:
1. user creates a key 
```bash
openssl genrsa -out jane.key 2048
```
2. Creates a request and sends it to the administrator
```bash
openssl req -new -key jane.key -subj "/CN=jane" -out jane.csr
```
3. The admin creates a certificatesigningrequest object. Under request insert the csr in a base64 format.
4. Now other admins can see the request
```bash
kubectl get csr
```
And approve it with
```bash
kubectl certificate approve jane
```
5. Now k8s creates a certificate. It is in the certificatesigningrequest status object. The certificate here is base64 format.

cat <>.csr | base64 -w 0


## Kubeconfig
When working woth kubectl all the options like --server, --client-key etc should be applied by every request.
Al these values can be set in a config file and select by kubectl get pods --kubeconfig config
Default kubectl looks at $HOME/.kube/config for the config file
Format kubeconfig file:
Three sections: cluster, contexts, users.
Clusters all the cluster you need access to. 
Contexts: connect the user to cluster: user@cluster
Users: all the user accounts you have access to the clusters. This section also contain the certificates
Context can also be applied on a specific namespace. in context you can specify a namespace. 
For certificates specify the full pad. With certificate-authority-data: you can specify the certificate data and not the file.


Kubeconfig is a yaml file kind Config.
```yaml
apiVersion: v1
kind: Config
current-context: (default context)
clusters:
contexts:
users:
```
To see current config
```bash
kubectl config view
```
Change current context
```bash
kubectl config use-context prod-user@production
```

## API groups
The kubernetes api is defined in groups: /metrics, /healthz, /version, /api, /apis, /logs.
Metrics and healthz for health of the cluster
version is for version of the cluster
logs for 3rd party logging tools.

api is core group. Core funcctionaliteity --> v1 --> namespace, pods etc. (api groups)
apis is named group. are more organized --> /apps, /extensions, /networking.k8s.io etc. apps --> v1 --> deployments, replicasets etc (resources)
Within the resources there are actions like list, get, create. (verbs)
core api groups --> resources --> verbs
newer funct would use named group

```bash
curl https://kube-master:6443/api/v1/pods
```
Return all the api groups
```bash
curl http://localhost:6443 -k
```
See al the named groups
```bash
curl http://localhost:6443/apis -k | grep "name"
```
Use the certificates for the above commands. To bypass use a kubectl proxy client it interact between user and kube apiserver
kube proxy is not the same as kubectl proxy

## Authorization
Authorization mechanisms;
node: Kubelet access the api server. This requests are authorized by the node authorizer. Node authorizer authorized certificates with group system:nodes and name system:node:nodename. 
ABAC (attribute based acess control): example users it specify the authorization for users. Define it for each user or group. 
RBAC role based access control: define a rol with a set of permissions and connect group or user to a rol.
Webhook: outsource authorization mechanisme. like open policy agent. 
The other mechanisms are alwaysallow and alwaysdeny. 
Mode are set in the authorization-mode option in the api server. Multiple modes are possible. A request is then ordered in the order defined.

### RBAC
Role based access control
Create a role with role object. In the object each rule has three sections: apiGroups, resources and verbs.
For the core group the apigroups can let blank. All other api groups define the name.
Next link a user to the role with a rolebinding object. This object has two sections: subjects for the user specifications and roleRef for the role specifications. Role and rolebindings are namespaces binded. 
Check access:
```bash
kubectl auth can-i create deployments
```
for a other user
```bash
kubectl auth can-i create deployments --as username --namespace namespace
```

### Cluster roles
Cluster wide resources like node. Resources are grouped as namespaced or cluster scoped.
Cluster scoped are like: nodes, pv, clusterroles, certificatesigningrequests, namespaces etc
Get all namespaced resources: (and for cluster set false)
```bash
kubectl api-resources --namespaced=true
```

To authorize user to cluster wide resources is done by clusterrole and clusterrolebindings. Clusterrole can be namespaced resource so the user can access resource on all namespace
```yaml
- apiGroups:
  - "storage.k8s.io"
  resources:
  - storageclasses
  verbs:
  - "*"
```

### Serviceaccounts
Types of accounts user and serviceaccounts.
Serviceaccounts are used for machines.  Example a account for a application monitoring
When creating a serviceaccount there is created a token. THis token must be used by the external application.
Token is stored as secret object.

If the application is hosted on the cluster, then the token secret can be mounted to the pod. By default there is a service account default, every pod has by default the token for this serviceaccount mounted. Default has very basic privileges. When changing serviceaccount the pods are recreated. To disable the tokenmount: automountServiceAccountToken: false.

Before k8s version 1.22 there was no expiration date. in v1.22  the tokenrequestapi is introduced. These tokens are audience bound, time bound, object bound. Now there is a token requested with a expirationseconds. 1.24 when creating serviceaccount no token and secrets is created. Kubectl create token. 

## Image security
Using private repo:
For docker run docker login private-registry.io
Authentication in a pod can be done secret type docker-registry. 

### Security in Docker
List process on docker host you also see the processes in a container. Docker host is a namespace and a container child namespace.
Default processes in containers run as root user. To change this define --user= in docker run. Better is to define this in Dockerfile when creating.
/usr/include/linux/capability.h
--cap-ad or --cap-drop --privileged

### Security contexts
Security settings set on a pod wil overwrite the security settings on a container.
Under the spec section in pod securityContext, or under the container section. 

