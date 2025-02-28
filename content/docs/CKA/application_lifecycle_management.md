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




