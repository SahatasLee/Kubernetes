# การสร้าง User บน Kubernetes

คู่มือนี้จะอธิบายวิธีการสร้าง user บน Kubernetes และการผูก Role/ClusterRole เพื่อกำหนดสิทธิ์การเข้าถึง พร้อมทั้งวิธีสร้าง kubeconfig สำหรับ user

## สารบัญ
- [ภาพรวม](#ภาพรวม)
- [ขั้นตอนการสร้าง User](#ขั้นตอนการสร้าง-user)
- [การสร้างและผูก Role](#การสร้างและผูก-role)
- [ตัวอย่าง: View Only User](#ตัวอย่าง-view-only-user)
- [ตัวอย่าง: Admin User](#ตัวอย่าง-admin-user)
- [การสร้าง Kubeconfig](#การสร้าง-kubeconfig)

---

## ภาพรวม

Kubernetes ไม่มีระบบจัดการ user แบบ built-in แต่ใช้วิธี authentication ผ่าน:
- **X.509 Client Certificates** (แนะนำสำหรับ user)
- Service Account (สำหรับ Pod/Application)
- Token-based authentication
- OIDC, LDAP, etc.

คู่มือนี้จะใช้วิธี **X.509 Client Certificates** ซึ่งเป็นวิธีที่นิยมใช้กันมากที่สุด

---

## ขั้นตอนการสร้าง User

### 1. สร้าง Private Key สำหรับ User

```bash
# สร้าง private key สำหรับ user (เช่น john)
openssl genrsa -out john.key 2048
```

### 2. สร้าง Certificate Signing Request (CSR)

```bash
# สร้าง CSR โดยระบุ username ใน CN (Common Name)
openssl req -new -key john.key -out john.csr -subj "/CN=john/O=developers"
```

**หมายเหตุ:**
- `CN` (Common Name) = username ที่จะใช้ใน Kubernetes
- `O` (Organization) = group ที่ user นี้เป็นสมาชิก (ใช้สำหรับผูก RoleBinding)

### 3. สร้าง CertificateSigningRequest ใน Kubernetes

สร้างไฟล์ `john-csr.yaml`:

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: john
spec:
  request: <BASE64_ENCODED_CSR>
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 31536000  # 1 ปี
  usages:
  - client auth
```

**เตรียม CSR ที่ encode เป็น base64:**

```bash
# แปลง CSR เป็น base64 (single line)
cat john.csr | base64 | tr -d "\n"
```

**หรือสร้างไฟล์ CSR โดยอัตโนมัติ:**

```bash
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: john
spec:
  request: $(cat john.csr | base64 | tr -d "\n")
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 31536000
  usages:
  - client auth
EOF
```

### 4. Approve CSR

```bash
# ตรวจสอบ CSR
kubectl get csr

# อนุมัติ CSR
kubectl certificate approve john

# ดึง certificate ที่ signed แล้ว
kubectl get csr john -o jsonpath='{.status.certificate}' | base64 -d > john.crt
```

---

## การสร้างและผูก Role

### Role vs ClusterRole

- **Role**: จำกัดสิทธิ์เฉพาะใน namespace เดียว
- **ClusterRole**: สิทธิ์ที่ใช้ได้ทั้ง cluster (ทุก namespace)

### RoleBinding vs ClusterRoleBinding

- **RoleBinding**: ผูก Role/ClusterRole กับ user ใน namespace เฉพาะ
- **ClusterRoleBinding**: ผูก ClusterRole กับ user ทั้ง cluster

---

## ตัวอย่าง: View Only User

### วิธีที่ 1: ใช้ ClusterRole ที่มีอยู่แล้ว (แนะนำ)

Kubernetes มี built-in ClusterRole `view` ที่ให้สิทธิ์อ่านอย่างเดียว:

```bash
# ผูก ClusterRole "view" กับ user john ใน namespace "default"
kubectl create rolebinding john-view \
  --clusterrole=view \
  --user=john \
  --namespace=default
```

**สำหรับทุก namespace:**

```bash
# ผูกสิทธิ์ view ทั้ง cluster
kubectl create clusterrolebinding john-view-all \
  --clusterrole=view \
  --user=john
```

### วิธีที่ 2: สร้าง Custom Role

สร้างไฟล์ `view-role.yaml`:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "watch"]
```

**สร้าง RoleBinding:**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: john
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

**Apply:**

```bash
kubectl apply -f view-role.yaml
```

---

## ตัวอย่าง: Admin User

### วิธีที่ 1: ใช้ ClusterRole ที่มีอยู่แล้ว (แนะนำ)

Kubernetes มี built-in ClusterRole หลายแบบ:
- `admin`: สิทธิ์ admin ใน namespace
- `cluster-admin`: สิทธิ์ admin ทั้ง cluster (superuser)

**Admin ใน namespace เดียว:**

```bash
kubectl create rolebinding john-admin \
  --clusterrole=admin \
  --user=john \
  --namespace=default
```

**Cluster Admin (ทั้ง cluster):**

```bash
kubectl create clusterrolebinding john-cluster-admin \
  --clusterrole=cluster-admin \
  --user=john
```

### วิธีที่ 2: สร้าง Custom ClusterRole

สร้างไฟล์ `custom-admin-role.yaml`:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: custom-admin
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: john-custom-admin
subjects:
- kind: User
  name: john
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: custom-admin
  apiGroup: rbac.authorization.k8s.io
```

**Apply:**

```bash
kubectl apply -f custom-admin-role.yaml
```

---

## การสร้าง Kubeconfig

### วิธีที่ 1: Manual (เข้าใจได้ง่าย)

#### ขั้นตอนที่ 1: ดึงข้อมูล Cluster

```bash
# ดูข้อมูล cluster ปัจจุบัน
kubectl config view --flatten --minify

# ดึง API Server URL
CLUSTER_SERVER=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}')

# ดึง CA Certificate
kubectl config view --flatten --minify -o jsonpath='{.clusters[0].cluster.certificate-authority-data}' | base64 -d > ca.crt
```

#### ขั้นตอนที่ 2: สร้าง Kubeconfig

สร้างไฟล์ `john-kubeconfig.yaml`:

```yaml
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority-data: <CA_CERT_BASE64>
    server: <CLUSTER_SERVER>
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: john
    namespace: default
  name: john@kubernetes
current-context: john@kubernetes
users:
- name: john
  user:
    client-certificate-data: <USER_CERT_BASE64>
    client-key-data: <USER_KEY_BASE64>
```

**หรือใช้คำสั่ง kubectl:**

```bash
# สร้าง kubeconfig ใหม่
export KUBECONFIG=john-kubeconfig

# เพิ่ม cluster
kubectl config set-cluster kubernetes \
  --server=$CLUSTER_SERVER \
  --certificate-authority=ca.crt \
  --embed-certs=true

# เพิ่ม user credentials
kubectl config set-credentials john \
  --client-certificate=john.crt \
  --client-key=john.key \
  --embed-certs=true

# สร้าง context
kubectl config set-context john@kubernetes \
  --cluster=kubernetes \
  --user=john \
  --namespace=default

# ตั้งเป็น default context
kubectl config use-context john@kubernetes
```

### วิธีที่ 2: Script แบบอัตโนมัติ

สร้าง script `create-user.sh`:

```bash
#!/bin/bash

USERNAME=$1
NAMESPACE=${2:-default}
ROLE=${3:-view}  # view, admin, หรือ cluster-admin

if [ -z "$USERNAME" ]; then
    echo "Usage: $0 <username> [namespace] [role]"
    exit 1
fi

echo "Creating user: $USERNAME"
echo "Namespace: $NAMESPACE"
echo "Role: $ROLE"

# 1. สร้าง private key
openssl genrsa -out ${USERNAME}.key 2048

# 2. สร้าง CSR
openssl req -new -key ${USERNAME}.key -out ${USERNAME}.csr -subj "/CN=${USERNAME}/O=developers"

# 3. สร้าง CSR ใน Kubernetes
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: ${USERNAME}
spec:
  request: $(cat ${USERNAME}.csr | base64 | tr -d "\n")
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 31536000
  usages:
  - client auth
EOF

# 4. Approve CSR
kubectl certificate approve ${USERNAME}

# 5. ดึง certificate
kubectl get csr ${USERNAME} -o jsonpath='{.status.certificate}' | base64 -d > ${USERNAME}.crt

# 6. สร้าง RoleBinding
if [ "$ROLE" == "cluster-admin" ]; then
    kubectl create clusterrolebinding ${USERNAME}-${ROLE} \
        --clusterrole=${ROLE} \
        --user=${USERNAME} \
        --dry-run=client -o yaml | kubectl apply -f -
else
    kubectl create rolebinding ${USERNAME}-${ROLE} \
        --clusterrole=${ROLE} \
        --user=${USERNAME} \
        --namespace=${NAMESPACE} \
        --dry-run=client -o yaml | kubectl apply -f -
fi

# 7. สร้าง kubeconfig
CLUSTER_SERVER=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}')
CLUSTER_CA=$(kubectl config view --flatten --minify -o jsonpath='{.clusters[0].cluster.certificate-authority-data}')
USER_CERT=$(cat ${USERNAME}.crt | base64 | tr -d "\n")
USER_KEY=$(cat ${USERNAME}.key | base64 | tr -d "\n")

cat > ${USERNAME}-kubeconfig.yaml <<EOF
apiVersion: v1
kind: Config
clusters:
- cluster:
    certificate-authority-data: ${CLUSTER_CA}
    server: ${CLUSTER_SERVER}
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: ${USERNAME}
    namespace: ${NAMESPACE}
  name: ${USERNAME}@kubernetes
current-context: ${USERNAME}@kubernetes
users:
- name: ${USERNAME}
  user:
    client-certificate-data: ${USER_CERT}
    client-key-data: ${USER_KEY}
EOF

echo "✅ User ${USERNAME} created successfully!"
echo "📁 Files created:"
echo "   - ${USERNAME}.key (private key)"
echo "   - ${USERNAME}.crt (certificate)"
echo "   - ${USERNAME}-kubeconfig.yaml (kubeconfig)"
echo ""
echo "Test access:"
echo "   kubectl --kubeconfig=${USERNAME}-kubeconfig.yaml get pods"
```

**วิธีใช้งาน:**

```bash
chmod +x create-user.sh

# สร้าง view-only user
./create-user.sh john default view

# สร้าง admin user
./create-user.sh sarah default admin

# สร้าง cluster-admin user
./create-user.sh alice default cluster-admin
```

---

## ทดสอบการใช้งาน

### ทดสอบด้วย kubeconfig ที่สร้าง

```bash
# ทดสอบ view-only user
kubectl --kubeconfig=john-kubeconfig.yaml get pods
kubectl --kubeconfig=john-kubeconfig.yaml get deployments

# ลองสร้าง pod (ควรจะไม่สำเร็จถ้าเป็น view-only)
kubectl --kubeconfig=john-kubeconfig.yaml run nginx --image=nginx

# ทดสอบ admin user
kubectl --kubeconfig=sarah-kubeconfig.yaml run nginx --image=nginx
kubectl --kubeconfig=sarah-kubeconfig.yaml delete pod nginx
```

### ตรวจสอบสิทธิ์

```bash
# ตรวจสอบว่า user สามารถทำอะไรได้บ้าง
kubectl --kubeconfig=john-kubeconfig.yaml auth can-i get pods
kubectl --kubeconfig=john-kubeconfig.yaml auth can-i create pods
kubectl --kubeconfig=john-kubeconfig.yaml auth can-i delete deployments

# ดูสิทธิ์ทั้งหมด
kubectl --kubeconfig=john-kubeconfig.yaml auth can-i --list
```

---

## Built-in ClusterRoles ที่น่าสนใจ

Kubernetes มี ClusterRole ที่สร้างไว้แล้วให้ใช้:

| ClusterRole | คำอธิบาย |
|-------------|----------|
| `view` | อ่านได้เกือบทุกอย่าง ยกเว้น Secrets และ RoleBindings |
| `edit` | อ่านและแก้ไขได้เกือบทุกอย่าง แต่ไม่สามารถแก้ไข Roles/RoleBindings |
| `admin` | สิทธิ์ admin ใน namespace (สามารถจัดการ Roles/RoleBindings) |
| `cluster-admin` | สิทธิ์สูงสุดทั้ง cluster (superuser) |

**ดู ClusterRole ทั้งหมด:**

```bash
kubectl get clusterroles
```

**ดูรายละเอียด ClusterRole:**

```bash
kubectl describe clusterrole view
kubectl describe clusterrole admin
kubectl describe clusterrole cluster-admin
```

---

## การลบ User

```bash
# 1. ลบ RoleBinding/ClusterRoleBinding
kubectl delete rolebinding john-view
kubectl delete clusterrolebinding john-cluster-admin

# 2. ลบ CSR
kubectl delete csr john

# 3. ลบไฟล์
rm -f john.key john.csr john.crt john-kubeconfig.yaml
```

---

## Best Practices

1. **ใช้ Principle of Least Privilege**: ให้สิทธิ์เท่าที่จำเป็นเท่านั้น
2. **ใช้ Namespaces**: แยก environment และกำหนดสิทธิ์ตาม namespace
3. **ใช้ Groups**: กำหนด Organization (O) ใน CSR และผูก RoleBinding กับ group แทน user แต่ละคน
4. **Certificate Expiration**: ตั้งค่า `expirationSeconds` ให้เหมาะสม และมีกระบวนการ renew
5. **เก็บ Private Key ให้ปลอดภัย**: อย่าเก็บใน git repository
6. **Audit**: ใช้ `kubectl auth can-i --list` เพื่อตรวจสอบสิทธิ์

---

## ตัวอย่างการใช้งานจริง

### Scenario 1: Developer Team

```bash
# สร้าง developers group ที่สามารถ deploy และ debug ได้ใน namespace "development"
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: developers-edit
  namespace: development
subjects:
- kind: Group
  name: developers
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: edit
  apiGroup: rbac.authorization.k8s.io
EOF

# สร้าง user ที่อยู่ใน developers group
openssl req -new -key dev1.key -out dev1.csr -subj "/CN=dev1/O=developers"
```

### Scenario 2: Read-only Monitoring

```bash
# สร้าง monitoring user ที่อ่านได้อย่างเดียวทั้ง cluster
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: monitoring-view
subjects:
- kind: User
  name: prometheus
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: view
  apiGroup: rbac.authorization.k8s.io
EOF
```

### Scenario 3: Namespace Admin

```bash
# สร้าง namespace admin ที่มีสิทธิ์เต็มใน namespace "production" เท่านั้น
kubectl create rolebinding prod-admin \
  --clusterrole=admin \
  --user=prod-admin \
  --namespace=production
```

---

## สรุป

การสร้าง user บน Kubernetes ประกอบด้วย:

1. ✅ สร้าง private key และ CSR
2. ✅ Submit และ approve CSR ใน Kubernetes
3. ✅ สร้าง Role/ClusterRole (หรือใช้ built-in)
4. ✅ สร้าง RoleBinding/ClusterRoleBinding
5. ✅ สร้าง kubeconfig สำหรับ user
6. ✅ ทดสอบสิทธิ์ด้วย `kubectl auth can-i`

Script และตัวอย่างในคู่มือนี้สามารถนำไปปรับใช้ได้ตามความต้องการ 🚀
