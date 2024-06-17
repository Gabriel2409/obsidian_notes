#k8s

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


