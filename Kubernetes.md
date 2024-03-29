#tuto #todo #k8s

# kubectl tricks

## Basics

- install autocompletion for kubectl: `sudo apt install bash-completion` and `source <(kubectl completion bash)`

- Two approaches:

  - imperative approach: handle resources in cli:
    - create pod: `kubectl run <podname> --image <imagename> OPTIONS`
    - create any resource: `kubectl create <resourcetype> <resourcename> OPTIONS`
    - list resource: `kubectl get <resource>`
    - delete resource: `kubectl delete <resource-type> <resource>`
    - edit configuration of resource on the fly: `kubectl edit <resource-type> <resource>`
    - patch specific fields: `kubectl patch <resource-type> <resource> -p '{"<key>": {"<subkey>": <value>}}'`
  - declarative: use a spec:
    - create resources: `kubectl apply -f <specification-file-or-folder>`
    - delete resources: `kubectl delete -f <specification-file-or-folder>`

- Possibility to add flag `--dry-run=client` or `--dry-run=server` to prevent creation of resource (if server is specified, request is passed to server API)
- See output with `-o yaml`, `-o json`, or `-o jsonpath` (can be used with `dry-run` to see resulting spec)

- describe a resource: `kubectl describe <resourcetype> <resourcename>`
- see all api resources and their abbreviation: `kubectl api-resources`
- infos about commands: `kubectl explain <resourcename>` (can be nested, for ex `kubectl explain pod.metadata.uid`)

## Cool tricks

- get image of pod www: `kubectl get po/www -o jsonpath='{.spec.containers[0].image}'`
- get image id of all pods in kube-system namespace: `kubectl get po -n kube-system -o jsonpath='{range.items[*]}{.status.containerStatuses[0].imageID}{"\n"}{end}'`
- custom columns: `kubectl get po -o custom-columns='NAME:metadata.name,IMAGES:spec.containers[*].image'`

# Installation

## kubectl

- kubectl is kubernetes cli

```bash
VERSION=$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)
curl -LO https://storage.googleapis.com/kubernetes-release/release/$VERSION/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

## Krew

Krew is kubectl plugin manager: https://github.com/kubernetes-sigs/krew/
Installation instructions: https://krew.sigs.k8s.io/docs/user-guide/setup/install/

```bash
(
  set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
  KREW="krew-${OS}_${ARCH}" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" &&
  tar zxvf "${KREW}.tar.gz" &&
  ./"${KREW}" install krew
)
```

Then in .bashrc:

```
export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
```

## kubectx and kubens

kubectx is a tool to switch between contexts (clusters) on kubectl faster.
kubens is a tool to switch between Kubernetes namespaces (and configure them for kubectl) easily.
https://github.com/ahmetb/kubectx

To install them with Krew:

```
kubectl krew install ctx
kubectl krew install ns
```

- switch context:
- kubectl: `kubectl config use-context <context-name>`
- plugin: `kubectl ctx <context-name>` (replace cmd with `kubectx` if not installed with Krew)

- switch namespace:
  - kubectl: `kubectl config set-context $(kubectl config current-context) --namespace=<name>`
  - plugin: `kubectl ns <name>` (replace cmd with `kubens` if not installed with Krew)

## multipass

IMPORTANT NOTE FOR WSL. When creating multipass machines and installing kubernetes on them,
kubectl is not able to access them because of network rules.
To solve this issue,

- we can launch this from powershell as admin: `Get-NetIPInterface | where {$_.InterfaceAlias -eq 'vEthernet (WSL)' -or $_.InterfaceAlias -eq 'vEthernet (Default Switch)'} | Set-NetIPInterface -Forwarding Enabled` (see https://stackoverflow.com/questions/65716797/cant-ping-ubuntu-vm-from-wsl2-ubuntu).
- we can also do some port forwarding: see https://pypi.org/project/WSL-Port-Forwarding/

NOTE: I am still unclear if we need both of them or just the first

- Install multipass: https://multipass.run/ : allows to easily launch a VM with either hyperV or virtual box
  To use it on WSL, it must be installed on windows, not on linux. And then, we can create
  an alias to use it directly on WSL: for ex: `alias multipass="/mnt/c/'Program Files'/Multipass/bin/multipass.exe"`
- set driver to hyper V: `multipass set local.driver=hyperv` or virtualbox: `multipass set local.driver=virtualbox`

- IMPORTANT: to use the mount features in WSL, you must be in a folder accessible by windows, such as `/mnt/c/...`
- enable mounting with `multipass set local.privileged-mounts=true`
- create a vm:

  - basic : `multipass launch node1`
  - with parameters: ` multipass launch -n node2 -c 2 -m 3G -d 10G` (2 cores, 3G ram, 10G disk size)

- start/stop/delete vm: `multipass start node1` / `multipass stop node1` / `multipass delete -p node1`
- get infos: `multipass infos node2`
- list vms: `multipass list`
- launch a shell: `multipass shell node1`
- launch a command on the node, for ex:
  - install docker: `multipass exec node1 -- /bin/bash -c "curl -sSL https://get.docker.com | sh"` (here, it launches the bash shell and execute the command between quotes. It is the same as getting into the sell and executing the command between quotes)
  - check that docker is correctly installed: `multipass exec node1 -- sudo docker version`
- copy file from local machine: `multipass transfer /tmp/test/hello node1:/tmp/hello` (both ways work)
- mount a folder: `multipass mount ./testfolder node1:/usr/share/test`.
  Note that if you want to use an absolute path: `multipass mount $(wslpath -w /mnt/d/projects/kubernetes-tuto/test2/) node1:/usr/share/test`
- unmount a folder: `multipass umount node1:/usr/share/test` or unmount all : `multipass umount node1`

# Create local clusters

Summarizes multiple ways to create local clusters

## Minikube

- Install minikube:

```bash
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x minikube
sudo mv minikube /usr/local/bin/
```

- to start minikube: `minikube start` (docker desktop needs to be running for this to work on wsl)
- to start minikube with 3 nodes while specifying docker driver: `minikube start --driver docker --nodes 3`
- see nodes: `kubectl get no`
- destroy cluster: `minikube delete`

## Kind

Kind (Kubernetes in docker) allows to deploy a kubernetes cluster such that each node runs in a docker container

- Install with go: `go install sigs.k8s.io/kind@v0.14.0` (replace v0.14.0 with stable version)
- Create a cluster: `kind create cluster --name k8s`
- We can see a container was created when running `docker container ls`: this container runs kubernetes
- Note that kind automatically creates a context and sets it as current context: `kubectl config get-contexts`
- We can list the nodes of the cluster: `kubectl get nodes`

To create clusters with several nodes, we need a yaml config file:

```yaml
# kindconfig.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
```

We create the cluster with: `kind create cluster --name k8s-2 --config kindconfig.yaml`

- Get the clusters with `kind get clusters`
- Cleanup: `kind delete cluster --name k8s`

## MicroK8s

Easy to use solution very good for local dev. Can also be configured with several nodes and used in prod

Note: In this example, install is done on VM:

- create VM: `multipass launch --name microk8s --mem 4G`
- install microk8s on VM: `multipass exec microk8s -- sudo snap install microk8s --classic`
- get config in yaml on local machine: `multipass exec microk8s -- sudo microk8s.config > microk8s.yaml`
- set KUBECONFIG var: `export KUBECONFIG=$PWD/microk8s.yaml`
- see nodes: `kubectl get no` (notes: see multipass notes on wsl)
- run `microk8s status` inside the vm to see all enabled and disabled addons
- before using cluster and deploying app, we can install CodeDNS via dns addon: `microk8s enable dns`

## K3s

Very light kubernetes distribution. Same as above, installed on a VM

- lauch: `multipass launch --name k3s-1`
- get ip: `IP=$(multipass info k3s-1 | grep IP | sed "s/\r//" | awk '{print $2}')` (not that IP will change if you restart host machine)
- install : `multipass exec k3s-1 -- bash -c "curl -sfL https://get.k3s.io | sh -"`
- create config with correct IP: `multipass exec k3s-1 sudo cat /etc/rancher/k3s/k3s.yaml | sed "s/127.0.0.1/$IP/" > k3s.cfg`
- set kubeconfig var: `export KUBECONFIG=$PWD/k3s.cfg`
- get nodes: `kubectl get nodes` (notes: see multipass notes on wsl)

- create other vms:

```bash
for node in k3s-2 k3s-3;do
  multipass launch -n $node
