# Deployment

ใน Kubernetes, **Deployment** เป็นทรัพยากรที่ใช้สำหรับการจัดการและควบคุมการปรับใช้ (deployment) ของแอปพลิเคชัน ซึ่ง Deployment ทำให้คุณสามารถอัปเดตและจัดการ Pods ได้อย่างอัตโนมัติ รวมถึงรองรับการสเกล (scale) แอปพลิเคชันได้ง่ายดาย

### คุณสมบัติหลักของ Kubernetes Deployment:
1. **อัปเดตและ Rollback** แอปพลิเคชัน: คุณสามารถทำการอัปเดตหรือ Rollback แอปพลิเคชันได้อย่างราบรื่นโดยไม่มี downtime
2. **สเกลแอปพลิเคชัน**: คุณสามารถสเกลจำนวน Pods ขึ้นหรือลงได้ตามความต้องการ
3. **ดูแลสถานะของ Pods**: Deployment จะช่วยตรวจสอบและดูแลให้ Pods ทำงานตามจำนวนที่กำหนดไว้เสมอ หากมี Pod ล้มเหลว Kubernetes จะพยายามสร้างใหม่ให้โดยอัตโนมัติ
4. **การตรวจสอบสถานะ**: Deployment ช่วยตรวจสอบสถานะของ Pods และการอัปเดตผ่านการทำงานของ Probes เช่น Readiness และ Liveness Probes

### โครงสร้างของ Deployment:
Deployment มักจะระบุรายละเอียดการทำงาน เช่น จำนวนของ Pods ที่ต้องการ, กลยุทธ์ในการอัปเดต, และภาพ (image) ของคอนเทนเนอร์ที่ใช้งาน

### ตัวอย่างไฟล์ YAML ของ Deployment:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
  labels:
    app: my-app
spec:
  replicas: 3  # จำนวน Pods ที่ต้องการ
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: my-app:1.0  # Image ของแอปพลิเคชันที่ต้องการใช้
        ports:
        - containerPort: 80  # พอร์ตที่คอนเทนเนอร์จะเปิดให้บริการ
```

### ฟิลด์สำคัญใน Deployment:
- **replicas**: จำนวนของ Pods ที่ต้องการสร้าง
- **selector**: ใช้สำหรับเลือก Pods ที่ Deployment จะจัดการ โดยระบุ `matchLabels` เพื่อจับคู่กับ Pods ที่มี Label ตรงกัน
- **template**: เป็นโครงสร้างที่ระบุลักษณะของ Pods ที่จะถูกสร้างขึ้น โดยประกอบด้วย metadata (เช่น labels) และ spec (การตั้งค่า containers ที่จะใช้งาน)
- **strategy**: กำหนดกลยุทธ์การอัปเดต Pods เช่น `RollingUpdate` หรือ `Recreate`

### การจัดการ Deployment:
1. **สร้าง Deployment**:
   ใช้คำสั่ง `kubectl` เพื่อนำไฟล์ Deployment ขึ้นสู่คลัสเตอร์:
   ```bash
   kubectl apply -f deployment.yaml
   ```

2. **ตรวจสอบสถานะของ Deployment**:
   สามารถตรวจสอบสถานะของ Deployment และ Pods ที่สร้างจาก Deployment นั้นได้:
   ```bash
   kubectl get deployments
   kubectl get pods
   ```

3. **อัปเดต Deployment**:
   เมื่อคุณทำการแก้ไข Deployment เช่น การเปลี่ยนแปลงเวอร์ชันของ image, Kubernetes จะใช้ **Rolling Update Strategy** เพื่อทำการอัปเดต Pods ทีละส่วน
   ```bash
   kubectl set image deployment/my-deployment my-container=my-app:2.0
   ```

4. **Rollback Deployment**:
   หากมีปัญหาเกิดขึ้นหลังจากการอัปเดต คุณสามารถ Rollback ไปยังเวอร์ชันก่อนหน้าได้:
   ```bash
   kubectl rollout undo deployment/my-deployment
   ```

5. **สเกล Deployment**:
   การสเกลจำนวน Pods สามารถทำได้ง่าย ๆ ด้วยคำสั่ง:
   ```bash
   kubectl scale deployment my-deployment --replicas=5
   ```

6. **ตรวจสอบประวัติการอัปเดต**:
   Kubernetes เก็บประวัติการอัปเดตของ Deployment ไว้เพื่อช่วยในการจัดการการ Rollback:
   ```bash
   kubectl rollout history deployment my-deployment
   ```

### Deployment Strategies:
Kubernetes มีสองกลยุทธ์หลักในการจัดการการอัปเดตแอปพลิเคชัน:
1. **Rolling Update** (ค่าเริ่มต้น): ทำการอัปเดต Pods ทีละส่วน (ไม่มี downtime)
2. **Recreate**: ลบ Pods เดิมทั้งหมดก่อนที่จะสร้าง Pods ใหม่ (อาจมี downtime)

### สรุป:
**Deployment** เป็นวิธีที่ดีที่สุดในการจัดการการปรับใช้แอปพลิเคชันใน Kubernetes เพราะสามารถอัปเดต, ดูแลสถานะของ Pods, และทำการสเกลแอปพลิเคชันได้อย่างมีประสิทธิภาพ