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




