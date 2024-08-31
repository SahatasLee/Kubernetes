# Resources

## Limit Range

limit-range-cpu.yaml

```yaml
apiVersion: v1
kind: LimitRange
metadata:
    name: limit-range-cpu
spec:
    limits:
    - default:
        cpu: 500m
      defaultRequest:
        cpu: 500m
      max:
        cpu: 1
      min:
        cpu: 100m
      type: container
```

limit-range-memory.yaml

```yaml
apiVersion: v1
kind: LimitRange
metadata:
    name: limit-range-memory
spec:
    limits:
    - default:
        memory: 1Gi
      defaultRequest:
        memory: 1Gi
      max:
        memory: 1Gi
      min:
        memory: 500Mi
      type: container
```

## Resource Quota

resource-quota.yaml

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
    name: my-resource-quota
spec:
    hard:
        requests.cpu: 4
        requests.memory: 4Gi
        limit.cpu: 10
        limit.memory: 10Gi
```