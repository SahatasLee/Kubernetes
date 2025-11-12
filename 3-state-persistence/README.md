# Kubernetes Volumes

## 🧱 **1. Kubernetes Volume**

* **คือ:** กลไกสำหรับ **แนบพื้นที่เก็บข้อมูล (storage)** ให้กับ container ภายใน Pod
* **ทำไมต้องมี Volume:** เพราะ container ปกติข้อมูลจะหายเมื่อ container ถูกลบ — Volume ทำให้ข้อมูล **อยู่รอดแม้ container ตาย**
* **ชนิดของ Volume มีหลายแบบ:**

  * `emptyDir` → เก็บชั่วคราว (หายเมื่อ pod หาย)
  * `hostPath` → ใช้พื้นที่จาก node ที่รัน pod
  * `nfs`, `openebs`, `awsElasticBlockStore`, `gcePersistentDisk`, `csi` ฯลฯ

### 📦 ตัวอย่าง `emptyDir`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
  - name: app
    image: busybox
    volumeMounts:
    - mountPath: /data
      name: temp-storage
  volumes:
  - name: temp-storage
    emptyDir: {}
```

* สร้างโฟลเดอร์ `/data` ใน pod
* หายหมดเมื่อ pod ถูกลบ

📍 **สรุป:**
`Volume` อยู่ในระดับ **Pod** — เก็บข้อมูลให้ container ใน pod ใช้ร่วมกันได้

## 💾 **2. PersistentVolume (PV)**

* **คือ:** Resource ระดับ cluster ที่แทน “พื้นที่จัดเก็บจริง” (storage backend)
* **ทำหน้าที่เหมือน:** “ดิสก์ถาวร” ที่เตรียมไว้ในระบบ (อาจมาจาก NFS, Ceph, EBS, etc.)
* **ถูกสร้างโดย admin (หรือโดย StorageClass แบบ dynamic)**

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /data/pv1
    server: 10.10.10.20
```

📍 **สรุป:**
PV = **พื้นที่จริง** ที่มีอยู่ในระบบ (เช่น 10Gi จาก NFS หรือ disk บน cloud)

## 📥 **3. PersistentVolumeClaim (PVC)**

* **คือ:** “คำขอใช้พื้นที่” จากผู้ใช้ (developer)
* Developer ไม่ต้องรู้ว่า PV อยู่ที่ไหน — แค่บอกว่าต้องการ storage เท่าไร
* Kubernetes จะจับคู่ PVC → PV ที่เหมาะสมโดยอัตโนมัติ

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

📍 **สรุป:**
PVC = **คำขอพื้นที่**
PV = **พื้นที่จริงที่จัดเตรียมไว้**

เมื่อจับคู่สำเร็จ → PVC จะ "bind" กับ PV หนึ่งตัว

## ⚙️ **4. StorageClass**

* **คือ:** ตัวกลางที่ใช้ “จัดการการสร้าง PV แบบอัตโนมัติ (Dynamic Provisioning)”
* แทนที่จะสร้าง PV เอง → Kubernetes จะสร้างให้ตามที่ StorageClass กำหนด
* ใช้ได้ดีมากใน cloud เช่น AWS EBS, GCE PD, Ceph RBD, CSI plugin ฯลฯ

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  fsType: ext4
reclaimPolicy: Delete
```

เมื่อ PVC ระบุ `storageClassName: fast-ssd` →
Kubernetes จะสร้าง PV ให้โดยอัตโนมัติ

## 🔄 **ความสัมพันธ์ของทั้ง 4 ตัว**

```
+-------------------------------+
|         Application (Pod)     |
|   └── ใช้ PVC เพื่อ mount data |
+-------------------------------+
              │
              ▼
+-------------------------------+
|  PVC (PersistentVolumeClaim)  |  <-- Developer ขอพื้นที่
+-------------------------------+
              │  bind
              ▼
+-------------------------------+
|  PV (PersistentVolume)        |  <-- Admin เตรียม storage จริง
+-------------------------------+
              │
              ▼
+-------------------------------+
|  Physical Storage (NFS, EBS,  |
|  Ceph, Local Disk, etc.)      |
+-------------------------------+
```

หรือในกรณีใช้ **StorageClass:**

```
Pod → PVC ─┬─> StorageClass ──> สร้าง PV ให้อัตโนมัติ
            └──────────────────> เชื่อมต่อ storage backend
```

## 📘 ตัวอย่างรวมทั้งหมด (Dynamic Provisioning)

```yaml
# 1. StorageClass
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3

---
# 2. PersistentVolumeClaim
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast
  resources:
    requests:
      storage: 5Gi

---
# 3. Pod ใช้ PVC
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
  - name: myapp
    image: nginx
    volumeMounts:
    - mountPath: /data
      name: mydata
  volumes:
  - name: mydata
    persistentVolumeClaim:
      claimName: mypvc
```

**Flow:**

1. Pod ขอใช้ PVC (`mypvc`)
2. PVC ใช้ StorageClass `fast` → Kubernetes สร้าง PV ให้อัตโนมัติ
3. PV bind กับ PVC
4. Pod mount `/data` จาก PV
5. ข้อมูลใน `/data` ไม่หายแม้ pod ตาย

## 🧭 **สรุปภาพรวม**

| Component                       | ระดับ     | ทำหน้าที่                      | ใครสร้าง        |
| :------------------------------ | :-------- | :----------------------------- | :-------------- |
| **Volume**                      | Pod       | พื้นที่เก็บข้อมูลภายใน pod     | Dev             |
| **PersistentVolume (PV)**       | Cluster   | แหล่ง storage จริง             | Admin / Dynamic |
| **PersistentVolumeClaim (PVC)** | Namespace | คำขอใช้ storage                | Dev             |
| **StorageClass**                | Cluster   | ตัวกำหนดวิธีสร้าง PV อัตโนมัติ | Admin           |

---

อยากไหมครับให้ผมวาด **diagram flow** ของ Storage ทั้ง 4 ตัว (Pod → PVC → PV → StorageClass → Storage Backend) แบบภาพให้เห็นการเชื่อมต่อทั้งหมด?
จะช่วยให้เข้าใจภาพรวมเร็วขึ้นมาก 🧩
