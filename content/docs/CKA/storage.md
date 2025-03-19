# Storage K8s

## Docker storage
File system:
/var/lib/docker: aufs, containers, image, volumes
Layered architecture: a image is build with layers. From ubuntu --> run apt etc.
When building a new image and a layer is not changed it won't rebuild that layer. 
Layers are read only. The container layer the last layer is read write.

Volume in Docker is to store the read write layer from the container = volume mounting it create a volume in the volume directory of docker
Bind mount binds a directory from any directory.
docker run --mount type=bind,source=,target=
Storage drivers; aufs, zfs, btrfs, device mapper, overlay, overlay2. For ubuntu aufs is default.

### Volume driver plugins docker
Volumes are handled by volume driver plugins. Default plugin is local. 
Other are azure file storage, convoy, digital ocean block storage etc.

## Container storage interface
In the past k8s only used container runtime docker. Now it use more like rkt, cri-o.
Container runtime interface is a standard on how to communicate with a container runtime. Every runtime that follows the cri is can work with Kubernetes. 
For networking there is CNI. 

For storage there is CSI (controller storage interface). Kubernetes uses CSI. CSI works with RPC's. For like createvolume. 
Github.com/container-storage-interface

## Volumes
Containers are meant to be to last a short periode of time. Same for the data in the container, it is destroyed when container is recreated. In a volume is data that need to persist.
In kubernetes can a volume connect to a pod. Volume can on the node host or outside cluster like aws.

### Persistent volumes
Is a cluster wide pool of storage volumes configured by a admin, used by users. Users can select a persisten volume out of this pool with a pvc. 
Access modes: readonlymany, readwriteonce, readwritemany. 
If the pvc is deleted the persistenvolumereclaimpolicy will decide what happens with the pv. Default is retain, it wont be available for other pvc.

### Persistent volume claims
PV and pvc are two different objects. Kubernetes binds the persistentvolumes to a pvc based on the request and properties set on the claim. During binding process that has sufficient capacity, access modes, volume modes, storage class.

If there are more possible matches for a claim, you can use labels to bind to specific to a volume. And a pvc can be bind to a bigger pv if there are no other pv's. If no pv are available the pvc keeps in pending state. 


## Storage class
Static provisioning: For every pv we need to manually create a cloud volume and create a pv.
Dynamic provisioning
With storage class you can define a provisioner like google that automatic provision google cloud storage. Now only a SC and PVC is needed, the pv is created automatically. IN the pvc define a storageClassname. 
