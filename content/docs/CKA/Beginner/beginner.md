## Networking 101
- IP address is assigned to a POD.
Eisen die k8s stelt aan netwerk, dit moet zelf geregeld worden.
- All containers/Pods can communicatie to one antoher without NAT
ALl nodes can communicate with all containers and vice-versa withoud NAT.

# Kubernetes concepts
Required fields yaml file: apiVersion, kind, metadata, spec.
Kind and services: pod v1, Service v1, ReplicaSet apps/v1, Deployment apps/v1.
Labels under metadata can anything you want. 

## PODS
kubectl run nginx --image=nginx

## Replicasets
Run multiple pods of the same spec:
### Labels en Selectors
Labels en selectors is used to connect replicaset to pods.
mathcLabels type: frontend and labels: type: frontend
MatchLabels is replicaset and labels is pod. If there al ready three pods with type frontend en then replicaset is created if the replicas is the same as the already pods then nothing happend but replica sets keep monitoring.

With kubectl scale you can scale up the replicas.
kubectl replace -f replicaset-definition.yaml
kubectl scale --replicas=6 -f replicaset-definition.yaml

## Deployment
Deploymont can upgrade, rolling updates, roll-back changes, pauze --> make changes and play.
Deployment is higher solution and uses replicasets.

### Update and rollback
When creating a deployment it triggers a rollout. A rollout create a new rollout revision. 
When de container version is upgraded, a new rollout and deployment revision is created. 
With this you can go back to older rollout.
kubectl rollout status deployment/myapp-deployment
kubectl rollout history deployment/myapp-deployment
kubectl rollout undo deployment/myapp-deployment
kubectl set image deployment/myapp-deployment nginx=nginx:1.9.4


#### Deployment strategy
- recreate: destory all old pods en create new pods with the newer image. (not default, down time)
- rolling update: bring down 1 pod and create new pod; pod by pod (default)
Apply this with kubectl apply
When upgrading with deployment there is created a new replicaset.