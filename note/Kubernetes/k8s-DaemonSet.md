

* [一、案例](#%E4%B8%80%E6%A1%88%E4%BE%8B)


---
# 一、案例
```bash
$ vim DaemonSet.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ds-test 
  labels:
    app: filebeat
spec:
  selector:
    matchLabels:
      app: filebeat
  template:
    metadata:
      labels:
        app: filebeat
    spec:
      containers:
      - name: logs
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: varlog
          mountPath: /tmp/log
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```

```bash
$ kubectl get pod
NAME            READY   STATUS    RESTARTS   AGE
ds-test-bn6vf   1/1     Running   0          26s
ds-test-tvxwc   1/1     Running   0          26s

$ kubectl exec -it ds-test-bn6vf bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
root@ds-test-bn6vf:/# cd /tmp/log/
root@ds-test-bn6vf:/tmp/log# ls
anaconda	   btmp        cron-20201115  grubby		  maillog-20201115   rhsm	      spooler-20201115	    vmware-network.2.log  vmware-vgauthsvc.log.0
audit		   chrony      dmesg	      grubby_prune_debug  messages	     secure	      tallylog		    vmware-network.3.log  vmware-vmsvc.log
boot.log	   containers  dmesg.old      lastlog		  messages-20201115  secure-20201115  tuned		    vmware-network.4.log  wtmp
boot.log-20201113  cron        firewalld      maillog		  pods		     spooler	      vmware-network.1.log  vmware-network.log	  yum.log
```