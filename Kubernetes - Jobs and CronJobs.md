#k8s

- jobs allow to execute a task on the cluster and crojobs allow to schedule them.
- to write a cronjob, you basically write a schedule and a jobtemplate

- example job:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: dump
spec:
  template:
    spec:
      restartPolicy: Never
      nodeSelector:
        app: dump # pod will be scheduled on node with given label
      containers:
        - name: mongo
          image: mongo:4.0
          command:
            - /bin/bash
            - -c
            - mongodump --gzip --host db --archive=/dump/db.gz
          volumeMounts:
            - name: dump
              mountPath: /dump
      volumes:
        - name: dump
          hostPath:
            path: /dump

---
apiVersion: batch/v1
kind: CronJob
metadata:
  name: dump
spec:
  schedule: '* * * * *' #
  jobTemplate:
    spec:
      template:
        spec:
          # everything in job spec.template.spec
```
