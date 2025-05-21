# Role Base Access Control

```sh
# Role
kubectl get role

# RoleBinding
kubectl get rolebinding

# ClusterRole
kubectl get clusterrole

# ClusterRoleBinding
kubectl get clusterrolebinding

# Check Access
kubectl auth can-i create pod

# As user
kubectl auth can-i create pod --as dev-user
```

# Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata: 
    name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "create", "update"]
  resourceNames: ["blue", "orange"]
```