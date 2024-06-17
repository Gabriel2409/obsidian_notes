#k8s

An initContainer is configured in a pod like all other containers, except that it is specified inside a initContainers section:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: myapp-container
      image: busybox:1.28
      command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
    - name: init-myservice
      image: busybox
      command:
        [
          'sh',
          '-c',
          'git clone <some-repository-that-will-be-used-by-application> ; done;',
        ]
```

When a POD is first created the initContainer is run, and the process in the initContainer
must run to a completion before the real container hosting the application starts.