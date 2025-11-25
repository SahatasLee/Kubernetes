# Kubeconfig File

## บทนำ

Kubeconfig เป็นไฟล์ที่ใช้ในการจัดเก็บข้อมูลการเชื่อมต่อกับ Kubernetes Cluster โดยไฟล์นี้จะบอกให้ `kubectl` รู้ว่าจะเชื่อมต่อไปที่ cluster ไหน ใช้ user credentials อะไร และจะใช้ context ใดในการทำงาน

ตำแหน่งเริ่มต้นของไฟล์: `~/.kube/config`

## โครงสร้างของ Kubeconfig

Kubeconfig ประกอบด้วย 3 ส่วนหลัก:

### 1. Clusters
เก็บข้อมูล Kubernetes Cluster ที่ต้องการเชื่อมต่อ

### 2. Users
เก็บข้อมูล credentials ของผู้ใช้งาน

### 3. Contexts
เชื่อมโยง cluster กับ user และ namespace เข้าด้วยกัน

## โครงสร้างไฟล์แบบละเอียด

```yaml
apiVersion: v1
kind: Config
current-context: my-context  # context ที่กำลังใช้งานอยู่

# กำหนดรายชื่อ cluster ทั้งหมด
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTi...  # CA cert ในรูปแบบ base64
    # หรือใช้ certificate-authority: /path/to/ca.crt
    server: https://kubernetes-api-server:6443  # URL ของ API server
  name: my-cluster  # ชื่อ cluster

# กำหนดรายชื่อ user ทั้งหมด
users:
- name: my-user  # ชื่อ user
  user:
    client-certificate-data: LS0tLS1CRUdJTi...  # Client cert ในรูปแบบ base64
    client-key-data: LS0tLS1CRUdJTi...  # Client key ในรูปแบบ base64
    # หรือใช้แบบ path
    # client-certificate: /path/to/client.crt
    # client-key: /path/to/client.key

# กำหนด context เพื่อเชื่อม cluster กับ user
contexts:
- context:
    cluster: my-cluster  # ชื่อ cluster ที่อ้างอิง
    user: my-user  # ชื่อ user ที่อ้างอิง
    namespace: default  # namespace เริ่มต้น (optional)
  name: my-context  # ชื่อ context
```

## รูปแบบ Authentication ต่างๆ

### 1. X.509 Client Certificates (แนะนำ)

```yaml
users:
- name: user-with-cert
  user:
    client-certificate-data: LS0tLS1CRUdJTi...
    client-key-data: LS0tLS1CRUdJTi...
```

### 2. Bearer Token

```yaml
users:
- name: user-with-token
  user:
    token: eyJhbGciOiJSUzI1NiIsImtpZCI6Ii...
```

### 3. Username และ Password (ไม่แนะนำ)

```yaml
users:
- name: user-basic-auth
  user:
    username: admin
    password: secret-password
```

### 4. Exec Plugin (สำหรับ authentication plugins)

```yaml
users:
- name: user-with-plugin
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1beta1
      command: aws-iam-authenticator
      args:
        - "token"
        - "-i"
        - "my-cluster"
```

### 5. OIDC (OpenID Connect)

```yaml
users:
- name: user-oidc
  user:
    auth-provider:
      name: oidc
      config:
        client-id: kubernetes
        client-secret: secret
        id-token: eyJhbGciOiJSUzI1NiIsImtpZCI6...
        idp-issuer-url: https://accounts.google.com
        refresh-token: eyJhbGciOiJSUzI1NiIsImtpZCI6...
```

## ตัวอย่างไฟล์ Kubeconfig แบบเต็ม

