# Labels & Annotations

**Labels** และ **Annotations** เป็นคุณสมบัติที่ใช้กับ Kubernetes Objects เช่น Pods, Services, และ Deployments เพื่อให้สามารถจัดการและกำหนดข้อมูลเพิ่มเติมให้กับ Objects นั้นได้ โดยมีความแตกต่างกันดังนี้:

### 1. **Labels (ป้ายกำกับ)**
- **วัตถุประสงค์**: ใช้สำหรับการจำแนก, การเลือก, และการกรอง (filter) Objects ต่าง ๆ ใน Kubernetes
- **ตัวอย่างการใช้งาน**: ใช้เพื่อกำหนดกลุ่มของ Pods ที่เป็นส่วนหนึ่งของ Service หรือ Deployment เดียวกัน
- **ข้อมูลที่จัดเก็บ**: มักจะเป็นข้อมูลที่มีความหมายต่อการจัดการหรือการประมวลผล เช่น `app=nginx`, `env=production`
- **การค้นหา**: สามารถใช้ `labels` ในการค้นหาหรือเลือก Objects ได้ด้วยการใช้ `kubectl` หรือเครื่องมืออื่น ๆ
- **รูปแบบการใช้งาน**:
  ```yaml
  metadata:
    labels:
      app: my-app
      environment: production
  ```

### 2. **Annotations (คำอธิบาย)**
- **วัตถุประสงค์**: ใช้เพื่อเก็บข้อมูลเพิ่มเติมหรือข้อมูล Metadata ที่ไม่ได้มีผลกับการเลือกหรือการกรอง Objects ใน Kubernetes
- **ตัวอย่างการใช้งาน**: ใช้เพื่อจัดเก็บข้อมูลต่าง ๆ เช่น URL, Checksum, หรือ Configuration ที่เกี่ยวข้องกับการทำงานของ Objects
- **ข้อมูลที่จัดเก็บ**: มักเป็นข้อมูลที่มีความหมายเฉพาะเจาะจงหรือข้อมูลสำหรับการอ้างอิงภายใน เช่น `description`, `maintainer`, หรือ `version`
- **การค้นหา**: ไม่สามารถใช้ Annotations ในการเลือก Objects โดยตรงผ่าน `kubectl`
- **รูปแบบการใช้งาน**:
  ```yaml
  metadata:
    annotations:
      description: "This is a sample application"
      maintainer: "admin@example.com"
  ```

### ความแตกต่างหลัก
- **Labels**: ใช้สำหรับการจัดการกลุ่มและการเลือก Objects อย่างมีประสิทธิภาพ
- **Annotations**: ใช้เพื่อเก็บข้อมูล Metadata ที่เป็นคำอธิบายเพิ่มเติม ซึ่งไม่จำเป็นต้องใช้ในการจัดการกลุ่มของ Objects

หากคุณต้องการจัดการหรือเลือก Objects ใน Kubernetes อย่างมีประสิทธิภาพ ควรใช้ **Labels** ในขณะที่ถ้าต้องการเก็บข้อมูลเพิ่มเติมที่ไม่ได้เกี่ยวข้องกับการจัดการหรือการเลือก Objects ควรใช้ **Annotations**.

คุณสามารถใช้ **Labels** เพื่อจำแนกและจัดกลุ่ม Pods ได้ ซึ่ง Labels เป็นคีย์-ค่าคู่ที่สามารถกำหนดให้กับ Objects ใน Kubernetes เช่น Pods, Services, หรือ Deployments เพื่อช่วยในการจัดการ, เลือก, หรือกรอง Objects เหล่านี้

### ตัวอย่างการใช้ Labels ในการจำแนก Pods

สมมติว่าเรามี 2 Pods ที่รันแอปพลิเคชันที่แตกต่างกันในสภาพแวดล้อมที่ต่างกัน (เช่น development และ production) เราสามารถใช้ Labels เพื่อระบุข้อมูลเหล่านี้ได้:

#### ไฟล์ YAML ตัวอย่าง: `deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
        environment: production  # Label เพื่อระบุสภาพแวดล้อม
    spec:
      containers:
        - name: nginx
          image: nginx
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
      environment: development
  template:
    metadata:
      labels:
        app: myapp
        environment: development  # Label เพื่อระบุสภาพแวดล้อม
    spec:
      containers:
        - name: nginx
          image: nginx
```

### การใช้งาน Labels กับคำสั่ง `kubectl`

หลังจากที่คุณได้สร้าง Pods ด้วยไฟล์ YAML ข้างต้นแล้ว คุณสามารถใช้คำสั่ง `kubectl` เพื่อแสดงเฉพาะ Pods ที่มี Labels ตรงตามที่ต้องการได้ เช่น:

1. **ดู Pods ทั้งหมดที่มี Label `app=myapp`**:

   ```bash
   kubectl get pods -l app=myapp
   ```

2. **ดู Pods ทั้งหมดในสภาพแวดล้อมการพัฒนา (`environment=development`)**:

   ```bash
   kubectl get pods -l environment=development
   ```

3. **ดู Pods ในสภาพแวดล้อมการผลิต (`environment=production`)**:

   ```bash
   kubectl get pods -l environment=production
   ```

### คำอธิบายเพิ่มเติม
- **Labels (`app` และ `environment`)**: ใช้เพื่อระบุประเภทของแอปพลิเคชัน (`app: myapp`) และสภาพแวดล้อม (`environment: production` หรือ `development`)
- คุณสามารถใช้ Labels เพื่อเลือก Pods ที่ต้องการสำหรับการทำงาน เช่น การอัปเดต, การปรับขนาด, หรือการแก้ไขข้อบกพร่อง.