#k8s

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