




* [一、一次性任务](#%E4%B8%80%E4%B8%80%E6%AC%A1%E6%80%A7%E4%BB%BB%E5%8A%A1)

--- 
# 一、一次性任务
```yaml
$ vim job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
```
backoffLimit: 失败重试次数(默认是6)

```bash
$ kubectl apply -f job.yaml

$ kubectl get pod
NAME       READY   STATUS    RESTARTS   AGE
pi-blclg   1/1     Running   0          56s

$ kubectl get job
NAME   COMPLETIONS   DURATION   AGE
pi     1/1           58s        96s
```