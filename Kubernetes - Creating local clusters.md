#k8s 

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