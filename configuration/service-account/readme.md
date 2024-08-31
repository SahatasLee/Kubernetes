# Service Account

```bash
# Create service account
kubectl create serviceaccount <serviceaccount-name> 

# Get service account list
kubectl get serviceaccount
```

## 1.22

```bash
# TokenRequest API
kubectl create token <serviceaccount-name>
```

deployment.yaml

```yaml
spec:
    spec:
        serviceAccountName: dashboard-sa
```

## 1.24