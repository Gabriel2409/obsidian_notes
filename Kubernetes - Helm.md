#k8s 


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