# Kubernetes Scripts
A collection of scripts for various tasks in [Kubernetes](https://kubernetes.io/).

## Usage
Each script has a `usage` function. See usage with
```shell script
$ <script> --help
```

## Scripts
* [findEmptyNamespaces.sh](findEmptyNamespaces.sh): Loop over all namespaces in a cluster and find empty ones.
* [getPodsTopCSV.sh](getPodsTopCSV.sh): Get a pod's cpu and memory usage (optionally per container) written as CSV formatted file.
* [getResourcesCSV.sh](getResourcesCSV.sh): Get all pods resources requests and limits per container in a CSV format with values normalized. 
CSV format is very automation friendly and is great for pasting in an Excel or Google sheet for further processing.
* [getRestartingPods.sh](getRestartingPods.sh): Get all pods (all or single namespace) that have restarts detected in one or more containers. Formatted in CSV.
* [podReady](podReady.sh): Simple script to check if pod is really ready. Check status is 'Running' and that all containers are ready.
Returns 0 if ready. Returns 1 if not ready.

## One liners
### Kubectl
* See all custer nodes load (top)
```shell script
kubectl top nodes
```

* Get cluster events
```shell script
# All cluster
kubectl get events

# Specific namespace events
kubectl get events --namespace=kube-system
```

* Get all cluster nodes IPs and names
```shell script
# Single call to K8s API
kubectl get nodes -o json | grep -A 12 addresses

# A loop for more flexibility
for n in $(kubectl get nodes -o name); do \
  echo -e "\nNode ${n}"; \
  kubectl get ${n} -o json | grep -A 8 addresses; \
done
```

* See all cluster nodes CPU and Memory requests and limits
```shell script
# Option 1
kubectl describe nodes | grep -A 2 -e "^\\s*CPU Requests"

# Option 2 (condensed)
kubectl describe nodes | grep -A 2 -e "^\\s*CPU Requests" | grep -e "%"
``` 

* Get all labels attached to all pods in a namespace
```shell script
for a in $(kubectl get pods -n namespace1 -o name); do \
  echo -e "\nPod ${a}"; \
  kubectl -n namespace1 describe ${a} | awk '/Labels:/,/Annotations/' | sed '/Annotations/d'; \
done
```

* Forward local port to a pod or service
```shell script
# Forward localhost port 8080 to a specific pod exposing port 8080
kubectl port-forward -n namespace1 web 8080:8080

# Forward localhost port 8080 to a specific web service exposing port 80
kubectl port-forward -n namespace1 svc/web 8080:80
```

* A great tool for port forwarding all services in a namespace + adding aliases to `/etc/hosts` is [kubefwd](https://github.com/txn2/kubefwd).
Note that this requires root or sudo to allow temporary editing of `/etc/host`.
```shell script
# Port forward all service in namespace1
kubefwd svc -n namespace1
```

* Extract and decode a secret's value
```shell script
# Get the value of the postgresql password
kubectl get secret -n namespace1 my-postgresql -o jsonpath="{.data.postgres-password}" | base64 --decode
```

* Start a shell in a temporary pod (will terminate once exited)
```shell script
# Ubuntu
kubectl run --generator=run-pod/v1 my-ubuntu --rm -i -t --image ubuntu -- bash

# CentOS
kubectl run --generator=run-pod/v1 my-centos --rm -i -t --image centos:8 -- bash

# Alpine
kubectl run --generator=run-pod/v1 my-alpine --rm -i -t --image alpine:3.10 -- sh

# Busybox
kubectl run --generator=run-pod/v1 my-busybox --rm -i -t --image busybox -- sh
```

* Get list of container images in pods. Useful for listing all running containers in your cluster
```shell script
kubectl get pod --all-namespaces \
    -o=jsonpath='{range .items[*]}{.metadata.namespace}, {.metadata.name}, {.spec.containers[].image}{"\n"}'
```

* Get per node resources requests and limits
```shell script
for n in $(kubectl get no --no-headers | awk '{print $1}'); do echo -e "\n-- ${n} --"; k describe no $n | grep -A 3 "Resource .*Requests .*Limits"; done
```

### Helm
**NOTE:** It is recommended to move to [Helm v3](https://helm.sh/docs/), which does not use tiller anymore.

* Helm v2 on an RBAC enabled cluster. This will give tiller `cluster-admin` role
```shell script
kubectl -n kube-system create sa tiller && \
    kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller && \
    helm init --service-account tiller
```

* View the templates generated by `helm install`. Useful for seeing the actual templates generated by helm before deploying.<br>
Can also be used for deploying the templates generated when cannot use Tiller
```shell script
helm template <chart>
```

* Debug a `helm install`. Useful for seeing the actual values resolved by helm before deploying
```shell script
helm install --debug --dry-run <chart>
```
