# Labels
 
 ใน Kubernetes (K8s) มีคอนเซปต์ของ **Labels**, **Selectors** และ **Annotations** ที่ใช้สำหรับการจัดการและค้นหาทรัพยากรต่าง ๆ ภายใน Cluster:

### 1. **Labels**:
- เป็นชุดของ key-value pairs ที่แนบไปกับ resources (เช่น Pods, Nodes, Services, และอื่น ๆ) เพื่อให้ระบุตัวตนของ resource นั้น ๆ ได้ง่ายขึ้น
- ใช้สำหรับการจัดกลุ่มและการจัดการ resource โดยสามารถเลือกหรือค้นหา resource ที่มี label ตรงตามที่ต้องการได้
- ตัวอย่าง label: `app: frontend`, `env: production`

  **การใช้งาน**: 
  ```yaml
  metadata:
    labels:
      app: frontend
      env: production
  ```

### 2. **Selectors**:
- ใช้ในการค้นหา resources ที่มี label ตรงกับเงื่อนไขที่กำหนด
- Selector สามารถใช้ได้กับหลายประเภทของ resources เช่น Deployments, Services, ReplicaSets เป็นต้น
- มีสองประเภทหลัก:
  - **Equality-based**: เช่น `app=frontend`, `env!=production`
  - **Set-based**: เช่น `env in (staging, production)`

  **ตัวอย่างการใช้ Selector**:  
  ```yaml
  selector:
    matchLabels:
      app: frontend
  ```

### 3. **Annotations**:
- เป็น metadata แบบ key-value pairs ที่สามารถแนบไปกับ resource ใด ๆ เพื่อให้มีข้อมูลเพิ่มเติม (metadata) แต่ไม่เกี่ยวข้องกับการค้นหาหรือเลือก resource (ตรงข้ามกับ labels)
- ใช้สำหรับเก็บข้อมูลเฉพาะเจาะจง เช่น ข้อมูลการตรวจสอบ (audit information), บันทึก, หรือข้อมูลเฉพาะอื่น ๆ

  **การใช้งาน**:
  ```yaml
  metadata:
    annotations:
      description: "This is a frontend service"
      version: "v1.2"
  ```

### ความแตกต่างระหว่าง Labels และ Annotations:
- **Labels**: ใช้สำหรับการค้นหาและจัดกลุ่ม resources
- **Annotations**: ใช้สำหรับเก็บข้อมูล metadata ที่ไม่ใช้ในการค้นหาหรือจัดการ resources

## Labels ใน Deployment

ใน Kubernetes, **Labels** ใน `Deployment` ใช้สำหรับจัดกลุ่มและระบุตัวตนของ Pods ที่สร้างขึ้นโดย Deployment นั้น ๆ ทำให้สามารถจัดการกับกลุ่ม Pods ได้ง่ายขึ้น โดยปกติจะใช้ร่วมกับ Selectors เพื่อเลือก Pods ที่มี Labels ตรงกับที่ระบุ

### การใช้งาน Labels ใน `Deployment`

1. **Labels ที่ระดับ Deployment**: Labels ที่กำหนดในส่วน `metadata` ของ `Deployment` ใช้เพื่อระบุตัวตนของ Deployment นั้น ๆ เอง

2. **Labels ที่ระดับ Pod Template**: Labels ที่กำหนดในส่วน `template.metadata.labels` ของ `Deployment` จะถูกส่งต่อไปยัง Pods ที่ถูกสร้างโดย Deployment นั้น ๆ

3. **Selector**: `Deployment` จะมี `selector` เพื่อเลือก Pods ที่มี Labels ตรงกับที่กำหนดใน `matchLabels` ซึ่งเป็นส่วนสำคัญในการทำงานของ `Deployment`

### ตัวอย่างการใช้ Labels ใน `Deployment`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
  labels:                        # Labels ระดับ Deployment
    app: frontend
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:                 # Selector ใช้เลือก Pods ที่มี Labels ตรงกัน
      app: frontend
  template:
    metadata:
      labels:                    # Labels ที่จะกำหนดให้กับ Pods ที่ถูกสร้างโดย Deployment นี้
        app: frontend
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:1.19.3
```

### คำอธิบาย:
- **metadata.labels**: ระบุตัวตนของ `Deployment` เอง
- **selector.matchLabels**: ใช้เลือก Pods ที่มี Labels ตรงกับเงื่อนไขที่กำหนด
- **template.metadata.labels**: Labels ที่กำหนดให้กับ Pods ที่สร้างจาก `Deployment` นี้

เมื่อ Kubernetes สร้าง Pods ขึ้นมาใน `Deployment` จะตรวจสอบว่า Pods มี Labels ที่ตรงกับ `selector.matchLabels` เพื่อเชื่อมโยง Pods กับ Deployment นั้น ๆ ซึ่งช่วยให้เราสามารถจัดกลุ่มและจัดการ Pods ได้ง่ายขึ้น.