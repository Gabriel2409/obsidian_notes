#k8s

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

