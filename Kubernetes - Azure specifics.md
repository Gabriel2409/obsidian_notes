#k8s

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
