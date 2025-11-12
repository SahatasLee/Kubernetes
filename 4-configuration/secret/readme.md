# Secret

**Secret** ใน Kubernetes คือวัตถุ (Object) ที่ใช้สำหรับเก็บข้อมูลที่มีความสำคัญและต้องการความปลอดภัย เช่น รหัสผ่าน (password), โทเค็น (tokens), คีย์ SSH, หรือข้อมูลการตั้งค่าที่เป็นความลับอื่น ๆ `Secret` จะถูกเข้ารหัสในรูปแบบ Base64 และเก็บไว้ใน etcd ของคลัสเตอร์เพื่อความปลอดภัย

## ประเภทของ Secret

Kubernetes มีประเภทของ `Secret` หลายประเภทที่สามารถใช้ได้ตามความต้องการ:

1. **Opaque**: ประเภทที่ใช้ทั่วไป สามารถเก็บข้อมูลที่กำหนดเองได้ทุกประเภทในรูปแบบคีย์-ค่า (`key-value`).
2. **kubernetes.io/service-account-token**: ใช้สำหรับเก็บโทเค็นที่ผูกกับ `Service Account`.
3. **kubernetes.io/dockercfg** และ **kubernetes.io/dockerconfigjson**: ใช้สำหรับเก็บข้อมูลรับรอง (credentials) สำหรับการดึงภาพ Docker จากรีจิสทรีส่วนตัว.
4. **bootstrap.kubernetes.io/token**: ใช้สำหรับเก็บโทเค็นที่ใช้ในการ bootstrapping คลัสเตอร์.

## การสร้าง Secret

คุณสามารถสร้าง Secret ใน Kubernetes ได้โดยใช้ไฟล์ YAML หรือคำสั่ง `kubectl`.

### ตัวอย่างการสร้าง Secret โดยใช้ YAML

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: YWRtaW4=        # 'admin' ในรูปแบบ base64
  password: cGFzc3dvcmQ=    # 'password' ในรูปแบบ base64
```

ในตัวอย่างนี้, `username` และ `password` ถูกเข้ารหัสด้วย Base64 เพื่อความปลอดภัย.

**หมายเหตุ**: คุณสามารถใช้คำสั่ง `echo -n 'admin' | base64` เพื่อเข้ารหัสข้อความเป็น Base64 และ `echo 'YWRtaW4=' | base64 --decode` เพื่อถอดรหัสได้

### การสร้าง Secret โดยใช้คำสั่ง `kubectl`

คุณสามารถสร้าง Secret แบบ Opaque ด้วยคำสั่ง `kubectl` ได้ดังนี้:

```bash
kubectl create secret generic my-secret --from-literal=username=admin --from-literal=password=password
```

หรือถ้าคุณต้องการสร้างจากไฟล์:

```bash
kubectl create secret generic my-secret --from-file=username.txt --from-file=password.txt
```

## การใช้งาน Secret ใน Pods

Secret สามารถใช้งานใน Pods ได้หลายวิธี เช่น:

1. **การใช้ใน Environment Variables**:
   
คุณสามารถใช้ Secret เป็น Environment Variables ใน Container ได้:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: my-container
      image: nginx
      env:
        - name: USERNAME
          valueFrom:
            secretKeyRef:
              name: my-secret
              key: username
        - name: PASSWORD
          valueFrom:
            secretKeyRef:
              name: my-secret
              key: password
```

2. **การ Mount Secret เป็น Volume**:

Secret สามารถ Mount เข้าไปใน Pod เป็น Volume ได้เพื่อให้ Container สามารถอ่านข้อมูลได้:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: my-container
      image: nginx
      volumeMounts:
        - name: secret-volume
          mountPath: "/etc/secret"
  volumes:
    - name: secret-volume
      secret:
        secretName: my-secret
```

## ความปลอดภัยของ Secret

- ข้อมูลใน `Secret` จะถูกเข้ารหัสในรูปแบบ Base64 ซึ่งไม่ถือเป็นการเข้ารหัสที่แข็งแกร่ง แต่จะช่วยหลีกเลี่ยงการจัดการข้อมูลที่เป็นข้อความปกติ (plain text).
- Kubernetes สนับสนุนการเข้ารหัสข้อมูล `Secret` ใน etcd ด้วย encryption-at-rest เพื่อเพิ่มความปลอดภัย.
- การเข้าถึงข้อมูลใน `Secret` ควรควบคุมด้วย Role-Based Access Control (RBAC) เพื่อให้มั่นใจว่าเฉพาะผู้ใช้หรือ Pods ที่ได้รับอนุญาตเท่านั้นที่สามารถเข้าถึงได้.

## สรุป

- **Secret** ใช้เก็บข้อมูลที่เป็นความลับหรือข้อมูลที่สำคัญใน Kubernetes อย่างปลอดภัย
- สามารถใช้งานผ่าน Environment Variables หรือ Mount เป็น Volume ใน Pods
- ควรใช้ RBAC และการเข้ารหัสที่เหมาะสมเพื่อป้องกันการเข้าถึงที่ไม่พึงประสงค์.

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