done
```

- retrieve token from master node: `TOKEN=$(multipass exec k3s-1 sudo cat /var/lib/rancher/k3s/server/node-token)`
- Join the nodes:

```bash
# Join node2

$ multipass exec k3s-2 -- \
bash -c "curl -sfL https://get.k3s.io | K3S_URL=\"https://$IP:6443\" K3S_TOKEN=\"$TOKEN\" sh -"

# Join node3
$ multipass exec k3s-3 -- \
bash -c "curl -sfL https://get.k3s.io | K3S_URL=\"https://$IP:6443\" K3S_TOKEN=\"$TOKEN\" sh -"

```

## K3d

Can deploy K3s clusters such that each node runs in a docker container. Installed directly
on local maching. Docker desktop must be running on WSL

- install: `curl -s https://raw.githubusercontent.com/rancher/k3d/main/install.sh | bash`
- check version: `k3d version`
- creates a cluster named k3s with 2 workers and 1 server: `k3d cluster create k3s --agents 2`
- more infos: `k3d cluster create --help`
- list containers: `docker ps`: we should see servers and workers
- switch context: `kubectl config use-context k3d-k3s`
- list nodes: `kubectl get nodes`
- delete cluster: `k3d cluster delete k3s`

# Production clusters

## managed solutions

Lots of cloud providers propose solutions to manage clusters for a prod environment:

- GKE: google kubernetes engine
- AKS: Azure Container service
- EKS: Amazon Elastic Container service
- DigitalOcean
- OVH

Using them is straightforward. Creation can be done using the web interface, then we
download the config file and modify KUBECONFIG var to point to the config. Then, we
can see the nodes with `kubectl get nodes`

## install kubernetes without providers

lots of solutions:

- kubeadm:
  - https://kubernetes.io/fr/docs/setup/production-environment/tools/kubeadm/
  - nodes must be provisionned prior to setting up cluster
- kops:
  - https://github.com/kubernetes/kops
  - deals with full cluster lifecycle
- kubespray:
  - https://github.com/kubernetes-sigs/kubespray
  - uses ansible to set up kubernetes
- rancher:
  - https://rancher.com/
  - allows to handle app lifecycle with a catalog
- Docker EE (deploy Swarm and Kubernetes)
- terraform + ansible (provisionning + config)

### details with kubeadm

- first: provision machines (masters, workers)
- install container runtime on the host (for ex install docker on all the nodes)
- install kubeadm tools on all the nodes
- initialize the master server
- create the pod network
- make workers join the master

# Pods

## Introduction

- Smallest applicative unit in kubernetes
- Group of countainer in same isolation context
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
- launch a pod directly from an image: `kubect run <pod-name> --image <image-name>`
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

```
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

```
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

# Services

- expose pods via network rules
- use labels to group pods
- persistant IP address (virtual IP address)
- kube-proxy in charge of load balancing on pods
- different types:
  - ClusterIP: exposition inside cluster
  - NodePort: exposition to outside
  - LoadBalancer: integration with cloud provider
  - ExternalName: associates service to DNS name

## ClusterIP

allows to have interaction inside the cluster

A pod inside a node can interact with the service and the service will forward
to relevant pods inside the cluster (even if not on same node)

```yaml
# pod.yaml - pod specification file
apiVersion: v1
kind: Pod
metadata:
  name: www
  labels:
    app: www
spec:
  containers:
    - name: nginx
      image: nginx:1.20-alpine
```

```yaml
# service.yaml - specification of the www service
apiVersion: v1
kind: Service
metadata:
  name: www
spec:
  selector:
    app: www
  type: ClusterIP
  ports:
    - port: 80 # service exposes port 80 in the cluster
      targetPort: 80 # requests forwarded to port 80 of pods
```

