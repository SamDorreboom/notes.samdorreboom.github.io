# Kustomize
Reuse Kubernetes configs for multiple environment without copying the config files. Only specify the individual specs.
Kustomize works with a Base config and overlays.
Base config is the config that is same on all the environments. In the base config there also default values. This is just a config file. 
In the overlay you can customize the specs that is in the base config.
Overlay file:
```yaml
spec:
    replicas: 1
```
Folder structure:
```shell
k8s/
    Base/
        kustomization.yaml
        nginx-depl.yaml
    Overlays/
        dev/
            kustomization.yaml
            config-map.yaml
        stg/
```

Base + Overlay --> Final manifests
- Kustomize comes built-in with kubectl
- Does not require learning how to use any complex & hard to read templating systems (like helm)
- Every artifact the kustomize uses is plain yaml and can be validated and processed as such.

Check kustomize is installed
```bash
kustomize version --short
```

## Kustomize vs helm
Helm makes use of go templates to allow assigning variables to properties. {{  }} = go templating. In helm create multiple value files, and select the right value file for the environment.
Helm is more than just a tool to customize configurations on a per environment basis. Helm is also a package manager for your app
Helm provides extra features like conditionals, loops, functions and hooks. 
Helm Templates are not valid yaml as they use go templating syntax. Complex templates become hard to read.

## Kustomization file
Kustomize looks for a kustomization file which contains
    - List of all the kubernetes manifests kustomize should manage
    - All of the customizations that should be applied
The kustomize build command combines all the manifests and applies the defined transformations
The kustomize build command does not apply/deploy the kubernetes resources to a cluster.
    The output needs to be redirected to the kubectl apply command

The apiVersion and kind in the file is optionally but adviced.

To run the Kustomization: (does not apply/deply config)
```bash
kustomize build k8s/
```

Example file:
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
    - nginx-deployment.yaml
# Custom
commonLabels:
    company: KodeKloud
```

**Kustomize output**
How to apply the config:
```bash
kustomize build k8s/ | kubectl apply -f -
```
or
```bash
kubectl apply -k k8s/
```
Delete the resources
```bash
kustomize build k8s/ | kubectl delete -f -
```
```bash
kubectl delete -k k8s/
```

**Managing directories**
Config files can be divided in multiple directories. To work with multiple directories just specify under the resources with the directories. 
```yaml
resources:
    - /test/..
    - test2/..
```
Other solution is a kustomization file in every directory. And then in the root kustomization file only define the directorys. 

## Transfomers
