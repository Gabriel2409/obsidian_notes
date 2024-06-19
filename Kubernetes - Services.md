#k8s

expose pods via network rules
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
the traffic accesses the load balancer which is outside of the cluster and will redirect
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