- creation of the pod: `kubectl apply -f pod.yaml`
- creation of the service: `kubectl apply -f service.yaml`
- see pods and service: `kubectl get po,svc`
- launch interactive shell and curl to port 80: `kubectl run debug -it --image alpine` and in shell: `wget -O- http://www`

If we want to TEMPORARILY expose a port to host machine (outside the cluster):

- `kubectl port-forward svc/www 8080:80` then go to localhost: 8080
- OR `kubectl proxy`: then go to localhost:8001/api/v1/namespaces/default/services/www:80/proxy

## NodePort

Contrary to ClusterIP, exposes ports outside of cluster. Note that it also exposes inside the cluster
In fact, it can do what a ClusterIP can but can also open a port on each machine of the cluster so
that outside world can access the service
Note: port must be in a specified range (32000 - 32767 by default)

Example:

```yaml
# service.yaml - exactly same as above except type and ports
..
spec:
  ..
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      nodePort: 31000 # service is accessible at port 31000 for each node of the cluster
```

Now we can create the service (same as before)
To retrieve the external IP of one of the nodes, `kubectl get no -o wide`
We can then go to our browser to `<IP>:31000`

NOTE: with multipass, if external IP is not visible, IP can be retrieved with a `multipass list`

## LoadBalancer

Same as NodePort but allows to create a LoadBalancer which will be the entrypoint to
access the infrastructure. Instead of accessing directly the port of a given machine,
the traffic accesses the load balancer which is outside of the custer and will redirect
the request to one of the machines of the cluster.

```yaml
# service.yaml - exactly same as above except type and ports
..
spec:
  ..
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 80
      # nodePort: 31000 # if not specified, will be set by k8s
```

Usually we use LoadBalancer with a cloud provider.
If I create a load balancer service, it will expose an external IP. Then I can go to port
80 of this IP and it will redirect to one of the machine of the cluster in the port
specified by nodePort

## Useful cmds

- launch from a speficiation file: `kubectl apply -f <service-spec.yaml>`

- create pod and expose it:

```bash
# create pod
kubectl run whoami --image containous/whoami
# Expose NodePort Service
kubectl expose pod whoami --type=NodePort --port=8080 --target-port=80
# service will have the same selector as pod
```

- create service directly: `kubectl create service nodeport whoami --tcp 8080:80` (selector will be app:whoami here)

- create pod and service which exposes it at once:
  `kubectl run db --image mongo:4.2 --port=27017 --expose`

- same as pods, use `--dry-run=client -o yaml` to generate spec

# Deployment

## Creation

A deployment is a more high level resource that deals with an ensemble of identical pods
and makes sure that there is a specific number of nodes that run in the cluster
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

# Namespaces

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

# ConfigMap

- Allows to decouple configuration from application specification

- create from literal: `kubectl create cm my-config --from-literal=mykey=myval --from-literal=mykey2=myval2`
  Or from env file: `kubectl create cm my-config --from-env-file=myenvfile`

```
# myenvfile
mykey=myval
mykey2=myval
```

The output is the same:

```yaml
# output config map
apiVersion: v1
data:
  mykey: myval
  mykey2: myval2
kind: ConfigMap
metadata:
  name: my-config
```

- If we use `--from-file` instead of `from-env-file`, we get

```
..
data:
  <file-name>: |
    # full content of the file
```

In conclusion, while --from-literal and --from-env-file allow to store data as key value
pairs, --from-file uses the name of the file as the key and the full content as the value

To use the config in a pod, there are several possibilities:

- mounted as volume. Each of the keys of the cm will be create as a file in the specified
  place and the content will be the value associated to it. The config is mounted as read
  only so it can not be modified from within the pod

```yaml
# pod spec
spec:
  containers:
    - ..
      volumeMounts:
        - name: config
          mountPath: "/path/to/config"
  volumes:
    - name: config
      configMap:
        name: <config-map-name>
```

- used as env variable inside the container

```yaml
# pod spec
..
spec:
  containers:
    - name: ..
      ..
      env:
        - name: MYVAR
          valueFrom:
            configMapKeyRef:
              name: app-config-list # name of configmap
              key: log_level # gets env variable from configmap key
```

or if you want to load the full config

```yaml
# in the container spec
envFrom:
  - configMapRef:
    name: <configmap-name>
```

NOTE: if a deployment uses a configmap and the file configmap spec is changed, it will NOT
automatically update the deployment when running `kubectl apply -f deploy.yaml`.
https://stackoverflow.com/questions/37317003/restart-pods-when-configmap-updates-in-kubernetes
A possibility is to add a config hash in the deployment spec:

```yaml
..
spec:
  ..
  template:
    ..
    spec:
      containers:
      - name: nginx
        image: nginx:1.14-alpine
        volumeMounts:
        - name: config
          mountPath: "/etc/nginx/"
        env:
        - name: CONFIG_HASH
          value: ${CONFIG_HASH}
```

- now when configmap is modified:
  - `export CONFIG_HASH=$(kubectl get cm -oyaml | sha256sum | cut -d' ' -f1)`
  - `envsubst '${CONFIG_HASH}' < deploy.yaml`
  - then when doing `kubectly apply -f deploy.yaml`, the change will be reflected

# Secrets

- almost the same as config map but used for sensitive information

## generic

- `kubectl create secret generic my-secret` followed by `--from-file=myfile`, `--from-env-file=myfile`, `--from-literal=key=val`
  -> output will look the same as in configmap but values are stored in base64

Secret can then be used in pods similarly to config map (the key name change, for ex `secretKeyRef` instead of `configMapKeyRef` and `secretRef` instead of configMapRef)

## docker-registry

used to identify on a docker registry:
`kubectl create secret docker-registry registry-creds --docker-server=REGISTRY --docker-username=USERNAME --docker-password=PASSWORD --docker-email=EMAIL`

Then to pull a private image, in the pod spec:

```yaml
..
spec:
  containers:
    - name: ..
      image: private-registry.io/apps/internal-image # full path
  imagePullSecrets:
    - name: registry-creds  # secret created above
```