```yaml
apiVersion: v1
kind: Config
current-context: dev-context

clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ0F3SUJBZ0lJZmVHdHdVQldIQXN3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TkRFeE1qUXdNREF3TURCYUZ3MHpOREV4TWpJd01EQTFNREJhTUJVeApFekFSQmdOVkJBTVRDbXQxWW1WeWJtVjBaWE13Z2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQkR3QXdnZ0VLCkFvSUJBUUM3...
    server: https://dev-cluster.example.com:6443
  name: development-cluster

- cluster:
    certificate-authority: /home/user/.kube/prod-ca.crt
    server: https://prod-cluster.example.com:6443
  name: production-cluster

users:
- name: dev-admin
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURJVENDQWdtZ0F3SUJBZ0lJY1N...
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFcEFJQkFBS0NBUUVBdXpO...

- name: dev-viewer
  user:
    client-certificate: /home/user/.kube/dev-viewer.crt
    client-key: /home/user/.kube/dev-viewer.key

- name: prod-admin
  user:
    token: eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImFkbWluLXRva2VuLXh4eHgiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiYWRtaW4iLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI...

contexts:
- context:
    cluster: development-cluster
    user: dev-admin
    namespace: default
  name: dev-context

- context:
    cluster: development-cluster
    user: dev-viewer
    namespace: development
  name: dev-viewer-context

- context:
    cluster: production-cluster
    user: prod-admin
    namespace: production
  name: prod-context
```

## คำสั่ง kubectl config ที่ใช้บ่อย

### ดู Configuration

```bash
# ดู config ทั้งหมด
kubectl config view

# ดู config แบบไม่ซ่อนข้อมูลที่เป็นความลับ
kubectl config view --raw

# ดู config ของไฟล์เฉพาะ
kubectl config view --kubeconfig=/path/to/custom-config
```

### จัดการ Context

```bash
# ดูรายการ context ทั้งหมด
kubectl config get-contexts

# ดู current context
kubectl config current-context

# เปลี่ยน context
kubectl config use-context dev-context

# สร้าง context ใหม่
kubectl config set-context my-new-context \
  --cluster=my-cluster \
  --user=my-user \
  --namespace=my-namespace

# ลบ context
kubectl config delete-context dev-context
```

### จัดการ Cluster

```bash
# เพิ่ม cluster
kubectl config set-cluster my-cluster \
  --server=https://kubernetes-api.example.com:6443 \
  --certificate-authority=/path/to/ca.crt

# หรือใช้ certificate-authority-data
kubectl config set-cluster my-cluster \
  --server=https://kubernetes-api.example.com:6443 \
  --certificate-authority-data=$(cat /path/to/ca.crt | base64 -w 0)

# ลบ cluster
kubectl config delete-cluster my-cluster
```

### จัดการ User

```bash
# เพิ่ม user ด้วย certificate
kubectl config set-credentials my-user \
  --client-certificate=/path/to/client.crt \
  --client-key=/path/to/client.key

# เพิ่ม user ด้วย token
kubectl config set-credentials my-user \
  --token=eyJhbGciOiJSUzI1NiIsImtpZCI6...

# เพิ่ม user ด้วย username/password
kubectl config set-credentials my-user \
  --username=admin \
  --password=secret

# ลบ user
kubectl config unset users.my-user
```

### ตั้งค่าเพิ่มเติม

```bash
# ตั้งค่า namespace เริ่มต้นสำหรับ context
kubectl config set-context --current --namespace=my-namespace

# ตั้งค่า current context
kubectl config use-context prod-context
```

## การใช้หลาย Kubeconfig Files

### วิธีที่ 1: ใช้ Environment Variable

```bash
# ใช้ไฟล์เดียว
export KUBECONFIG=/path/to/config

# ใช้หลายไฟล์พร้อมกัน (merge)
export KUBECONFIG=/path/to/config1:/path/to/config2:/path/to/config3

# ใช้กับคำสั่งเดียว
KUBECONFIG=/path/to/custom-config kubectl get pods
```

### วิธีที่ 2: ใช้ --kubeconfig flag

```bash
kubectl --kubeconfig=/path/to/custom-config get pods
```

### วิธีที่ 3: Merge หลายไฟล์

```bash
# Merge และบันทึกเป็นไฟล์ใหม่
KUBECONFIG=config1:config2:config3 kubectl config view --flatten > merged-config

# Merge และเขียนทับไฟล์เดิม
KUBECONFIG=~/.kube/config:new-config kubectl config view --flatten > /tmp/merged
mv /tmp/merged ~/.kube/config
```

