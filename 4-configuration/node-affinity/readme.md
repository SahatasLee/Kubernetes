# Node Affinity

**Node Affinity** ใน Kubernetes คือกลไกที่ใช้ในการกำหนดข้อจำกัดหรือความต้องการของ Pods ว่าจะถูกกำหนดให้ทำงานบน Nodes ที่มีคุณสมบัติตรงตามที่ระบุไว้หรือไม่ โดยเป็นส่วนหนึ่งของคุณสมบัติการกำหนดตารางงาน (Scheduling) ซึ่งจะช่วยให้เราสามารถควบคุมว่า Pods ควรจะทำงานบน Nodes ใดตามเงื่อนไขที่เรากำหนดไว้

## การทำงานของ Node Affinity

**Node Affinity** ทำงานในลักษณะคล้ายกับ **Node Selector** ซึ่งใช้เพื่อกำหนดให้ Pods ทำงานบน Nodes ที่มี Labels ตรงตามที่กำหนด แต่ Node Affinity มีความยืดหยุ่นมากกว่า เนื่องจากสามารถระบุเงื่อนไขที่ซับซ้อนมากขึ้น เช่น เงื่อนไขแบบ "ต้องการ" (required) หรือ "ต้องการแต่ไม่บังคับ" (preferred)

## ประเภทของ Node Affinity

1. **requiredDuringSchedulingIgnoredDuringExecution**:
   - เป็นข้อกำหนดที่จำเป็น ต้องตรงตามเงื่อนไขที่กำหนดไว้ก่อนที่ Pod จะถูกวางลงบน Node.
   - ถ้าไม่มี Nodes ใดที่ตรงตามเงื่อนไขนี้ Pod จะไม่ถูกกำหนดตารางงาน (Scheduled).

2. **preferredDuringSchedulingIgnoredDuringExecution**:
   - เป็นข้อกำหนดที่ต้องการแต่ไม่บังคับ Kubernetes จะพยายามกำหนดตารางงานของ Pod บน Node ที่ตรงตามเงื่อนไขนี้ แต่ถ้าไม่มี Node ใดที่ตรงตามเงื่อนไขนี้ Kubernetes จะยังคงกำหนดตารางงานให้กับ Node อื่นที่มีความเหมาะสมที่สุดเท่าที่จะเป็นไปได้.

## ตัวอย่างการใช้ Node Affinity

ตัวอย่างต่อไปนี้เป็นไฟล์ YAML ที่แสดงการใช้ **Node Affinity** เพื่อกำหนดว่า Pods จะถูกกำหนดให้ทำงานเฉพาะบน Nodes ที่มี Label `disktype=ssd`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:  # ต้องการให้ตรงกับเงื่อนไขนี้เท่านั้น
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
  containers:
  - name: nginx
    image: nginx
