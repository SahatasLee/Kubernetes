# Secret

## Create

```bash
# Imperative
kubectl create secret generic <secret-name> --from-literal=<key>=<value> \
    --from-literal=<key>=<value> \
    --from-literal=<key>=<value>
kubectl create secret generic <secret-name> --from-file=<path-to-file> 

# Declarative
kubectl create -f secret-data.yaml
```

Example: `secret-data.yaml`

```yaml
apiVersion: v1
kind: Secret
metadata:
    name: app-secret
data:
    DB_HOST: c3Fs # sql | base64
    DB_USER: cm9vdA== # root | base64
    DB_PASSWORD: cGFzc3dvcmQ= # password | base64
```

For `base64`

```bash
echo -n 'root' | base64
# decode
echo -n 'c3Fs' | base64 --decode ; echo
```

> The -n flag with echo prevents the addition of a newline character at the end of the string.

## Access

### Env

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: sql-app
    labels:
        name: sql-app
spec:
    containers:
    - name: sql-app
      image: sql
      ports:
        - containerPort: 443
    envFrom:
      - secretRef:
            name: app-secret # same name with secret name
```

### Single Env

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: sql-app
    labels:
        name: sql-app
spec:
    containers:
    - name: sql-app
      image: sql
      ports:
        - containerPort: 443
    env:
      - name: DB_HOST
        valueFrom:
            secretKeyRef:
              name: app-secret # same name with secret name
              key: DB_HOST
```

### Volume

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: sql-app
    labels:
        name: sql-app
spec:
    containers:
    - name: sql-app
      image: sql
      ports:
        - containerPort: 443
    volume:
    - name: app-secret-volume
      secret:
        secretName: app-secret # same name with secret name
```

Inside the Container

```bash
> ls /opt/app-secret-volumes
DB_HOST DB_USER DB_PASSWORD
> cat /opt/app-secret-volumes/DB_HOST
sql
```

## Note

- Secret are not Encryted. Only encoded.
- Do not check-in Secret objects to SCM along with code
- Secrets are not encryted in ETCD
    - Enable encryption at rest
- Anyone able to create pods/deployments in the same namepace cam access the secrets
    - Configure least-privilege access to Secret - RBAC
- Consider third-party secret store providers
    - AWS Provider, Azure Provider, GCP Provider, Vault Provider