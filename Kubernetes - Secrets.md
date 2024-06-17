#k8s

- almost the same as config map but used for sensitive information

## generic

- `kubectl create secret generic my-secret` followed by `--from-file=myfile`, `--from-env-file=myfile`, `--from-literal=key=val`
  -> output will look the same as in configmap but values are stored in base64

Secret can then be used in pods similarly to config map (the key name change, for ex `secretKeyRef` instead of `configMapKeyRef` and `secretRef` instead of configMapRef)

## docker-registry

used to identify on a docker registry:
`kubectl create secret docker-registry registry-creds --docker-server=REGISTRY --docker-username=USERNAME --docker-password=PASSWORD --docker-email=EMAIL`

Then to pull a private image, in the pod spec:

```yaml
..
spec:
  containers:
    - name: ..
      image: private-registry.io/apps/internal-image # full path
  imagePullSecrets:
    - name: registry-creds  # secret created above
```

- when doing `kubectl get secret registry-creds -o yaml`, in the data key, we see `.dockerconfigjson` with creds encoded in b64

## tls

- use openssl to create a couple public private key: `openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out cert.pem`
- create secret: `kubectl create secret tls domain-pki --cert cert.pem --key key.pem`
- When we do `kubectel get secret domain-pki -o yaml`, we can see tls.crt and tls.key are here (encoded in b64)

- use the secret in a pod, in the spec of the pod:

```yaml
# use a secret as volume
..
spec:
  containers:
  - name: proxy
    image: nginx:1.12.2
    volumeMounts:
      - name: tls
        mountPath: "/etc/ssl/certs"
  volumes:
    - name: tls
      secret:
        secretName: domain-pki
```

```yaml
# use a secret by getting the key
..
spec:
  env:
    - name: MONGODB_URL
      valueFrom:
        secretKeyRef:
          name: mongo
          key: mongo_url
```