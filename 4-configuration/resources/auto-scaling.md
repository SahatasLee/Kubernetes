# Auto Scaling

ยอดเยี่ยมมากครับ 🔥
หัวข้อนี้คือ “**Kubernetes Autoscaling**” — หัวใจสำคัญของการทำ **ระบบให้ขยายตัวอัตโนมัติ (Elastic Scaling)**

ใน Kubernetes มีการ “Auto Scale” อยู่ 3 ระดับหลัก ๆ ได้แก่ 👇

| ประเภท                 | ย่อจาก                    | ระดับที่ควบคุม      | หน้าที่หลัก                              |
| :--------------------- | :------------------------ | :------------------ | :--------------------------------------- |
| **HPA**                | Horizontal Pod Autoscaler | **ระดับ Pod**       | เพิ่ม/ลดจำนวน Pod ตามโหลด                |
| **VPA**                | Vertical Pod Autoscaler   | **ระดับ Container** | ปรับ Request/Limit ของ Pod ให้เหมาะสม    |
| **Cluster Autoscaler** | -                         | **ระดับ Node**      | เพิ่ม/ลด Node ของ Cluster ตามความต้องการ |

เรามาแยกอธิบายทีละตัว พร้อมภาพรวมการทำงานครับ 👇

---

## 🧩 **1️⃣ Horizontal Pod Autoscaler (HPA)**

**HPA** = “ขยาย *แนวนอน*” → เพิ่ม/ลดจำนวน **Pod**
ใช้เมื่อโหลดของ Service เพิ่มขึ้น เช่น CPU หรือ Memory ใช้สูงเกิน threshold

---

### 📄 ตัวอย่าง HPA

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hpa-test-app
  labels:
    app: hpa-test-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hpa-test-app
  template:
    metadata:
      labels:
        app: hpa-test-app
    spec:
      containers:
      - name: hpa-test-app
        image: hpa-test-app:latest  # Change this to your image
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 5000
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 256Mi
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-test-app
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: hpa-test-app
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0%
```

📌 ความหมาย:

* ใช้กับ Deployment ชื่อ `web-app`
* จะมี Pod ขั้นต่ำ 2 สูงสุด 10
* ถ้า CPU เฉลี่ย > 70% → HPA จะเพิ่ม Pod
* ถ้าโหลดลด → จะลด Pod ลงอัตโนมัติ

---

### 📈 การทำงานของ HPA

1. **Metrics Server** เก็บค่าการใช้ทรัพยากรของแต่ละ Pod
2. **HPA Controller** ตรวจค่า metric ทุก ๆ 15 วินาที (โดย default)
3. ถ้าเกิน target → คำนวณจำนวน Pod ที่เหมาะสม
4. ปรับ `replicas` ของ Deployment แบบอัตโนมัติ

---

### 🔧 Metric ที่ใช้ได้

* `cpu` / `memory` (จาก metrics-server)
* Custom metric (จาก Prometheus Adapter)
* External metric (เช่น queue length, API latency ฯลฯ)

---

### 🎯 ใช้ HPA เมื่อ:

* แอปต้องรองรับโหลดที่เปลี่ยนตลอดเวลา
* ใช้กับ stateless app เช่น API, frontend

---

## 🧩 **2️⃣ Vertical Pod Autoscaler (VPA)**

**VPA** = “ขยาย *แนวตั้ง*” → ปรับ resource (CPU/Memory) ของ **Pod** ให้เหมาะสมกับโหลด

ไม่เพิ่มจำนวน Pod แต่ “เพิ่มขนาดของแต่ละ Pod”
(เช่น จาก 500m → 1 CPU)

---

### 📄 ตัวอย่าง VPA

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: backend-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: backend
  updatePolicy:
    updateMode: "Auto"
```

📌 ความหมาย:

* ใช้กับ Deployment `backend`
* โหมด `Auto` = ปรับ request/limit ให้เองโดยอัตโนมัติ

---

### 🔍 Mode ของ VPA

| โหมด        | คำอธิบาย                        |
| :---------- | :------------------------------ |
| **Off**     | แค่แนะนำค่า ไม่ปรับจริง         |
| **Initial** | ใช้ค่าที่แนะนำเฉพาะตอนสร้าง Pod |
| **Auto**    | ปรับค่าให้ตลอดเวลา              |

