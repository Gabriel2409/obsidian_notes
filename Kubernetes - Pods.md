#k8s 
## Introduction

- Smallest applicative unit in kubernetes
- Group of container in same isolation context
- Share network stack and volumes
- Dedicated ip address, no NAT (network address translation) for communication between pods

- An app consists of several specification of pods.
- Each specification corresponds to a microservice
- Horizontal scaling with nb of replica of a pod

- Pods can be created manually but are usually created in a ReplicaSet inside a Deployment
- They are exposed in the cluster or to the outside via a Service

## Lifecycle

```yaml
# example pod specification
# www.yaml
apiVersion: v1
kind: Pod
metadata:
  name: www
spec:
  containers:
    - name: nginx
      image: nginx:1.12.2
```

- launch a pod from a file: `kubectl apply -f <pod-specification.yaml>`
- launch a pod directly from an image: `kubectl run <pod-name> --image <image-name>`
- list pods: `kubectl get pods`
- describe a pod: `kubectl describe pod <pod_name>`
- logs of a pod: `kubectl logs <pod-name> [-c <container-name>]` (no need to specify container-name if pod has only one container)
- launch a command in a pod: `kubectl exec <pod-name> [-c <container-name>] -- <command>`
  - ex: interactive shell: `kubectl exec -it <pod-name> -- /bin/bash`
- forward port of www pod to host machine: `kubectl port-forward www 8080:80`
- delete a pod: `kubectl delete pod <pod-name>`

Note: generate a pod specification: `kubectl run db --image mongo:4.0 --dry-run=client -o yaml`
--dry-run simulates the resource creation with 2 possible options:

- client: resource not sent to server API
- server: resource set but not persisted to server API

## Scheduling

- selection of the node where a pod is deployed
- done by kube-scheduler component

Example:

```bash
kubect run www --image nginx:1.16-alpine --restart=Never
kubectl describe pod www
# the output shows that the default scheduler assigned default/www to a node
```

Here the scheduler was able to select a node in our cluster. It is also possible to
add constraints to help the scheduler select a node

### Assigning pods to node without a scheduler

If we deploy our own cluster without the kube-scheduler, the pod will remain in a pending
state. To assign the pod to a node, there are two possibilities:

1.add a nodeName property in the spec of the pod:

```yaml
..
spec:
  nodeName: <mynode>
```

However, this can only be done before pod creating. A pod with this spec will effectively bypass the kube-scheduler.

2. Assign it yourself
   To assign a pending pod to a node, we must use a binding object:

```yaml
apiVersion: v1
kind: Binding
metadata:
  name: <binding-name>
target:
  apiVersion: v1
  kind: Node
  name: <node-name>
```

then do a post request to the binding api: `curl --header "Content-Type:application/json --request POST --data '{<json>}' http://$SERVER/api/v1/namespaces/default/pods/<pod-name>/binding`
where `<json>` represents the binding object in a json format.
This effectively mimicks what the kube-scheduler does for us.

### taints and toleration

A taint is added on a node and is repulsive. For a pod to be scheduled on this node,
it must tolerate this taint

```yaml
# example: on a master node, there is a taint to prevent scheduling
$ kubectl get no master -o yaml
...
spec:
  taints:
    - effect: NoSchedule
      key: node-role.kubernetes.io/master
```

Now if I want to deploy a pod on this node, in the pod specification file, i put the
same key and same effect as the taint.

To manually add a taint to a node: `kubectl taint nodes <node-name> <key>=<value>:<taint-effect>`
To remove a taint, `kubectl taint nodes <node-name> <key> -`

Note: `<key>` and `<value>` can be anything but `<taint-effect>` is either:

- `NoSchedule`: pods who don't have the toleration will not be scheduled
- `PreferNoSchedule`: pods can still be scheduled here but only if necessary
- `NoExecute`: pods that are already on the node but that don't have the toleration will be killed

For a pod to be able to be scheduled on node above, it must have the following toleration (note the quotes):

```yaml
..
spec:
  tolerations:
    - effect: "<taint-effec>"
      key: "<key>"
      operator: "Equal"
      value: "<value>"
```

Note: even if a pod has a toleration to a taint, it doesn't guarantee a schedule on said node with the taint.
Taint and tolerations are only a repulsive strategy

### nodeSelector: schedule a pod on a node with a specific label

```bash
# add label on a node
kubectl label nodes <node-name> disktype=ssd
# see node with specific selector
kubectl get no --selector <key>=<value>
```

- then in the file specification of the pod:

```yaml

..
kind: Pod
spec:
  containers:
    - ...
  nodeSelector:
    disktype: ssd
```

Now when creating the pod, the scheduler will only use nodes that have the correct label.
If no node correspond, it will fail

### nodeAffinity