## ตัวอย่างการสร้าง Kubeconfig สำหรับ User ใหม่

### สคริปต์สร้าง Kubeconfig แบบสมบูรณ์

```bash
#!/bin/bash

# ตัวแปร
USER_NAME="john-viewer"
CLUSTER_NAME="my-cluster"
CONTEXT_NAME="${USER_NAME}@${CLUSTER_NAME}"
NAMESPACE="default"
KUBECONFIG_FILE="${USER_NAME}-kubeconfig.yaml"

# ดึงข้อมูลจาก current config
CLUSTER_CA=$(kubectl config view --raw -o jsonpath='{.clusters[0].cluster.certificate-authority-data}')
CLUSTER_SERVER=$(kubectl config view --raw -o jsonpath='{.clusters[0].cluster.server}')

# สร้าง Certificate (สมมติว่ามี cert และ key แล้ว)
CLIENT_CERT=$(cat ${USER_NAME}.crt | base64 -w 0)
CLIENT_KEY=$(cat ${USER_NAME}.key | base64 -w 0)

# สร้างไฟล์ kubeconfig
cat > ${KUBECONFIG_FILE} <<EOF
apiVersion: v1
kind: Config
current-context: ${CONTEXT_NAME}

clusters:
- cluster:
    certificate-authority-data: ${CLUSTER_CA}
    server: ${CLUSTER_SERVER}
  name: ${CLUSTER_NAME}

users:
- name: ${USER_NAME}
  user:
    client-certificate-data: ${CLIENT_CERT}
    client-key-data: ${CLIENT_KEY}

contexts:
- context:
    cluster: ${CLUSTER_NAME}
    user: ${USER_NAME}
    namespace: ${NAMESPACE}
  name: ${CONTEXT_NAME}
EOF

echo "Kubeconfig file created: ${KUBECONFIG_FILE}"
```

### วิธีใช้คำสั่ง kubectl config

```bash
#!/bin/bash

USER_NAME="john-viewer"
CLUSTER_NAME="my-cluster"
KUBECONFIG_FILE="${USER_NAME}-kubeconfig.yaml"

# ดึงข้อมูล cluster จาก current config
CLUSTER_SERVER=$(kubectl config view --raw -o jsonpath='{.clusters[0].cluster.server}')
CLUSTER_CA_FILE="/path/to/ca.crt"

# สร้างไฟล์ใหม่
export KUBECONFIG=${KUBECONFIG_FILE}

# ตั้งค่า cluster
kubectl config set-cluster ${CLUSTER_NAME} \
  --server=${CLUSTER_SERVER} \
  --certificate-authority=${CLUSTER_CA_FILE} \
  --embed-certs=true

# ตั้งค่า user credentials
kubectl config set-credentials ${USER_NAME} \
  --client-certificate=${USER_NAME}.crt \
  --client-key=${USER_NAME}.key \
  --embed-certs=true

# สร้าง context
kubectl config set-context ${USER_NAME}@${CLUSTER_NAME} \
  --cluster=${CLUSTER_NAME} \
  --user=${USER_NAME} \
  --namespace=default

# ตั้งค่า current context
kubectl config use-context ${USER_NAME}@${CLUSTER_NAME}

echo "Kubeconfig file created: ${KUBECONFIG_FILE}"
```

## Best Practices

### 1. รักษาความปลอดภัย

```bash
# ตั้งค่า permission ที่ถูกต้อง
chmod 600 ~/.kube/config

# อย่าเก็บไฟล์ใน Git
echo ".kube/*" >> .gitignore
```

### 2. ใช้ Namespace ที่เหมาะสม

```yaml
contexts:
- context:
    cluster: my-cluster
    user: dev-user
    namespace: development  # ระบุ namespace ชัดเจน
  name: dev-context
```

### 3. ตั้งชื่อที่มีความหมาย

```yaml
# ดี - มีความหมายชัดเจน
contexts:
- context:
    cluster: production-cluster
    user: admin-user
    namespace: production
  name: prod-admin

# ไม่ดี - ชื่อไม่มีความหมาย
contexts:
- context:
    cluster: cluster1
    user: user1
    namespace: ns1
  name: context1
```

