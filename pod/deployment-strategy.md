# Deployment Strategy

ใน Kubernetes, **Deployment Strategy** หมายถึงวิธีที่ Kubernetes ใช้ในการจัดการการอัปเดตหรือปรับเปลี่ยนเวอร์ชันของแอปพลิเคชันที่รันอยู่ใน Pods โดยใช้การทำงานของ **Deployment** โดยเฉพาะเมื่อมีการเปลี่ยนแปลงโครงสร้าง (เช่น อัปเดต image ใหม่ หรือเปลี่ยนแปลงการตั้งค่า)

### Deployment Strategies หลัก ๆ ใน Kubernetes:
1. **Recreate Strategy**
2. **Rolling Update Strategy** (ค่าเริ่มต้น)

### 1. **Recreate Strategy**:
- วิธีนี้คือการหยุดและลบ Pods เดิมทั้งหมดก่อนที่จะสร้าง Pods ใหม่ขึ้นมา
- ไม่มีการทำงานพร้อมกันระหว่าง Pods เก่าและ Pods ใหม่ เมื่อมีการอัปเดตจะเกิด downtime (ช่วงเวลาที่แอปพลิเคชันไม่สามารถให้บริการได้)

#### การใช้งาน Recreate Strategy:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 3
  strategy:
    type: Recreate  # ใช้ strategy Recreate
  template:
    spec:
      containers:
      - name: my-app
        image: my-app:2.0  # เวอร์ชันใหม่ของคอนเทนเนอร์
```

**กรณีที่ควรใช้ Recreate Strategy**:
- เหมาะกับแอปพลิเคชันที่ไม่สามารถรองรับการรันพร้อมกันของหลายเวอร์ชันได้
- แอปพลิเคชันที่ไม่มีการแชร์ state ระหว่าง Pods หรือที่มีการจัดการ state ภายนอก (stateless)

### 2. **Rolling Update Strategy** (ค่าเริ่มต้น):
- วิธีนี้คือการค่อย ๆ อัปเดต Pods ทีละส่วน โดยจะสร้าง Pods ใหม่ก่อนที่จะลบ Pods เก่า
- ทำให้เกิดการเปลี่ยนแปลงอย่างราบรื่น โดยที่แอปพลิเคชันยังคงสามารถทำงานได้ในระหว่างที่กำลังอัปเดตอยู่ (no downtime)
- จำนวน Pods ที่ถูกสร้างหรือลบในแต่ละครั้งสามารถควบคุมได้ผ่านฟิลด์ `maxUnavailable` และ `maxSurge`

#### การใช้งาน Rolling Update Strategy:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 3
  strategy:
    type: RollingUpdate  # ใช้ strategy RollingUpdate (ค่าเริ่มต้น)
    rollingUpdate:
      maxSurge: 1        # จำนวน Pods ใหม่ที่เพิ่มขึ้นในขณะที่ทำการอัปเดต (ค่าเริ่มต้น 25%)
      maxUnavailable: 1  # จำนวน Pods ที่สามารถหยุดทำงานได้ในขณะที่ทำการอัปเดต (ค่าเริ่มต้น 25%)
  template:
    spec:
      containers:
      - name: my-app
        image: my-app:2.0  # เวอร์ชันใหม่ของคอนเทนเนอร์
```

**ฟิลด์ที่สำคัญใน Rolling Update**:
- **maxSurge**: จำนวน Pods ใหม่ที่สามารถสร้างขึ้นได้เกินจากจำนวนที่กำหนดใน `replicas` เพื่อให้รองรับทราฟฟิกขณะที่ Pods ใหม่ถูกสร้างขึ้น (เช่น `replicas: 3` และ `maxSurge: 1` หมายความว่า Kubernetes สามารถสร้าง 4 Pods ชั่วคราวได้)
- **maxUnavailable**: จำนวน Pods ที่สามารถหยุดทำงานได้ในขณะอัปเดต (หากกำหนดค่า `maxUnavailable: 1` หมายความว่า 1 Pod สามารถหยุดทำงานได้ขณะที่ทำการอัปเดตโดยยังมี Pod อื่นที่ให้บริการอยู่)

**กรณีที่ควรใช้ Rolling Update Strategy**:
- เหมาะกับแอปพลิเคชันที่ต้องการให้มีความพร้อมใช้งานตลอดเวลา (minimal downtime)
- แอปพลิเคชันที่สามารถรันหลายเวอร์ชันพร้อมกันได้ หรือมีการจัดการ state ภายนอก

### การเลือก Deployment Strategy:
- **Recreate**: เหมาะสำหรับแอปพลิเคชันที่ไม่สามารถรันหลายเวอร์ชันพร้อมกัน หรือไม่ต้องการให้มี Pods ใหม่และเก่ารันพร้อมกัน เช่น Stateful Applications ที่มีการจัดการ state ภายใน Pods
- **Rolling Update**: เหมาะสำหรับแอปพลิเคชันที่ต้องการลด downtime ให้น้อยที่สุด และสามารถรองรับการรันเวอร์ชันใหม่และเก่าพร้อมกันได้ หรือแอปพลิเคชันที่มีการทำงานแบบ stateless

### การตรวจสอบสถานะการอัปเดต Deployment:
สามารถใช้คำสั่ง `kubectl` เพื่อดูสถานะของการอัปเดต Deployment ได้ดังนี้:
- ตรวจสอบสถานะการอัปเดต:
  ```bash
  kubectl rollout status deployment my-deployment
  ```

- ตรวจสอบประวัติการปรับใช้:
  ```bash
  kubectl rollout history deployment my-deployment
  ```

### การจัดการ Rollback:
หากมีปัญหาหลังจากการอัปเดต คุณสามารถใช้คำสั่ง Rollback เพื่อย้อนกลับไปยังเวอร์ชันก่อนหน้าได้:
```bash
kubectl rollout undo deployment my-deployment
```

### Summary:
- **Recreate**: หยุด Pods เก่าทั้งหมดก่อนสร้าง Pods ใหม่ (เกิด downtime)
- **Rolling Update**: ค่อย ๆ อัปเดต Pods ทีละส่วน (ไม่เกิด downtime)