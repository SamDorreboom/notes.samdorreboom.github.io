# Application lifecycle management

## Rolling updates and Rollbacks
How update image:
1. With a deployment file, edit the image version and run kubectl apply for upgrading.
2. 
```bash 
kubectl set image deployment/myapp \
    nginx-container=nginx:1.9.1
```
## Rollout and versioning
When creating a deployment it trigger a rollout, a new rollout creates a new deployment revision. 
When the image version is upgraded and a new deployment revision is created. By this rollback function.
```bash
kubectl rollout status deployment/myapp-deployment
```
```bash
kubectl rollout history deployment/myapp
```
When upgrading application the deployment creates a new replicaset and in the new replicaset comes the new version. Add a rollback the new replicaset is destroyed and the old replicaset started up.
```bash
kubectl rollout undo deployment/myapp
```

## Deployment strategy
1. Recreate: Upgrade to new version by destory all instance and then create new gives downtime. Kubernetes changes replicas to 0 and then to the defined replicas. 
2. Rolling update: destory and recreate one by one (default). 


## Application commands & arguments
Define other CMD in docker image: containers --> args: ["10"]
Define other entrypoint in Docker image: containers --> command ["sleep2.0"]

### Environments variables
containers --> - --> env: --> - --> name, value
Andere manieren van enviroments is met ConfigMap of Secrets. dan wordt value: valuefrom --> configmapkeyref, secretkeyref --> name, key
#### Configmaps
In a configmap object you can define variables which can be used in a pod.
1. Create configmap 
2. inject into pod
Imperative: kubectl create configmap <configname> --from-literal=APP_COLOR=blue , kubectl create configmap app-config --from-file=lorem.properties
Declarative: kubectl apply -f <definition file>
#### Secrets
Secrets objects are to store secrets in kubernetes. The secrets need to be saved in base64 format when using declarative.
Imperative: kubectl create secret generic <secretname> --from-literal=<key>=<value>, --from-file=<pathtofile>
echo -n 'mysql' | base64 (--decode)
Configure it in a pod: spec: --> containers --> - --> envFrom: --> - secretRef: --> name: <secretname>
Secret can also be a volume in a pod. 
Secrets are not encrypted also not in ETCD! But it is possible to encrypt in etcd it in rest with object encryptionconfiguration (encrypting data at rest).
Anyone able to create pods/deployments in the same namespace can access the secrets.

## Secret store CSI driver
https://www.youtube.com/watch?v=MTnQW9MxnRI
Secret store driver is for managing secrets. 
Secrets store csi driver synchronizes secrets from external APIs and mounts them into containers as volumes.
Allows you to manage secrets in a central place like hashicorp vault or aws secrets manager.
! Dont need to check secrets into Git
It uses a SecretProviderClass object.

Difference with ESO & Sealed Secrets:
- No longer store credentials in K8s secrets. It pull it dynamically.
    - Minimize attack surface as much as possible.
    - Great from a compliance and regualtory perspectives

## Multi-container pod
Multi container is used to have like log agent and web server in one pod. 
Same lifecycle, same network space, same storage volumes.
As simple it is just add a container under containers. 

### init containers
When a POD is first created the initContainer is run, 
and the process in the initContainer must run to a completion before the real container hosting the application starts.
You can configure multiple such initContainers as well, like how we did for multi-containers pod. In that case, each init container is run one at a time in sequential order.
If any of the initContainers fail to complete, Kubernetes restarts the Pod repeatedly until the Init Container succeeds.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'git clone  ;']
```

## Autoscaling
Different type: horizontale pod autoscaling (hpa), vertical pod autoscaling (vpa)
Vertical means add more resource to existing application. 
Horizontale means add more instances
In kubernetes you can scale workloads (applications) and scaling cluster infra (node).
This can be done manual or automated.
Automated: cluster autoscaler, horizontale pod autoscaler (hpa), vertical pod autoscaler (vpa)