- allows to schedule pods on certain nodes only
- applied on node labels
- more granular than nodeSelector
- List of Operators: `In`, `NotIn`, `Exists`, `DoesNotExit`, `Gt`, `Lt`
- different rules:
  - hard constraint, not deployed on failure: `requiredDuringSchedulingIgnoredDuringExecution`
  - soft constraint, lets selector decide which pod to use on failure: `preferredDuringSchedulingIgnoredDuringExecution`

File specification example:

```yaml
..
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: kubernetes.io/e2e-az-name
                operator: In
                values:
                  - e2e-az1
                  - e2e-az2
      preferredDuringSchedulingIgnoredDuringExecution:
        - preferencee:
          matchExpressions:
            - key: disktype
              operator: In
              values:
                - ssd
```

Note: the spec is hard to remember but you can run `kubectl explain po.spec.affinity.nodeAffinity --recursive=true` to retrieve the structure

Note 2: what can be done with nodeSelector can be rewritten with nodeAffinity

```yml
# with nodeSelector
spec:
  containers:
  - name: my-container
    image: my-image
  nodeSelector:
    environment: production

# with nodeAffinity
spec:
  containers:
  - name: my-container
    image: my-image
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: environment
            operator: In
            values:
            - production
```

### podAffinity / podAntiAffinity

- allows to schedule pods based on labels of other pods
- different rules: `requiredDuringSchedulingIgnoredDuringExecution` and `preferredDuringSchedulingIgnoredDuringExecution` (same as nodeAffinity)
- key `topologyKey` can be used to specify hostname, region, az, ... and specifies where the constraint must be applied. More generally, this key allows you to express constraints based on node topology domains, ensuring that pods are distributed or co-located in a way that aligns with your cluster's infrastructure setup. 

File specification example where pods will be preferrably scheduled on same availability zones.

```yaml
..
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
         matchExpressions:
            - key: security
              operator: In
              values:
                - S1
        topologyKey: failure-domain.beta.kubernetes.io/zone
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - podAffinityTerm:
            labelSelector:
              matchExpressions:
                - key: security
                  operator: In
                  values:
                    - S2
            topologyKey: kubernetes.io/hostname
```

### Resource allocation

in the pod specification

```yaml
..
spec:
  containers:
    - name: ..
      ..
      resources:
        requests:
          memory: "64Mi"
          cpu: "250m"
        limits:
          memory: "128Mi"
          cpu: "500m"
```

## Static pods

If the node containing kubelet is on its own (not part of a k8s cluster),
it can still create static pods. Note that because there is no api-server in this case,
you must specify the pod-manifest path containing the yaml files when creating the kubelet service.
Once the pods have been created, you can see them with `docker ps` (there is no kubectl)

Now how does it work in the cluster? In fact kubelet can create pods from different sources:

- through the pod definition file
- through an http api endpoint: the kube-api server provides input to kubelet

Note that even if kubelet creates a static pod, it creates a mirror object in the api-server.
You can view details of the pods with kubectl but can not delete or edit it like usual pods.
Note that the node name is automatically appended to the name of the pod

Usecase: to create a cluster from scratch,

- install kubelet on all the master nodes
- create a pod definition file for all the control plane components (kube-apiserver, controller-manager, etcd,... )
- place these files in the designated manifest folder

That way if these services crash, kubelet starts them again !
That is how kubeadm does it. That's why when listing the pods, you can see the different
control plane components in the kubeadm tool

Check what controls the pod with `kubectl describe po -n kube-system | grep "Controlled By"`
If it starts with Node/... then it is a static pod

Note: to check kubelet config for a given node, do `kubectl proxy` and then go to
`http://localhost:8001/api/v1/nodes/<node-name>/proxy/configz`
ALternatively, look at /var/lib/kubelet/config.yaml

## Multiple schedulers

Kubernetes is extensible. It is possible to use custom schedulers instead of the standard
kube-scheduler.
It is also possible to use multiple schedulers. For example, we can have one application
that uses a specific scheduler while the others use the standard kube-scheduler

Now to configure another scheduler, we can create a new yaml file in the manifests folder and
use the property of kubelet that will deploy it as a static pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-custom-scheduler
  namespace: kube-system
spec:
  containers:
    - image: k8s.gcr.io/kube-scheduler-amd64:v1.11.3
      name: custom-kube-scheduler
      command:
        - kube-scheduler
        - --address 127.0.0.1
        - --kubeconfig=/etc/kubernetes/scheduler.conf
        - --leader-elect=true # important in case of multiple schedulers
        - --scheduler-name=my-custom-scheduler
        - --lock-object-name=my-custom-scheduler # important when voting
```

Now in a pod yaml file:

```yaml
---
spec:
  schedulerName: my-custom-scheduler
```
