#k8s

- Set of rules to connect to cluster services from the internet
- If using minikube: `minikube addons enable ingress`

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: www
spec:
  rules:
    - host: www.example.com
      http:
        paths:
          - backend:
              service:
                name: www
                port:
                  number: 80
            path: /www
            pathType: Exact
```

By themselves, ingress do not work on their own. They must be associated with an ingress
controller (such as nginx-ingres-controller) that will update its config based on the
ingress

Then when i go to example.com I am redirected to www service (provided dns resolution
points to correct ip). If testing in local machine, you must modify /etc/hosts
(C:\Windows\System32\drivers\etc\hosts on windows)