#k8s

## Creation

A deployment is a more high level resource that deals with an ensemble of identical pods
and makes sure that there is a specific number of nodes that run in the cluster.

If one of the replicas crash, the deployment can restart one.
A deployment deals with pod lifecycle:

- creation / deletion
- scaling
- rollout / rollback

ReplicaSet = An intermediate resource between deployment and pod whose role is to make
sure pod is working as intended and takes corrective measures if needed

Note that we rarely manipulate ReplicaSet directly

example:

```yaml
# deployment

apiVersion: apps/v1 # version is apps/v1 and not just v1
kind: Deployment
metadata:
  name: www
spec:
  replicas: 5
  selector:
    matchLabels:
      app: www # selector must match pod label

  template: # contains the pod specification
    metadata:
      labels:
        app: www
    spec:
      containers:
        - name: www
          image: nginx:1.16
```

- run from spec: `kubectl apply -f <deploy-spec.yaml>`
- imperative cmd: `kubectl create deploy <name> --image <image>`
  Note: `--dry-run=client -o yaml` to have specification as usual

## Update

- update strategy. By default k8s uses a rolling update strategy. We can specify
  extra parameters in the spec.

```yaml
# deployment
..
spec:
  ..
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1 # how many extra replicas I can have
      maxUnavailable: 0 # how many replicas can be removed at most
..
```

In the example above, for the update, we start with 3 nodes at v1, then we add a node at v2,
then remove a v1, ... until we only have 3 v2s.
The rolling update will be triggered each time the pod specification in the deployment is
modified. So for example, we modify the spec and run a `kubectl apply -f <deploy_spec>`.
k8s will then run the rolling update process

- we can also update a deployment imperatively: `kubectl set image deploy/www www=nginx:1.18 --record`
  (here the deployment is called www and the container to modify is called www).

- list history of deployment: `kubectl rollout history deploy/www`:
  - CHANGE-CAUSE column contains the command that brought to current version if `--record` flag was set when doing the update
  - 10 revisions by default -> modified by property `.spec.revisionHistoryLimit` of deployment
- revert: `kubectl rollout undo deploy/www` with optional `--to-revision=X`

- we can also force an update with `kubectl rollout restart deploy/www`: the pods of the deployments are stopped and new ones are launched

## Scaling

- scale a deployment: `kubectl scale deploy <name> --replicas=3`

We can also use a HPA
A HorizontalPodAutoscaler (HPA for short) automatically updates a workload resource
(such as a Deployment or StatefulSet)

from a spec:

```yaml
# hpa-spec.yaml

apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: www
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: www
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50
```

and `kubectl apply -f <hpa-spec.yaml>`

or imperatively: `kubectl autoscale deploy www --min=2 --max=10 --cpu-percent=50`

NOTE: we will have an error if the deployment does not exist

- we can access the hpa with `kubectl get hpa`

- Note that for HPA to work correctly:
  - a service exposing the pods must be running
  - resources must be specified in the container spec (not sure about that)
  - metrics server must be installed:
    - check that it is installed: `kubectl get po -n kube-system -l k8s-app=metrics-server`
    - install it if necessary: `kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml`
    - note that for minikube, you must do `minikube addons enable metrics-server` instead
