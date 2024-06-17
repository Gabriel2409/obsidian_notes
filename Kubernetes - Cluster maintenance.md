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
