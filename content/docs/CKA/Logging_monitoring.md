# Logging & monitoring

## Monitor cluster components
K8s has no fully builtin monitoring solution.
Solutions are;
Metrics server: one metrics server per cluster, retreive from each node and pod metrics, in memory solution, cannot see history data.
Kubelet has cAdvisor, and cAdvisor is responsible to collect metrics from pods and send to the api server.
Metrics can be see with 
```bash
kubectl top node
```
```bash
kubectl top pod
```

## Managing application logs
To see the logs from a pod application:
```bash
kubectl logs -f <file>
```

When multiple containers in pod:
```bash
kubectl logs -f <filename> <containername>
```
