#k8s
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