---

### ⚠️ หมายเหตุ

* ถ้า VPA ปรับค่าขณะ Pod รันอยู่ → ต้อง recreate Pod (restart)
* ไม่ควรใช้ HPA และ VPA แบบ “ปรับ CPU พร้อมกัน” → จะชนกัน
  (แต่ใช้ HPA ตาม **custom metric** + VPA สำหรับ CPU ได้)

---

### 🎯 ใช้ VPA เมื่อ:

* workload มี pattern การใช้ resource ไม่แน่นอน
* ต้องการให้ระบบ optimize เอง
* เช่น backend, batch job, ML workload

---

## 🧩 **3️⃣ Cluster Autoscaler**

**Cluster Autoscaler (CA)** คือระบบที่ “เพิ่ม/ลดจำนวน Node” ของ cluster ตามความต้องการ

📦 หมายถึง ถ้า Pod ต้องการลง แต่ Node ที่มีอยู่ **ไม่พอ** → CA จะสร้าง Node ใหม่
หรือถ้า Node ไหนไม่มี workload เลย → CA จะ **ลบ Node** ออกเพื่อลดต้นทุน

---

### 📈 หลักการทำงาน

1. Scheduler พยายามวาง Pod แต่ไม่มี Node ไหนพอ (ตาม requests)
2. Cluster Autoscaler เห็นว่า “มี unscheduled Pod”
3. → สั่ง cloud provider (เช่น AWS, GCP, Azure) เพิ่ม Node ใหม่
4. เมื่อโหลดลด → ลบ Node ที่ idle ออก

---

### 📄 การใช้งาน (ผ่าน cloud provider)

**บน AWS EKS ตัวอย่าง:**

```bash
eksctl create nodegroup --cluster my-cluster \
  --name autoscale-nodes \
  --nodes 2 --nodes-min 2 --nodes-max 10
```

**บน GKE:**

```bash
gcloud container clusters update my-cluster \
  --enable-autoscaling --min-nodes 1 --max-nodes 10
```

---

### 🎯 ใช้ Cluster Autoscaler เมื่อ:

* Cluster ต้องรองรับ workload ที่เปลี่ยนแปลงบ่อย
* ต้องการ optimize cost (จ่ายเฉพาะช่วงมีโหลด)

---

## 🧭 **4️⃣ ภาพรวมทั้งหมด**

```
               ┌──────────────────────────────┐
               │       User / Workload        │
               └──────────────────────────────┘
                              │
             ┌────────────────┴────────────────┐
             │                                 │
     ┌───────────────┐                ┌────────────────┐
     │   HPA (Pods)  │                │   VPA (Pod Size)│
     └───────────────┘                └────────────────┘
             │                                 │
             └────────────────┬────────────────┘
                              ▼
               ┌──────────────────────────────┐
               │ Cluster Autoscaler (Nodes)   │
               └──────────────────────────────┘
```

**สรุปง่ายสุด:**

| ประเภท                 | ปรับอะไร           | ระดับ           | ตัวอย่าง               |
| :--------------------- | :----------------- | :-------------- | :--------------------- |
| **HPA**                | จำนวน Pod          | Workload level  | เพิ่มจาก 2 → 5 Pod     |
| **VPA**                | ขนาด Pod (CPU/Mem) | Container level | เพิ่ม CPU จาก 500m → 1 |
| **Cluster Autoscaler** | จำนวน Node         | Cluster level   | เพิ่ม Node จาก 3 → 5   |

---

## 💡 **Best Practice**

✅ ใช้ **HPA** สำหรับ stateless app เช่น web/api
✅ ใช้ **VPA** สำหรับ workload ที่ไม่แน่นอน เช่น batch job
✅ ใช้ **Cluster Autoscaler** เพื่อให้ระบบเพิ่ม Node อัตโนมัติเมื่อ cluster เต็ม
✅ อย่าใช้ HPA (CPU) + VPA พร้อมกันใน target เดียว

---

อยากให้ผมวาด **diagram แสดงการเชื่อมกันระหว่าง HPA → VPA → Cluster Autoscaler** ไหมครับ?
จะเห็นภาพการขยายระบบอัตโนมัติจาก Pod → Node → Cluster แบบครบวงจรเลย 🚀
