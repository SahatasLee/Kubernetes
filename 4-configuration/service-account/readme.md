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

Update

**Service Account** ใน Kubernetes เป็นบัญชีชนิดพิเศษที่ใช้โดย Pods เพื่อเข้าถึง API ของ Kubernetes และทรัพยากรอื่น ๆ ภายในคลัสเตอร์ โดย `Service Account` จะทำหน้าที่แทนแอปพลิเคชันที่ทำงานภายใน Pod ทำให้สามารถรับรองตัวตนและควบคุมการเข้าถึงทรัพยากรที่จำเป็นได้

## จุดประสงค์ของ Service Account

1. **การระบุตัวตน**: ใช้เพื่อระบุตัวตนของแอปพลิเคชันที่ทำงานใน Pods เพื่อให้ Kubernetes ทราบว่าผู้ใช้งานใดกำลังทำงานอยู่ในคลัสเตอร์
2. **การควบคุมการเข้าถึง (RBAC)**: ใช้ร่วมกับ Role-Based Access Control (RBAC) เพื่อควบคุมการเข้าถึงทรัพยากรของ Kubernetes โดยแอปพลิเคชันภายในคลัสเตอร์
3. **การแยกแอปพลิเคชัน**: ช่วยแยกสิทธิ์และขอบเขตการทำงานของแอปพลิเคชันหลายตัวที่รันอยู่ในคลัสเตอร์เดียวกัน เพื่อป้องกันไม่ให้แอปพลิเคชันเข้าถึงข้อมูลหรือทรัพยากรที่ไม่จำเป็น

## การสร้าง Service Account

คุณสามารถสร้าง `Service Account` ใหม่ได้โดยใช้ไฟล์ YAML หรือคำสั่ง `kubectl` เช่น:

### ตัวอย่างไฟล์ YAML สำหรับสร้าง Service Account

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-service-account
  namespace: default
```

จากนั้น ใช้คำสั่ง `kubectl` เพื่อสร้าง Service Account:

```bash
kubectl apply -f service-account.yaml
```

หรือใช้คำสั่ง `kubectl` เพื่อสร้าง Service Account โดยตรง:

```bash
kubectl create serviceaccount my-service-account --namespace=default
```

## การใช้ Service Account ใน Pods

หลังจากที่คุณสร้าง `Service Account` แล้ว คุณสามารถกำหนดให้ Pod ใช้ `Service Account` นั้นได้โดยการระบุในไฟล์ YAML ของ Pod หรือ Deployment:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  serviceAccountName: my-service-account  # ระบุชื่อ Service Account ที่จะใช้
  containers:
  - name: my-container
    image: nginx
```

## การจัดการการเข้าถึงด้วย RBAC

Kubernetes ใช้ `Role-Based Access Control (RBAC)` เพื่อจัดการการเข้าถึงทรัพยากรต่าง ๆ ภายในคลัสเตอร์ โดยสามารถกำหนด `Roles` หรือ `ClusterRoles` ให้กับ `Service Account` เพื่อควบคุมสิทธิ์ในการเข้าถึงทรัพยากร ตัวอย่างเช่น:

### สร้าง Role ที่กำหนดสิทธิ์เข้าถึงเฉพาะ

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: my-role
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

### สร้าง RoleBinding เพื่อเชื่อมโยง Role กับ Service Account

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: my-role-binding
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: my-role
subjects:
- kind: ServiceAccount
  name: my-service-account
  namespace: default
```

## สรุป

- **Service Account** เป็นวิธีในการให้แอปพลิเคชันที่ทำงานภายใน Pods สามารถเข้าถึง API ของ Kubernetes ได้อย่างปลอดภัยและมีการควบคุม
- ช่วยในการควบคุมสิทธิ์การเข้าถึงของแอปพลิเคชันในคลัสเตอร์ด้วยการใช้ร่วมกับ `RBAC`
- สามารถสร้างและใช้ Service Account ได้อย่างง่ายดายด้วยไฟล์ YAML หรือคำสั่ง `kubectl`.