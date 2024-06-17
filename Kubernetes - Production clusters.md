#k8s 

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