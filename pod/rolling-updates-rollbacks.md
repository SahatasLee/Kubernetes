# Rolling Update & Rollback

**Rolling Update** และ **Rollback** ใน Kubernetes เป็นส่วนหนึ่งของการจัดการ `Deployment` ที่ช่วยในการอัปเดตแอปพลิเคชันหรือเวอร์ชันของคอนเทนเนอร์อย่างราบรื่นโดยไม่กระทบต่อการใช้งานของผู้ใช้:

### 1. **Rolling Update**:
- **Rolling Update** คือการอัปเดตแอปพลิเคชันแบบต่อเนื่อง โดยจะค่อย ๆ อัปเดต Pods ทีละชุด แทนที่จะปิดทุก Pod แล้วค่อยเปิดใหม่หมด (แบบ Recreate)
- วิธีนี้ช่วยให้แอปพลิเคชันยังคงทำงานได้ระหว่างการอัปเดต โดยมีบางส่วนที่ยังคงให้บริการในขณะที่ Pods ชุดใหม่กำลังถูกสร้างและเริ่มทำงาน

#### ขั้นตอนการทำงานของ Rolling Update:
- Kubernetes จะสร้าง Pods ใหม่ (ตามที่มีการเปลี่ยนแปลง เช่นเวอร์ชันของคอนเทนเนอร์)
- จากนั้นจะค่อย ๆ ลบ Pods เก่าทิ้งเมื่อ Pods ใหม่พร้อมทำงาน
- จำนวน Pods ที่จะอัปเดตหรือลบในแต่ละครั้งสามารถกำหนดได้ผ่านฟิลด์ `maxSurge` และ `maxUnavailable`

#### ตัวอย่างการกำหนด Rolling Update:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1              # จำนวน Pods ใหม่ที่เพิ่มขึ้นในแต่ละครั้ง (default: 25%)
      maxUnavailable: 1        # จำนวน Pods เก่าที่สามารถหยุดทำงานได้ในขณะเดียวกัน (default: 25%)
  template:
    spec:
      containers:
      - name: frontend
        image: nginx:1.19.3     # เวอร์ชันใหม่ของ image ที่จะอัปเดต
```

### 2. **Rollback**:
- **Rollback** คือการย้อนกลับ `Deployment` ไปยังเวอร์ชันก่อนหน้า หากเกิดปัญหาหลังจากการอัปเดต
- Kubernetes เก็บบันทึกประวัติการอัปเดต (revisions) ของ Deployment ไว้ ทำให้สามารถย้อนกลับไปยังการปรับใช้เวอร์ชันก่อนหน้าได้อย่างรวดเร็ว

#### ขั้นตอนการ Rollback:
- Kubernetes จะคืนค่าการตั้งค่า Deployment ไปยัง revision ก่อนหน้า โดยจะสร้าง Pods ใหม่ตามการตั้งค่าที่เคยใช้งานใน revision นั้น

#### การ Rollback ด้วยคำสั่ง `kubectl`:
- คำสั่งด้านล่างจะทำให้ Deployment ย้อนกลับไปยัง revision ล่าสุดที่ไม่มีปัญหา:
  ```bash
  kubectl rollout undo deployment frontend-deployment
  ```

- หากต้องการย้อนกลับไปยัง revision เฉพาะ (เช่น revision 2):
  ```bash
  kubectl rollout undo deployment frontend-deployment --to-revision=2
  ```

### การตรวจสอบสถานะการอัปเดตและ Rollback:
- ดูสถานะของการอัปเดต:
  ```bash
  kubectl rollout status deployment frontend-deployment
  ```

- ดูประวัติ revision:
  ```bash
  kubectl rollout history deployment frontend-deployment
  ```

### ข้อดีของ Rolling Update:
- ลด downtime ของแอปพลิเคชัน เพราะยังมี Pods บางส่วนทำงานขณะที่อัปเดต
- ลดความเสี่ยงที่จะเกิดปัญหาจากการอัปเดตโดยสามารถควบคุมการสร้าง Pods ใหม่และลบ Pods เก่าได้

### ข้อดีของ Rollback:
- ลดเวลาในการแก้ไขปัญหาจากการอัปเดตที่ผิดพลาด
- Kubernetes สามารถจัดการย้อนกลับได้อัตโนมัติหากเจอข้อผิดพลาดจากการอัปเดต