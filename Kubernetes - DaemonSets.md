#k8s 
## DaemonSet

- Used to make sure a pod run on each machine
- Often used for system/supervision app which need to deploy an agent on each machine:
  - log collection
  - monitoring
  - storage
    Spec looks like deployment.
    Ex:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: <name>
spec:
  template: <pod-spec>
```

### Example: use of weavescope

Weavescope is an app used to see the processes, containers and all running in the cluster

- Retrieve the spec with: `curl -sSL -o scope.yaml "https://cloud.weave.works/k8s/scope.yaml?k8s-version=$(kubectl version | base64 | tr -d '\n')"`
- Create the resources: `kubectl apply -f scope.yaml`
- Resources are created in the namespace weave: we can list them with: `kubectl get all -n weave`
- We can see it contains a Deployment called weave-scope-app (target port 4040) and a DaemonSet which creates weave-scope-agent pods
- port forward the corresponding pod: `kubectl port forward -n weave pod/weave-scope-app-6996cfd49b-48zz4 4040`
- app is available at `localhost:4040`