- when doing `kubectl get secret registry-creds -o yaml`, in the data key, we see `.dockerconfigjson` with creds encoded in b64

## tls

- use openssl to create a couple public private key: `openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out cert.pem`
- create secret: `kubectl create secret tls domain-pki --cert cert.pem --key key.pem`
- When we do `kubectel get secret domain-pki -o yaml`, we can see tls.crt and tls.key are here (encoded in b64)

- use the secret in a pod, in the spec of the pod:

```yaml
# use a secret as volume
..
spec:
  containers:
  - name: proxy
    image: nginx:1.12.2
    volumeMounts:
      - name: tls
        mountPath: "/etc/ssl/certs"
  volumes:
    - name: tls
      secret:
        secretName: domain-pki
```

```yaml
# use a secret by getting the key
..
spec:
  env:
    - name: MONGODB_URL
      valueFrom:
        secretKeyRef:
          name: mongo
          key: mongo_url
```

# initContainers

An initContainer is configured in a pod like all other containers, except that it is specified inside a initContainers section:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: myapp-container
      image: busybox:1.28
      command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
    - name: init-myservice
      image: busybox
      command:
        [
          'sh',
          '-c',
          'git clone <some-repository-that-will-be-used-by-application> ; done;',
        ]
```

When a POD is first created the initContainer is run, and the process in the initContainer
must run to a completion before the real container hosting the application starts.

# Security

## TLS in k8s

In k8s, all connection is TLS encrypted. Which means, each client needs a pair of certificates / key and
each server as well. They also need ca (root) certificates
So for ex, kube-scheduler is a client of kube-apiserver. kube-apiserver is a client for etcdserver, ...

## Authentication

### Basic User auth

- we can use a basic file containing users and passwords. To do so, we must pass the
  `--basic-auth-file=<file.csv>` to the kube-apiserver. For ex in kubeadm, we can modify the
  static pod definition. <file.csv> should have 3 columns: password, user, userid.
  Then to connect to api server: `curl -v -k https://localhost:6443/api/v1/pods -u "user1:password123`

- we can use a static token file: `--token-auth-file=<file.csv>`: <file.csv> has 4
  columns: token,user,userid,group. Then to connect to the api server: `curl -v -k https://master-node-ip:6443/api/v1/pods --header "Authorization: Bearer <token>`

Note that we must configure the roles and rolebindings. The rolebinding must contain:

```yaml
subjects:
  - kind: User
    name: user1
    apiGroup: rbac.auhorization.k8s.io
```

### Certificate auth

Preferred way to authenticate a user when not using third party solutions `--client-ca-file=<FILE>`

To add a new admin to the cluster, we would need to do the following:

- new admin generates a key and a certificate signing request
- new admin sends the csr to main admin
- main admin passes csr to the node containing root certificate (usually controlplane)
- csr is signed with the associated private key on this node
- main admin retrieves crt
- main admin passes crt to new admin.
- new admin now has access to cluster

Process must be repeated when the crt expires.
This is tedious work. Fortunately, k8s has a certificate api. So if a new-admin wants to
gain access to the cluster:

- new admin creates a key and csr: `openssl genrsa -out newadmin.key 2048` and a csr: `openssl req -new -key newadmin.key -subj "/CN=newadmin" -out newadmin.csr`
- new admin moves the output in base64 to a CertificateSigningRequest yaml spec file.
- once the object is created, main admin can do `kubectl get csr` to see the csr and `kubectl certificate approve newadmin` to approve it
- view the certificate with `kubectl get csr newadmin -o yaml` and extract the certificate

All the certificate operations (CSR-APPROVING, CSR-SIGNING) is done by the controller manager, which is why we specify the `--cluster-signing-cert-file` and `--cluster-signing-key-file` in its config

example yaml file:

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: akshay
spec:
  usages:
    - key encipherment
    - digital signature
    - client auth
  signerName: kubernetes.io/kube-apiserver-client
  request: LS...
# request is obtained by piping csr file to base64 -w 0
```

### Kubeconfig

Now that the cert is configured, to have access to the cluster:
`curl https://<master-node-ip>:<port>/api/v1/pods --key admin.key --cert admin.cert --cacert ca.crt`

- When using the kubectl utility, these args are passed automatically by using the kubeconfig file
  By default kubeconfig file is in $HOME/.kube/config but it can be modified by passing `--kubeconfig <file>` to kubectl cmd or by
  changing the KUBECONFIG var

Kubeconfig contains 3 parts:

- clusters = list of clusters
- users = list of users (with the way to get the creds)
- context which contains a list of user/cluster pair

We can switch context with `kubectl use-context <context-name>`

### ServiceAccount

To authenticate a process inside the cluster, we use a ServiceAccount:

- gives rights to containers in a pod
- possibility to generate a JWT token (see section on Web interface)
- by default, pods use the ServiceAccount called default

To add a service account to a pod, add `serviceAccountName` in the spec. A token will be mounted
in the pod at /var/run/secrets/kubernetes.io/serviceaccount.
To create additionnal token, use `kubectl create token <sa-name>`
It is also possible to create a secret that references the service account name (see Web Interface section)

## Authorization

- Being authenticated is not enough to be granted permissions. You need authorization.
  There are several builtin ways in kubernetes:
- NODE: access within the cluster for users part of the system:node group
- ABAC (attribute based access control): you must define a policy that link a user to a list of actions. You must restart kube-api server each time you add a user. As such, this is not used much
- RBAC (role based access control):
  - Each ServiceAccount/User must then be linked to a Role via a RoleBinding or a ClusterRole via a ClusterRoleBinding
  - RoleBindings are limited to one namespace, ClusterRoleBindings apply to the whole custer
  - Roles and ClusterRoles grant access to resources with high granularity.
    - For ex a ServiceAccount can be authorized to list pods but not delete them
  - Note :To see which clusterroles a user / service account has access to, you can list all the
    clusterrolebindings and search for the name associated to the user / service account: `kubectl get customrolebindings -o yaml | less -p 'name: <name>'`
    I did not find a better way
- Webhook: Forward authorization process to external application
- AlwaysAllow: all requests are authorized
- AlwaysDeny: no requests are authorized