```

### การทำงานของตัวอย่างข้างต้น
- **requiredDuringSchedulingIgnoredDuringExecution**: Pod จะถูกกำหนดให้ทำงานเฉพาะบน Node ที่มี Label `disktype` ที่มีค่า `ssd` เท่านั้น.
- **nodeSelectorTerms** และ **matchExpressions**: ใช้เพื่อระบุเงื่อนไขของ Label ที่ต้องการบน Node โดยใช้คีย์ (`key`), ตัวดำเนินการ (`operator`), และค่าที่ต้องการ (`values`).

### การใช้งานอื่น ๆ ของ Node Affinity

- คุณสามารถใช้ **Node Affinity** เพื่อกำหนดให้ Pods ทำงานบน Node ที่มีคุณสมบัติเฉพาะ เช่น มีทรัพยากรเฉพาะ, อยู่ในโซนหรือภูมิภาคเฉพาะ, หรือมีค่า Label ที่ต้องการ.
- ช่วยในการกระจายการโหลดของ Pods ไปยัง Nodes ที่เหมาะสมเพื่อให้สามารถใช้งานทรัพยากรได้อย่างมีประสิทธิภาพมากขึ้น.

Node Affinity เป็นวิธีที่มีความยืดหยุ่นในการกำหนดข้อกำหนดในการกำหนดตารางงาน Pods ใน Kubernetes ซึ่งช่วยให้คุณสามารถควบคุมตำแหน่งของ Pods ได้มากขึ้นกว่าการใช้ Node Selector แบบปกติ.

## Type เพิ่มเติม

ใน Kubernetes, `Node Affinity` ถูกใช้เพื่อกำหนดความต้องการของ Pods สำหรับการกำหนดตารางงาน (Scheduling) ไปยัง Nodes ที่เหมาะสม โดยมีเงื่อนไขที่สามารถตั้งค่าได้หลากหลาย ซึ่งแยกออกเป็นประเภทต่าง ๆ ดังนี้:

### Types of Node Affinity

1. **Available Node Affinity Types**
   - มีสองประเภทหลักที่ใช้งานได้:
   - **`requiredDuringSchedulingIgnoredDuringExecution`**: กำหนดให้ Pod ถูกกำหนดตารางงาน (scheduled) เฉพาะบน Nodes ที่ตรงตามเงื่อนไขเท่านั้น แต่ถ้าหากเงื่อนไขของ Node เปลี่ยนไปหลังจากที่ Pod ถูกกำหนดตารางงานแล้ว (เช่น Label ของ Node ถูกแก้ไข) Kubernetes จะไม่บังคับให้ Pod ถูกย้ายออกหรือทำการเปลี่ยนแปลงใดๆ (Ignored During Execution).
   - **`preferredDuringSchedulingIgnoredDuringExecution`**: เป็นการกำหนดเงื่อนไขที่ต้องการแต่ไม่บังคับให้ตรงตามเสมอ Kubernetes จะพยายามกำหนดตารางงานของ Pod บน Nodes ที่ตรงตามเงื่อนไข แต่ถ้าไม่พบ Node ที่ตรงตามเงื่อนไขนี้ ก็จะยังคงกำหนดตารางงานของ Pod บน Node ที่เหมาะสมที่สุดที่มีอยู่ได้ (Preferred During Scheduling).

2. **During Scheduling**
   - เป็นเงื่อนไขที่ถูกนำมาพิจารณาในขั้นตอนการกำหนดตารางงาน (Scheduling) ของ Pods เท่านั้น:
   - **`requiredDuringScheduling`**: กำหนดเงื่อนไขที่ Node ต้องตรงตามก่อนที่ Pod จะถูกกำหนดตารางงาน (Scheduled).
   - **`preferredDuringScheduling`**: กำหนดเงื่อนไขที่ Node ควรจะตรงตามเมื่อกำหนดตารางงาน แต่ไม่ได้บังคับ (Preferred).

3. **During Execution**
   - เป็นเงื่อนไขที่เกี่ยวข้องกับการดำเนินการของ Pods เมื่อ Pod ได้ถูกกำหนดตารางงานแล้ว:
   - **`IgnoredDuringExecution`**: Kubernetes จะไม่บังคับตรวจสอบหรือบังคับเงื่อนไขเหล่านี้ระหว่างที่ Pod กำลังทำงานอยู่ (Execution).
   - **`RequiredDuringExecution`**: (ไม่ใช่คุณสมบัติที่ใช้งานได้ในปัจจุบัน) หมายถึงเงื่อนไขที่ Kubernetes จะบังคับให้ตรงตามระหว่างการทำงานของ Pods.

4. **Planned Node Affinity Type (ไม่ได้ใช้งานในปัจจุบัน)**
   - **`requiredDuringSchedulingRequiredDuringExecution`**: 
     - เป็นประเภทที่ยังไม่ได้ถูกนำมาใช้จริงใน Kubernetes เวอร์ชันปัจจุบัน แต่มันหมายถึงเงื่อนไขที่ Node ต้องตรงตามทั้งในขั้นตอนการกำหนดตารางงาน (`Scheduling`) และระหว่างการทำงาน (`Execution`). ถ้า Node ที่ Pod ทำงานอยู่ไม่ตรงตามเงื่อนไขนี้ในภายหลัง Kubernetes อาจย้ายหรือหยุด Pod นั้นทันที (ตัวอย่างเช่นเพื่อให้แน่ใจว่า Pod จะรันเฉพาะบน Node ที่มีคุณสมบัติพิเศษหรือความปลอดภัย).

### ความหมายและการใช้งาน
- **`requiredDuringSchedulingIgnoredDuringExecution`**: เหมาะสำหรับใช้เมื่อคุณต้องการให้ Pods ถูกกำหนดตารางงานเฉพาะบน Nodes ที่ตรงตามเงื่อนไขอย่างเข้มงวดในขั้นตอนการ Scheduling แต่ไม่จำเป็นต้องตรวจสอบเงื่อนไขนี้ในภายหลังเมื่อ Pod เริ่มทำงานแล้ว.
  
- **`preferredDuringSchedulingIgnoredDuringExecution`**: ใช้เมื่อคุณต้องการให้ Pods ถูกกำหนดตารางงานบน Nodes ที่ตรงตามเงื่อนไขเป็นการที่ต้องการ แต่ไม่บังคับ และ Kubernetes สามารถเลือก Node ที่เหมาะสมที่สุดที่พร้อมใช้งานได้.

### สรุป
`Node Affinity` ช่วยให้คุณสามารถกำหนดข้อกำหนดที่ยืดหยุ่นสำหรับการกำหนดตารางงาน Pods ใน Kubernetes โดยมีเงื่อนไขที่หลากหลาย ทั้งที่เป็นการบังคับและไม่บังคับ ขึ้นอยู่กับความต้องการในการกำหนดตารางงานและการดำเนินการของคุณ.