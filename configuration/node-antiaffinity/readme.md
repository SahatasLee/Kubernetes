# Node Anti-affinity

**Node Anti-Affinity** ใน Kubernetes เป็นกลไกที่ใช้เพื่อหลีกเลี่ยงการกำหนดตารางงาน (Scheduling) ของ Pods บน Nodes ที่มี Pods อื่นที่ตรงตามเงื่อนไขเดียวกันอยู่แล้ว ซึ่งช่วยป้องกันไม่ให้ Pods ถูกกำหนดให้รันบน Nodes เดียวกันและเพิ่มความยืดหยุ่นในการกระจายโหลดหรือเพื่อวัตถุประสงค์ทางด้านความพร้อมใช้งาน (High Availability).

## การทำงานของ Node Anti-Affinity

`Node Anti-Affinity` ใช้ในการกำหนดเงื่อนไขว่า Pods ไม่ควรจะรันบน Nodes ที่มี Pods ที่ตรงตามเงื่อนไข Label ที่ระบุอยู่แล้ว ซึ่งตรงข้ามกับ `Node Affinity` ที่กำหนดว่า Pods ควรรันบน Nodes ที่มี Label ที่ระบุ

### ตัวอย่างการใช้งาน Node Anti-Affinity

เพื่อใช้งาน `Node Anti-Affinity` คุณต้องกำหนดเงื่อนไขในไฟล์ YAML ของ Pod หรือ Deployment โดยใช้คำสั่ง `preferredDuringSchedulingIgnoredDuringExecution` หรือ `requiredDuringSchedulingIgnoredDuringExecution` สำหรับการกำหนดเงื่อนไขที่ต้องการหรือบังคับตามลำดับ

## ตัวอย่าง YAML ของ Node Anti-Affinity

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: app
                    operator: In
                    values:
                      - nginx
              topologyKey: "kubernetes.io/hostname"
      containers:
      - name: nginx
        image: nginx
```

ในตัวอย่างนี้:

- **`requiredDuringSchedulingIgnoredDuringExecution`**: ระบุว่า Pod ใหม่จะไม่ถูกกำหนดตารางงานบน Nodes ที่มี Pods ที่มี Label `app=nginx` อยู่แล้วโดยเด็ดขาด (เงื่อนไขบังคับ).
- **`labelSelector`**: กำหนดเงื่อนไขของ Labels ที่จะใช้ในการจับคู่กับ Pods ที่มีอยู่ใน Nodes.
- **`topologyKey`**: ใช้ในการกำหนดระดับที่ `Anti-Affinity` มีผล โดยทั่วไปจะใช้ `kubernetes.io/hostname` เพื่อบังคับใช้กฎในระดับ Node.

## ประเภทของ Node Anti-Affinity

1. **`requiredDuringSchedulingIgnoredDuringExecution`**: เป็นเงื่อนไขที่บังคับว่าต้องไม่มีกระจายงาน (Pods) ที่มี Label ตรงตามที่ระบุบน Node เดียวกัน Kubernetes จะไม่กำหนดตารางงานของ Pod ถ้าไม่สามารถหาตำแหน่งที่ตรงตามเงื่อนไขนี้ได้.
  
2. **`preferredDuringSchedulingIgnoredDuringExecution`**: เป็นเงื่อนไขที่ไม่บังคับ แต่ต้องการให้ Kubernetes พยายามหลีกเลี่ยงการกำหนดตารางงานของ Pod บน Node ที่มี Pods อื่นที่ตรงตามเงื่อนไขอยู่แล้ว หากไม่สามารถหาตำแหน่งที่ตรงตามเงื่อนไขนี้ได้ Kubernetes จะเลือกตำแหน่งที่ดีที่สุดที่มีอยู่แทน.

## ข้อดีของ Node Anti-Affinity

- **เพิ่มความพร้อมใช้งาน (High Availability):** ช่วยให้ Pods กระจายตัวไปยัง Nodes หลาย ๆ ตัว ลดความเสี่ยงจากการที่ Node เดียวล่ม.
- **เพิ่มความทนทานต่อข้อผิดพลาด (Fault Tolerance):** ลดโอกาสที่หลาย Pods ที่ทำงานร่วมกันจะถูกกำหนดตารางงานบน Node เดียวกัน.
- **ช่วยในการกระจายโหลด (Load Balancing):** ช่วยกระจายโหลดของ Pods ระหว่าง Nodes อย่างสมดุล.

## ข้อจำกัดของ Node Anti-Affinity

- **ขาดความยืดหยุ่นเมื่อมีทรัพยากรจำกัด:** ถ้าหากไม่มี Nodes ที่สามารถตรงตามเงื่อนไข `requiredDuringSchedulingIgnoredDuringExecution` Kubernetes จะไม่สามารถกำหนดตารางงานของ Pod ได้ ซึ่งอาจทำให้ Pods บางตัวไม่สามารถรันได้.
- **ซับซ้อนกว่า Node Affinity:** เนื่องจากมีหลายเงื่อนไขที่สามารถกำหนดได้ การตั้งค่าอาจจะซับซ้อนกว่าและต้องการการวางแผนล่วงหน้า.

## การเลือกใช้ Node Affinity กับ Node Anti-Affinity

- ใช้ **Node Affinity** เมื่อคุณต้องการให้ Pods ถูกกำหนดตารางงานเฉพาะบน Nodes ที่มี Label ที่ต้องการ.
- ใช้ **Node Anti-Affinity** เมื่อคุณต้องการหลีกเลี่ยงไม่ให้ Pods ถูกกำหนดตารางงานบน Nodes ที่มี Pods ที่มี Label ที่ระบุอยู่แล้ว.

## สรุป

- **Node Anti-Affinity** ใช้สำหรับกำหนดว่า Pods ไม่ควรถูกกำหนดตารางงานบน Nodes ที่มี Pods อื่น ๆ ที่มีเงื่อนไข Label เดียวกัน.
- สามารถช่วยในการเพิ่มความพร้อมใช้งาน, ความทนทานต่อข้อผิดพลาด และการกระจายโหลดในคลัสเตอร์.
- มีสองประเภท คือ `requiredDuringSchedulingIgnoredDuringExecution` (บังคับ) และ `preferredDuringSchedulingIgnoredDuringExecution` (ต้องการ).