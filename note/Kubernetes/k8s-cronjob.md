

* [一、定时任务](#%E4%B8%80%E5%AE%9A%E6%97%B6%E4%BB%BB%E5%8A%A1)

---
# 一、定时任务
```yaml
$ vim CronJob.yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```

```bash
$ kubectl get pod
NAME                     READY   STATUS      RESTARTS   AGE
hello-1605672120-j8lgz   0/1     Completed   0          27s

$ kubectl get cronjob
NAME    SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello   */1 * * * *   False     0        47s             2m47s

$ kubectl logs hello-1605672120-j8lgz
Wed Nov 18 04:02:27 UTC 2020
Hello from the Kubernetes cluster
```