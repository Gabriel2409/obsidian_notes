#k8s

- Allows to decouple configuration from application specification

- create from literal: `kubectl create cm my-config --from-literal=mykey=myval --from-literal=mykey2=myval2`
  Or from env file: `kubectl create cm my-config --from-env-file=myenvfile`

```
# myenvfile
mykey=myval
mykey2=myval
```

The output is the same:

```yaml
# output config map
apiVersion: v1
data:
  mykey: myval
  mykey2: myval2
kind: ConfigMap
metadata:
  name: my-config
```

- If we use `--from-file` instead of `from-env-file`, we get

```
..
data:
  <file-name>: |
    # full content of the file
```

In conclusion, while --from-literal and --from-env-file allow to store data as key value
pairs, --from-file uses the name of the file as the key and the full content as the value

To use the config in a pod, there are several possibilities:

- mounted as volume. Each of the keys of the cm will be create as a file in the specified
  place and the content will be the value associated to it. The config is mounted as read
  only so it can not be modified from within the pod

```yaml
# pod spec
spec:
  containers:
    - ..
      volumeMounts:
        - name: config
          mountPath: "/path/to/config"
  volumes:
    - name: config
      configMap:
        name: <config-map-name>
```

- used as env variable inside the container

```yaml
# pod spec
..
spec:
  containers:
    - name: ..
      ..
      env:
        - name: MYVAR
          valueFrom:
            configMapKeyRef:
              name: app-config-list # name of configmap
              key: log_level # gets env variable from configmap key
```

or if you want to load the full config

```yaml
# in the container spec
envFrom:
  - configMapRef:
    name: <configmap-name>
```

NOTE: if a deployment uses a configmap and the file configmap spec is changed, it will NOT
automatically update the deployment when running `kubectl apply -f deploy.yaml`.
https://stackoverflow.com/questions/37317003/restart-pods-when-configmap-updates-in-kubernetes
A possibility is to add a config hash in the deployment spec:

```yaml
..
spec:
  ..
  template:
    ..
    spec:
      containers:
      - name: nginx
        image: nginx:1.14-alpine
        volumeMounts:
        - name: config
          mountPath: "/etc/nginx/"
        env:
        - name: CONFIG_HASH
          value: ${CONFIG_HASH}
```

- now when configmap is modified:
  - `export CONFIG_HASH=$(kubectl get cm -oyaml | sha256sum | cut -d' ' -f1)`
  - `envsubst '${CONFIG_HASH}' < deploy.yaml`
  - then when doing `kubectly apply -f deploy.yaml`, the change will be reflected