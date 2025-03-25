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
Kustomize transfomers can transform values in config file.
Common tansformers:
- commonLabel: adds a label to all Kubernetes resources
- namePrefix/Suffix: adds a common prefix-suffic to all resource names
- Namespace: adds a common namespace to all resources
- commonAnnotations: adds an annotation to all resources

**Image transformer**
Modify the image with kustomize. in kustomization file:
```yaml
nginx:
    - name: nginx (imagename)
      newName: haproxy
```
This code above search for every container that uses the image nginx and replace it with haproxy
Other option is to add a tag or together
```yaml
nginx:
    - name: nginx (imagename)
      newTag: 2.4
```
## Patches
Kustomize patches provide another methode to modifying Kubernetes configs.
Unlike common transformers patches provide a more surgical approach to targeting one or more specific sections in a kubernetes resource.
To create a patch 3 parameters must be provided
- Operation type: add/remove/replace
- target what resource should this patch be applied on
- value: what is the value that will either be replaced or added with
Json 6902 patch
```yaml
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: replace
        path: /spec/replicas
        value: 5
```
strategic merge path (only define the spec you wanna config)
```yaml
patches:
  - patch: |-
      apiVer
      kind
      metad
        name
      spec:
        repl
```