Authorized mechanisms are specified when configuring kube-apiserver : `--authorization-mode=Node,RBAC,Webhook`
Note that if a mode fails, it is forwarded to the next so the order is important

To check if an action is possible: `kubectl auth can-i <verb> <resource>`
If you are an admin, check if someone else can do an action: `kubectl auth can-i <verb> <resource> --as <sa|user>`

### Example RBAC with ServiceAccount

- Create a token associated to sa default: `kubectl create token default` (visualise it at https://jwt.io,)
- To retrieve the token, we must access it from a pod that uses this service account (see below)
- Run a debug container: `kubectl run debug -it --image alpine`
- Inside the container:

  - install curl `apk add --update curl`
  - try to access kubernetes server API: `curl https://kubernetes/api/v1 --insecure` => we get an error
  - retrieve TOKEN here: `TOKEN=$(cat /run/secrets/kubernetes.io/serviceaccount/token)`
  - try again: `curl -H "Authorization: Bearer $TOKEN" https://kubernetes/api/v1/ --insecure` => it works
  - try to access the list of pods: `curl -H "Authorization: Bearer $TOKEN" https://kubernetes/api/v1/namespaces/default/pods/ --insecure` => forbidden, we don't have enough permissions

- Create a new service account, a new role and a new role binding

```yaml
# demo-sa.yaml

apiVersion: v1
kind: ServiceAccount
metadata:
  name: demo-sa

---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: list-pods
  namespace: default
rules:
  - apiGroups:
      - ''
    resources:
      - pods
    verbs:
      - list

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: list-pods_demo-sa
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: list-pods
subjects:
  - kind: ServiceAccount
    name: demo-sa
    namespace: default
```

- run a basic alpine pod (specify serviceAccountName: demo-sa in spec) and do the
  same steps as above. Listing pods is now possible

## Security context

When running a pod, it is possible to specify the security context:

```yaml
..
## example at the pod level
spec:
  ..
  securityContext:
    runAsUser: 1000

--
## example at the container level
spec:
  ..
  containers:
    - name:
      securityContext:
        runAsUser: 1000
        capabilities:
          add: ["MAC_ADMIN"] # adds a privilege, only supported at container level
```

## Network policy

By default, all pods can communicate with each other.
We can restrict the communication by creating a network policy
We use labels and selector to link a policy to a pod.

```yaml
# example policy
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db # select pods with the role:db label
    policyTypes:
      - Ingress #only ingress traffic is restricted.
    ingress:
      - from:
          - podSelector:
              matchLabels:
                name: api-pod
        ports:
          - protocol: TCP
            port: 3306
```

Note: Kube-router, Calico, Romana, Weave-net support network policy but Flannel does not

# Web interface

If on minikube:

- enable dashboard: `minikube addons enable dashboard`
- launch dashboard: `minikube dashboard`

If not on minikube:

- `kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/recommended.yaml`
- `kubectl proxy` then go to `http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/`
  Then we need to login, either by uploading the kubeconfig, copy the token from kubeconfig or create an admin token with authentication rights
  To do so, we create a Service account and a CLusterRoleBinding to give it admin rights. We also create a secret

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: admin-user
    namespace: kube-system
---
apiVersion: v1
kind: Secret
metadata:
  name: admin-user
  namespace: kube-system
  annotations:
    kubernetes.io/service-account.name: admin-user
type: kubernetes.io/service-account-token
```

and we get the token with: `echo $(kubectl -n kube-system get secret admin-user -o jsonpath='{.data.token}' | base64 --decode)`

- Note: instead of creating the secret (which creates a non expiring token), we can also do: `kubectl -n kube-system create token admin-user`
  and retrieve the token. If we use this method without binding the token to a secret, see prev section to retrieve the token

# DaemonSet

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

## Example: use of weavescope

Weavescope is an app used to see the processes, containers and all running in the cluster

- Retrieve the spec with: `curl -sSL -o scope.yaml "https://cloud.weave.works/k8s/scope.yaml?k8s-version=$(kubectl version | base64 | tr -d '\n')"`
- Create the resources: `kubectl apply -f scope.yaml`
- Resources are created in the namespace weave: we can list them with: `kubectl get all -n weave`
- We can see it contains a Deployment called weave-scope-app (target port 4040) and a DaemonSet which creates weave-scope-agent pods
- port forward the corresponding pod: `kubectl port forward -n weave pod/weave-scope-app-6996cfd49b-48zz4 4040`
- app is available at `localhost:4040`

# Jobs and Cronjobs

- jobs allow to execute a task on the cluster and crojobs allow to schedule them.
- to write a cronjob, you basically write a schedule and a jobtemplate

- example job:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: dump
spec:
  template:
    spec:
      restartPolicy: Never
      nodeSelector:
        app: dump # pod will be scheduled on node with given label
      containers:
        - name: mongo
          image: mongo:4.0
          command:
            - /bin/bash
            - -c
            - mongodump --gzip --host db --archive=/dump/db.gz
          volumeMounts:
            - name: dump
              mountPath: /dump
      volumes:
        - name: dump
          hostPath:
            path: /dump

---
apiVersion: batch/v1
kind: CronJob
metadata:
  name: dump
spec:
  schedule: '* * * * *' #
  jobTemplate:
    spec:
      template:
        spec:
          # everything in job spec.template.spec
```

# Networking

## Node addresses

- Get ip addresses of nodes with `kubectl get no -o wide`
- Inside a node (after ssh),
  - List all ip addresses with `ip addr` => identify which network the node is part of
  - Run `ip link show <network-name>` to have infos about Mac addresses
  - See docker networks: `docker network ls`. Inspect a network `docker inspect <id>` to get the ip
  - see the routes and default gateway: `ip route`
  - see the ports currently listened: `netstat -l` (add -n to have numeric addresses and -p to have process names)

Note: usually, docker network ls shows the bridge network. It corresponds to docker0 on the host

## Pod networking

K8s does not implement the networking solution. Instead, it expects us to use a plugin that
respects the CNI. Rules are the following:

- Every Pod should have an IP address
- Every Pod should be able to communicate with every other Pod in the same node using that IP address
- Every Pod should be able to communicate with every other Pod on other nodes without NAT

There are many networking solutions such as flannel, cilium, vmware NSX, ...

Prior to 1.24, the CNI plugin was configured on the kubelet service on each node of the cluster:

```
# PRIOR TO 1.24 ONLY, removed in current version
--network-plugin=cni
--cni-bin-dir=/opt/cni/bin # all supported cni plugins as executables
--cni-conf-dir=/ect/cni/net.d # where kubelet finds which plugins should be used
```

Check current kubelet config with `ps -aux | grep kubelet`

After 1.24 I don't know how to easily get the plugin used. Most of the time it is in
/etc/cni/net.d/

Note: if no network plugin is install, pods will be stuck in containerCreating as no
ip can be given to the pod. If the cluster is running, you can for example install
weave-net with `kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"`

## Service networking

When a service is created, it is assigned an ip address from a predefined range.
It can be seen in the kube-apiserver config `--service-cluster-ip-range ipNet` (default 10.0.0.0/24)
kube-proxy is then in charge of creating forwarding rules from the service to the underlying pod
Now when a pod tries to reach a service, it is forwarded to the underlying node that is accessible from anywhere in the cluster

kube-proxy can be configured with `--proxy-mode [iptables | userspace | ipvs]` (iptables by default)

Note: from within a pod, use `nslookup <service-name-or-ip>` to get extra info about ip

# Ingress

- Set of rules to connect to cluster services from the internet
- If using minikube: `minikube addons enable ingress`

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: www
spec:
  rules:
    - host: www.example.com
      http:
        paths:
          - backend:
              service:
                name: www
                port:
                  number: 80
            path: /www
            pathType: Exact
```

By themselves, ingress do not work on their own. They must be associated with an ingress
controller (such as nginx-ingres-controller) that will update its config based on the
ingress

Then when i go to example.com I am redirected to www service (provided dns resolution
points to correct ip). If testing in local machine, you must modify /etc/hosts
(C:\Windows\System32\drivers\etc\hosts on windows)

# Stateful Workload

- Volume = share data between containers in a pod
- PersistentVolume = storage provisionned by an admin
- PersistentVolumeClaim = ask for a storage, consumes a PV
- StorageClass: Dynamic provision of storage
- StatefulSet:

## Volume

- Separates data from container lifecycle

- lots of types: `kubectl explain pods.spec.volumes`

```yaml
..
# emptyDir: temporary directory that shares a pod's lifetime.
spec:
  containers:
    - ..
      volumeMounts:
        - mountPath: /data/db
          name: data
  volumes:
    - name: data
      emptyDir: {}
```

```yaml
# hostPath: directly mounted on host machine
spec:
  ..
  volumes:
    - name: data
      hostPath:
        path: /data-db
```

## PersistentVolume and PersistentVolumeClaim

- PersistentVolume is provisionned storage (statically or dynamically)
- PersistentVolumeClaim asks for storage with constraints and consumes an existing or a
  StorageClass. It is used by a pod

For ex, it is possible to create a PV and a PVC manually:

```yaml
#pvc
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ssd
spec:
  storageClassName: manual # same storageClassName as PV
  resources:
    requests:
      storage: 500Mi
  accessModes:
    - ReadWriteMany
```

```yaml
#pv
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv
spec:
  storageClassName: manual # same storageClassName as PVC
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: '/var/lib/mysql'
```

```
#pod
..
spec:
  containers:
    - ..
    volumeMounts:
      - name: data-db
        mountPath: /var/lib/mysql
  volumes:
    - name: data-db
      PersistentVolumeClaim:
        claimName: ssd
```

- When doing a kubectl get pvc,pv, we will see that the PVC and PV are bound

A better approach is to use a storage class for dynamic providing.
For ex:

```
# storage class
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: manual
provisioner: kubernetes.io/no-provisioner # lots of available provisioners
volumeBindingMode: WaitForFirstConsumer
```

Here by creating this storage class, the binding between pv and pvc will wait for
the pod scheduling.

Note: Usually, we use a PVC and a storageClass but we don't create the PV ourselves.
Instead, we specify a provisioner in the storageClass (azure, aws, etc)

## StatefulSet

- used for stateful applications
- manages pods in a NON interchangeable way
- each pod has a constant name, a persistent network id, its own storage
- Example with a mysql cluster:
  - one master pod (for writing)
  - slaves pods (for reading)

# Cluster maintenance

## Node drain

- Makes a node unschedulable and evicts the pods `kubectl drain <node-name>`
- If there are pods that are part of a daemonset on the node, pass `--ignore-daemonsets`
- If there are pods that are not part of a replicaset, you must pass `--force`. However, the pods in question will be forever lost

- To make a node schedulable again: `kubectl uncordon <node-name>`

Note: To make a pod unschedulable without evicting the existing pods: `kubectl cordon <node-name>`

## Note on upgrades

Compatibility of k8s elements:

- if kube-apiserver is v1.X:
  - controller manager and kube-scheduler can be 1.X-1
  - kubelet and kube-proxy can be 1.X-2
  - kubectl can be between 1.X-1 and 1.X+1

To upgrade a cluster with kubeadm: https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/

Note: do not forget to upgrade kubelet as well with : `apt install kubelet=1.X.x-00`
and then restart services: `systemctl daemon-reload` and `systemctl restart kubelet`

Note: when doing `kubectl get no -o wide`, the version we see is the version of kubelet

## Backup and restore

Best strat is to use the declarative approach and save yaml definition files to source control
However, some resources can be declared using the imperative approach so we could use : `kubectl get all -A -o yaml > all-deploy-services.yaml`

Another approach is to backup the etcd server itself.
When configuring etcd, the --data-dir option specifies where all the data is stored. That is the directory we want to backup.
We can also use the etcdctl tool:

```bash
# first, we specify the api version:
export ETCDCTL_API=3
# Then we need to get the credentials of etcd. If it runs in a pod, we can look for
# the configuration. We must have the files on the machine where etcdctl runs
# then we save the snapshot
etcdctl snapshot save --endpoints=<controlpane-ip> --cert=<server.crt> --cacert=<ca.crt> --key=<server.key> /path/to/snapshot.db
# then we create the restore
etcdctl snapshot restore /path/to/snapshot.db --data-dir=/path/to/new-data/
# If process was not done on controlplane node, we transfer the folder
scp /path/to/new-data controlplane-node:/var/lib/data-new
# Then, in the master node, we update the pod spec to point to the new data-dir
```

# Helm

Install helm: https://github.com/helm/helm/releases
then add stable repo: `helm repo add stable https://charts.helm.sh/stable`

- main hub: https://artifacthub.io. When clicking on a given package, i have the instructions
  to install it with helm, for ex
  `helm repo add nginx-stable https://helm.nginx.com/stable` and `helm install nginx-ingress nginx-stable/nginx-ingress`

NOTE: to install on azure, they suggest:
`helm install ingress-nginx ingress-nginx/ingress-nginx --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-health-probe-request-path"=/healthz`
which adds some annotations. It seems it does not work without it for some reason

- list all installed helm packages: `helm list`
- get spec: `helm get manifest nginx-ingress`
- delete : `helm delete nginx-ingress`

Note: you can use kubectl to see the installed resources

- Helm allows for templating:
  - create new project: `helm create <project>`
  - in values.yaml, put your values
  - in the templates folder, you can use templating: `{{ .Values.<key>.<field> }}`
  - install app: `helm install tick <path/to/helm/dir>`

## Example nginx ingress controller

- `helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx`
- `helm install ingress-nginx ingress-nginx/ingress-nginx --create-namespace -namespace ingress-nginx`  
  Note: on azure, you must set the controller annotations: `--set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-health-probe-request-path"=/healthz`

If we look at the created services, there is ingress-controller which exposes a load
balancer and ingress-controller-admission (ClusterIP)

ingress-controller-admission is there to validate the ingress rules.

In practice, several resources are created:

- A ServiceAccount (ingress-nginx-admission)
  with associated role / rolebindings (to allow to get and create secrets in the namespace) and
  associated clusterroles / clusterrolebindings (to allow to get and update validatingwebhookconfigurations in admissionregistration.k8s.io apiGroups)
- A ValidatingWebhookConfiguration
- Jobs to create secrets and patch the ValidatingWebhookConfiguration
- admission service

- To validate the ingress yaml file:
  - When you deploy an ingress YAML, the Validation admission intercepts the request.
  - Kubernetes API then sends the ingress object to the validation admission controller service endpoint based on admission webhook endpoints.
  - Service sends the request to the Nginx deployment on port 8443 for validating the ingress object.
  - The admission controller then sends a response to the k8s API.
  - If it is a valid response, the API will create the ingress object.

Then the ingress-controller is in charge of the routing

After that, you can create your service, deployment and ingress rules as usual.
All inbound traffic will be handled by ingress-controller. (You can see that it exposes a loadbalancer)

# Azure specifics

For all the examples below, we set env variables:

```bash
export RG=myResourceGroup # the resource group, needed for everything
export AKS_CLUSTER=myManagedCluster # the managed cluster where k8s is deployed
export AKS_ADMIN_GROUP=myAKSAdminGroup # the group that has the admin rights on the cluster
export LOCATION=francecentral # location of the cluster (see azure locations)

export AKS_DEV_GROUP=appdev # the group used to demonstrate Role Based Access Control
export CONTAINER_REGISTRY=myContainerRegistry # the container registry used to pull / push images
export STORAGE_ACCOUNT_NAME=mystorageaccount$RANDOM # name must be unique and lowercase
export SHARE_NAME=aksshare # name of service to share the account, must be lowercase

```

## Azure cli

- Install on ubuntu:

  - get current version: `AZ_REPO=$(lsb_release -cs)`
  - add repo to sources lits: `echo "deb [arch=amd64] https://packages.microsoft.com/repos/azure-cli/ $AZ_REPO main" | sudo tee /etc/apt/sources.list.d/azure-cli.list`
  - install: `sudo apt update` and `sudo apt install azure-cli`

- Install azure AKS cli and kubelogin: `sudo az aks install-cli`

## Cluster Creation steps

### Create a resource group

You can create from cli :`az group create --name $RG --location $LOCATION`
Note: On Web interface, resource groups are visible in Home/Resource groups

### Create groups

Create the admin group :`az ad group create --display-name $AKS_ADMIN_GROUP --mail-nickname $AKS_ADMIN_GROUP`
The admin group will be in charge of the cluster

Optionally, create the dev group: `az ad group create --display-name $AKS_DEV_GROUP --mail-nickname $AKS_DEV_GROUP`
The dev group will have limited access to the cluster (used to demonstrate RBAC)

On the web, groups can be found in Home/Groups

### Create the cluster

Finally, you can create the cluster.
Because dealing with user authentication can be difficult, it is possible to use azure active directory which simplifies the process
(this is the enable-aad in the command)

First, you need to get the admin group id: `ADMIN_ID=$(az ad group show --group myAKSAdminGroup --query id -o tsv)`
Then create the cluster: `az aks create -g $RG -n $AKS_CLUSTER --enable-aad --aad-admin-group-object-ids $ADMIN_ID [--aad-tenant-id <organisation-id>]`

Alternatively, to create the cluster, go to the web interface in Kubernetes Services.
Then click Create a Kubernetes cluster. Be sure to select Azure AD authentication with Kubernetes RBAC in Access and select the $AKS_ADMIN_GROUP as admin cluster

### Connect as admin to the cluster

As a user that is part of the $AKS_ADMIN_GROUP:

- login to azure: `az login`
- get creds: `az aks get-credentials --resource-group $RG --name $AKS_CLUSTER`
  (this will modify the file pointed by $KUBECONFIG (or ~.kube/config by default))
- then try a `kubectl get po`: it should work, you should have access to everything

## Azure AKS and RBAC

See following resources:
https://docs.microsoft.com/fr-fr/azure/aks/azure-ad-rbac
https://docs.microsoft.com/fr-fr/azure/aks/managed-aad

To demonstrate role based access control on azure,
first, retrieve the id of the AKS_DEV_GROUP with : `az ad group show --group $AKS_DEV_GROUP --query id -o tsv`

Then, Still as ad admin, you can create a role and a role binding as usual: `kubectl apply -f <spec-file>`

```yaml
# spec-file

kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: dev-user-full-access
  namespace: dev
rules:
  - apiGroups: ['', 'extensions', 'apps']
    resources: ['*']
    verbs: ['*']
  - apiGroups: ['batch']
    resources:
      - jobs
      - cronjobs
    verbs: ['*']
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: dev-user-access
  namespace: dev
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: dev-user-full-access
subjects:
  - kind: Group
    namespace: dev
    name: <AKS_DEV_ID> # replace with id of AKS_DEV_GROUP
```

For the users of the AKS_DEV_GROUP to be able to access the cluster, you must specify their rights in the
cluster (replace <AKS_DEV_ID> with the id you got earlier):
`az role assignment create --assignee <AKS_DEV_ID> --role "Azure Kubernetes Service Cluster User Role" --scope $AKS_CLUSTER`
Alternatively, access the IAM (access control) of the cluster and add the role Azure Kubernetes Service Cluster User Role for AKS_DEV_GROUP

Now a user from the group which logs into the cluster with `az login`
and sets the creds with `az aks get-credentials --resource-group $RG --name $AKS_CLUSTER`
will be able to access the parts defined in the role (in this example do what they want on namespace dev)
Other operations will result in forbidden error

## Add an azure container registry

https://docs.microsoft.com/en-us/azure/aks/cluster-container-registry-integration?tabs=azure-cli

Create with `az acr create -n $CONTAINER_REGISTRY -g $RG --sku basic`
NOTE: on the portal, it is in Home/Container registries

Attach it to the cluster: `az aks update -n $AKS_CLUSTER -g $RG --attach-acr $CONTAINER_REGISTRY`
Now if you go to IAM (Access control) of the registry, you should see the role AcrPull given to the agent pool of the cluster.

- Check that cluster can pull the images with: `az aks check-acr -g $RG --name $AKS_CLUSTER --acr ${CONTAINER_REGISTRY}.azurecr.io`

- You can also connect to the registry: `az acr login -n $CONTAINER_REGISTRY` and then use
  docker pull and docker push (image name must start with `${CONTAINER_REGISTRY}.azurecr.io/`)

## Add a persistent volume to share data between pods

https://docs.microsoft.com/en-gb/azure/aks/azure-files-volume

First create a storage account: `az storage account create -n $STORAGE_ACCOUNT_NAME -g $RG -l $LOCATION --sku Standard_LRS`
Storage can be seen on the portal in Home/Storage Accounts

Get the connection string of the storage account: `export AZURE_STORAGE_CONNECTION_STRING=$(az storage account show-connection-string -n $STORAGE_ACCOUNT_NAME -g $RG -o tsv)`

Create the file share: `az storage share create -n $SHARE_NAME --connection-string $AZURE_STORAGE_CONNECTION_STRING`
On the portal, the file share can be seen inside the Data Storage menu in the associated storage account

Get the storage key : `STORAGE_KEY=$(az storage account keys list --resource-group $RG --account-name $STORAGE_ACCOUNT_NAME --query "[0].value" -o tsv)`
Create a kubernetes secret: `kubectl create secret generic azure-secret --from-literal=azurestorageaccountname=$STORAGE_ACCOUNT_NAME --from-literal=azurestorageaccountkey=$STORAGE_KEY`

Finally, create a volume or persistent volume. Below is example spec:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: azurefile
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: azurefile-csi
  csi:
    driver: file.csi.azure.com
    readOnly: false
    volumeHandle: unique-volumeid # make sure this volumeid is unique in the cluster
    volumeAttributes:
      resourceGroup: EXISTING_RESOURCE_GROUP_NAME # optional, only set this when storage account is not in the same resource group as agent node
      shareName: ${SHARE_NAME} # replace with value of share name or use envsubst
    nodeStageSecretRef:
      name: azure-secret # same name as the secret that was just created
      namespace: default
  mountOptions:
    - dir_mode=0777
    - file_mode=0777
    - uid=0
    - gid=0
    - mfsymlinks
    - cache=strict
    - nosharesock
    - nobrl
```

The volume is now ready to be used with any PersistentVolumeClaim / StorageClass, for ex

```
yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc
spec:
  storageClassName: azurefile-csi # same storage class as pv
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 100Mi
```

- To be sure everything worked, checked that pv and pvc are bound when doing `kubectl get pv,pvc`

# Summary of useful concepts

- Container:

  - is a process
  - visible from host machine
  - with limited vision of the system
  - with limited accessible resources
  - combines two linux primitives: namespace and control groups (cgroups)

- Docker:

  - simplifies use of containers
  - brings concept of images = packaging of an app and its dependencies instantiated in a container that can be deployed in several environments
  - see http://hub.docker.com

- Microservices architecture:

  - application separated in several services
  - each service can be developed / updated independently
  - well defined interface is needed for communication between services
  - containers are particularly adapted for Microservices
  - complexity is moved to the orchestration of full application

- Cloud native app:

  - Microservices oriented application
  - packaged in containers
  - Dynamic orchestration
  - a lot of project supported by Cloud Native Computing Foundation: https://www.cncf.io (kubernetes for ex)

- Devops

  - goal: minimise time to deliver a feature
  - Frequent deployments
  - With tested code
  - With automated processes
  - Short improvement loop
  - Example:
    - infrastructure (aws, digitalOcean, azure)
    - provisionning (terraform, CloudFormation)
    - build (github, gitlab, docker)
    - test (selenium, jenkins, circleci)
    - deploy (Ansible, CHEF, registry docker)
    - run (kubernetes, Swarm)
    - mesure (prometheus, elastic)

- Kubernetes role:
  - deal with applications running in containers (deployment, scaling, self-healing)
  - ensures that application is running as intended (reconciliation loop)
  - can deal with stateless and stateful apps
  - can handle secrets and configs
  - can run long-running processes or batch jobs
  - RBAC (role based access control)
