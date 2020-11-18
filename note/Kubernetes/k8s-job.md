



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