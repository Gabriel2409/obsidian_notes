#k8s

- Namespaces are scopes for pods, services, deployments,...
- allows to share a cluster (for ex team, project, client, ...)
- 3 namespaces by default: default, kube-public and kube-system
- resources are created in default if non specified
- see namespaces: `kubectl get namespace`
- create namespace directly: `kubectl create namespace <name>`
- create namespace from spec: `kubectl create -f <namespacename.yaml>`
- delete namespace: `kubectl delete namespace <name>`

example spec:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: development
  labels:
    name: development
```

- to put a pod in a namespace:

  - method 1: in the specification:

  ```yaml
  ..
  metadata:
    ..
    namespace: <namespace_name>
  ```

  - method 2: `kubectl run <pod_name> --namespace <namespace_name> --image <image_name>`

Now, when we list the pods, to see it: `kubectl get po --namespace=<namespace_name>` or `kubectl get-po --all-namespaces`
If namespace is not specified, default is used

Note that we can specify namespaces for other resources such as deployments.

- quotas: `kubectl apply -f <quota.yaml> --namespace <namespacename>`
- get the resource quotas: `kubectl get resourcequota`

```yaml
#quota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: quota
spec:
  hard:
    requests.cpu: '1' #cannot ask for more than 1 cpu
    requests.memory: 1Gi
    limits.cpu: '2' #cannot use more than 2 cpu
    limits.memory: 2Gi
```

- limitRange: `kubectl apply -f <limitrange.yaml> --namespace <namespacename>`

```yaml
#limitrange.yaml

apiVersion: v1
kind: LimitRange
metadata:
  name: memory-limit-range
spec:
  limits:
    - default:
        memory: 128M
      defaultRequest:
        memory: 64M
      max:
        memory: 256M
      type: Container
```

- Now when creating a pod,
  - if resource and requests are not specified, they will use the limitRange default
  - if specified and correct, will use, will use the ones specified
  - if specified but outside limitRange, will throw error