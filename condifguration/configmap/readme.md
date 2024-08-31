# ConfigMap

Share Values

```bash
# Create from literal
kubectl create configmap <configmap-name> --from-literal=<key>=<value>
# Example
kubectl create configmap app-config --from-literal=APP_COLOR=blue --from-literal=APP_MODE=prod
# Create from file
kubectl create configmap <configmap-name> --from-file=<path-to-file>
# Example
kubectl create configmap <configmap-name> --from-file=app_config.properties
# 
kubectl create -f <path-to-file>
```

Env

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
    name: app-config
data:
    APP_COLOR: blue
    APP_MODE: prod
```

Configmap in Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: webapp
    labels:
        name: webapp
spec:
    container:
    - name: webapp
      image: 
      ports:
        - containerPort: 8080
      envFrom:
        - configMapRef:
            name: app-config
```

Single Env

```yaml

```

Volume

```yaml
volumes:
    - name: app-volume
      configMap:
        name: app-config
```

## Lab

Create WEB App and Set Env to WEB App