### 4. Embed Certificates

```bash
# เมื่อต้องการแชร์ไฟล์ kubeconfig ให้ใช้ --embed-certs
kubectl config set-cluster my-cluster \
  --server=https://api.example.com:6443 \
  --certificate-authority=ca.crt \
  --embed-certs=true
```

### 5. แยก Config ตาม Environment

```bash
# Development
~/.kube/config-dev

# Staging
~/.kube/config-staging

# Production
~/.kube/config-prod

# Merge เมื่อใช้งาน
export KUBECONFIG=~/.kube/config-dev:~/.kube/config-staging:~/.kube/config-prod
```

## Troubleshooting

### ปัญหา: Unable to connect to the server

```bash
# ตรวจสอบ server URL
kubectl config view -o jsonpath='{.clusters[0].cluster.server}'

# ทดสอบการเชื่อมต่อ
curl -k https://kubernetes-api-server:6443
```

### ปัญหา: x509: certificate signed by unknown authority

```bash
# ตรวจสอบ CA certificate
kubectl config view --raw -o jsonpath='{.clusters[0].cluster.certificate-authority-data}' | base64 -d

# หรือตั้งค่าให้ skip TLS verification (ไม่แนะนำ)
kubectl config set-cluster my-cluster --insecure-skip-tls-verify=true
```

### ปัญหา: User ไม่มีสิทธิ์

```bash
# ตรวจสอบ user credentials
kubectl config view --raw

# ทดสอบการ authenticate
kubectl auth whoami

# ตรวจสอบสิทธิ์
kubectl auth can-i get pods
kubectl auth can-i create deployments
```

### ปัญหา: Context ไม่ถูกต้อง

```bash
# แสดง current context
kubectl config current-context

# แสดง context ทั้งหมด
kubectl config get-contexts

# เปลี่ยน context
kubectl config use-context correct-context
```

## เครื่องมือเสริม

### kubectx และ kubens

ช่วยในการสลับ context และ namespace ได้ง่ายขึ้น

```bash
# ติดตั้ง
brew install kubectx

# ใช้งาน kubectx (สำหรับ context)
kubectx  # แสดงรายการ context
kubectx dev-context  # เปลี่ยนไป dev-context
kubectx -  # กลับไป context ก่อนหน้า

# ใช้งาน kubens (สำหรับ namespace)
kubens  # แสดงรายการ namespace
kubens development  # เปลี่ยนไป namespace development
kubens -  # กลับไป namespace ก่อนหน้า
```

### k9s

Terminal UI สำหรับจัดการ Kubernetes

```bash
# ติดตั้ง
brew install k9s

# ใช้งาน
k9s  # จะใช้ current context
k9s --context dev-context  # ระบุ context
```

### kubeconfig-merge

ช่วยในการ merge หลายไฟล์ kubeconfig

```bash
# ติดตั้ง
go install github.com/kubeconfig-merge/kubeconfig-merge@latest

# ใช้งาน
kubeconfig-merge config1.yaml config2.yaml > merged.yaml
```

## สรุป

Kubeconfig เป็นไฟล์สำคัญที่ใช้ในการกำหนดค่าการเชื่อมต่อกับ Kubernetes Cluster โดยมีองค์ประกอบหลัก 3 ส่วน:

1. **Clusters** - ข้อมูล Kubernetes API Server
2. **Users** - ข้อมูล Authentication
3. **Contexts** - การเชื่อมโยง Cluster, User และ Namespace

การจัดการ kubeconfig ที่ดีจะช่วยให้:
- สลับระหว่าง cluster ต่างๆ ได้ง่าย
- แยกสิทธิ์การเข้าถึงได้ชัดเจน
- รักษาความปลอดภัยของ credentials
- ทำงานกับหลาย environment ได้สะดวก

จำไว้ว่า kubeconfig เป็นไฟล์ที่มีข้อมูลสำคัญ ควรเก็บรักษาให้ปลอดภัยและไม่ควรเผยแพร่ในที่สาธารณะ
