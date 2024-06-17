#k8s

- Volume = share data between containers in a pod
- PersistentVolume = storage provisionned by an admin
- PersistentVolumeClaim = ask for a storage, consumes a PV
- StorageClass: Dynamic provision of storage
- StatefulSet:

## Volume

- Separates data from container lifecycle

- lots of types: `kubectl explain pods.spec.volumes`

```yaml
..
# emptyDir: temporary directory that shares a pod's lifetime.
spec:
  containers:
    - ..
      volumeMounts:
        - mountPath: /data/db
          name: data
  volumes:
    - name: data
      emptyDir: {}
```

```yaml
# hostPath: directly mounted on host machine
spec:
  ..
  volumes:
    - name: data
      hostPath:
        path: /data-db
```

## PersistentVolume and PersistentVolumeClaim

- PersistentVolume is provisionned storage (statically or dynamically)
- PersistentVolumeClaim asks for storage with constraints and consumes an existing or a
  StorageClass. It is used by a pod

For ex, it is possible to create a PV and a PVC manually:

```yaml
#pvc
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ssd
spec:
  storageClassName: manual # same storageClassName as PV
  resources:
    requests:
      storage: 500Mi
  accessModes:
    - ReadWriteMany
```

```yaml
#pv
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv
spec:
  storageClassName: manual # same storageClassName as PVC
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: '/var/lib/mysql'
```

```
#pod
..
spec:
  containers:
    - ..
    volumeMounts:
      - name: data-db
        mountPath: /var/lib/mysql
  volumes:
    - name: data-db
      PersistentVolumeClaim:
        claimName: ssd
```

- When doing a kubectl get pvc,pv, we will see that the PVC and PV are bound

A better approach is to use a storage class for dynamic providing.
For ex:

```
# storage class
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: manual
provisioner: kubernetes.io/no-provisioner # lots of available provisioners
volumeBindingMode: WaitForFirstConsumer
```

Here by creating this storage class, the binding between pv and pvc will wait for
the pod scheduling.

Note: Usually, we use a PVC and a storageClass but we don't create the PV ourselves.
Instead, we specify a provisioner in the storageClass (azure, aws, etc)

## StatefulSet

- used for stateful applications
- manages pods in a NON interchangeable way
- each pod has a constant name, a persistent network id, its own storage
- Example with a mysql cluster:
  - one master pod (for writing)
  - slaves pods (for reading)