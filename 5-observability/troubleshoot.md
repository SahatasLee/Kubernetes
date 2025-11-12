# Troubleshoot

```sh
# ดู pod status ก่อน
kubectl get pods

# ดู pod ก่อน
kubectl describe pod

# ถ้า pod pending หรือ crash → ดู event
kubectl get events --sort-by=.metadata.creationTimestamp

# ถ้า pod running แต่ app ไม่ทำงาน → ดู logs
kubectl logs myapp-pod -f

# ถ้ายังไม่เจอปัญหา → เข้า container ตรวจสอบโดยตรง
kubectl exec -it myapp-pod -- sh
```

## Logs



## Exec

```sh
kubectl exec -it <pod-name> -- <command>
kubectl exec -it myapp-pod -- /bin/bash
kubectl exec -it myapp-pod -- sh
kubectl exec -it myapp-pod -- ls /app
kubectl exec -it myapp-pod -- cat /etc/hosts
kubectl exec -it myapp-pod -- env
```