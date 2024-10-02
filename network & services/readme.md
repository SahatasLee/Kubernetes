# Services

ใน Kubernetes, **Services** เป็นทรัพยากรที่ทำหน้าที่เป็นจุดเชื่อมต่อในการสื่อสารระหว่าง Pods ภายในคลัสเตอร์หรือกับทรัพยากรภายนอก โดย Services จะช่วยจัดการการรับส่งทราฟฟิกไปยัง Pods ที่เกี่ยวข้อง ซึ่งมักถูกกำหนดผ่าน **Labels** และ **Selectors** เพื่อติดตาม Pods ที่เป็นส่วนหนึ่งของบริการนั้น

### ประเภทของ Kubernetes Services:
มี 4 ประเภทหลัก ๆ ดังนี้:

#### 1. **ClusterIP** (ค่าเริ่มต้น):
- ใช้สำหรับการเข้าถึงภายในคลัสเตอร์เท่านั้น (ไม่สามารถเข้าถึงจากภายนอกคลัสเตอร์ได้)
- ClusterIP สร้าง IP ภายในคลัสเตอร์ที่สามารถใช้เพื่อสื่อสารระหว่าง Pods
- เหมาะสำหรับการเชื่อมต่อระหว่างบริการภายในคลัสเตอร์ เช่น ฐานข้อมูล

  **ตัวอย่าง**:
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: my-clusterip-service
  spec:
    selector:
      app: my-app
    ports:
      - protocol: TCP
        port: 80       # พอร์ตที่บริการนี้จะใช้
        targetPort: 8080 # พอร์ตของ Pod ที่จะรับทราฟฟิก
  ```

#### 2. **NodePort**:
- เปิดให้สามารถเข้าถึงจากภายนอกคลัสเตอร์ได้ โดยจะเปิดพอร์ตบนทุก Node ของคลัสเตอร์เพื่อให้ทราฟฟิกสามารถเข้าสู่คลัสเตอร์ได้
- ใช้เมื่อคุณต้องการให้บริการในคลัสเตอร์ของคุณสามารถเข้าถึงได้จากภายนอกโดยตรง แต่ต้องกำหนดพอร์ตที่อยู่ในช่วง 30000-32767

  **ตัวอย่าง**:
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: my-nodeport-service
  spec:
    type: NodePort
    selector:
      app: my-app
    ports:
      - protocol: TCP
        port: 80
        targetPort: 8080
        nodePort: 30007  # พอร์ตบน Node ที่จะเปิดรับทราฟฟิกจากภายนอก
  ```

#### 3. **LoadBalancer**:
- ใช้ในการเปิดให้บริการเข้าถึงได้จากภายนอกผ่าน Load Balancer ของคลาวด์ผู้ให้บริการ (เช่น AWS, GCP, Azure)
- LoadBalancer จะกำหนด IP ภายนอกให้ และกระจายทราฟฟิกไปยัง Pods ที่อยู่ภายใต้ Service นั้น
- เหมาะสำหรับบริการที่ต้องการให้มีการกระจายโหลดทราฟฟิกจากผู้ใช้ภายนอกคลัสเตอร์

  **ตัวอย่าง**:
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: my-loadbalancer-service
  spec:
    type: LoadBalancer
    selector:
      app: my-app
    ports:
      - protocol: TCP
        port: 80
        targetPort: 8080
  ```

#### 4. **ExternalName**:
- ใช้สำหรับการแม็ปชื่อ DNS ภายนอกไปยังบริการภายในคลัสเตอร์
- ExternalName จะไม่สร้าง ClusterIP แต่จะสร้าง DNS ชื่อหนึ่งที่ชี้ไปยังชื่อ DNS ที่กำหนดใน `externalName`
- ใช้เมื่อบริการต้องการติดต่อกับทรัพยากรภายนอกที่ไม่มีในคลัสเตอร์ Kubernetes

  **ตัวอย่าง**:
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: my-externalname-service
  spec:
    type: ExternalName
    externalName: example.com  # DNS ที่จะชี้ไปยังทรัพยากรภายนอก
  ```

### การทำงานของ Service:
- **Selector**: Services จะใช้ **Selectors** เพื่อติดตาม Pods ที่ต้องการให้ทราฟฟิกส่งไปหา โดยดูจาก **Labels** ที่ถูกกำหนดใน Pods
- **Endpoints**: Kubernetes จะคอยดูแลและอัปเดต **Endpoints** ของ Pods ที่ตรงกับ Selector ใน Service เมื่อมีการเพิ่มหรือลบ Pods

### การกำหนด Service:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app  # Pods ที่มี label app=my-app จะถูกเลือก
  ports:
    - protocol: TCP
      port: 80  # พอร์ตของ Service ที่จะใช้ภายนอก
      targetPort: 8080  # พอร์ตของ Pod ที่จะรับทราฟฟิก
```

### การเชื่อมต่อกับ Service:
- Service จะได้รับชื่อ DNS ภายในคลัสเตอร์จาก Kubernetes ทำให้ Pods หรือทรัพยากรอื่น ๆ สามารถเชื่อมต่อกับ Service โดยใช้ชื่อ DNS นั้น เช่น `my-service.default.svc.cluster.local`

### ตัวอย่างการใช้งานจริง:
หากคุณมีการ Deploy แอปพลิเคชันหลาย Pod และต้องการให้ผู้ใช้ภายนอกคลัสเตอร์สามารถเข้าถึงแอปพลิเคชันนั้นได้ คุณสามารถใช้ `LoadBalancer` หรือ `NodePort` เพื่อเปิดให้เข้าถึงจากภายนอกคลัสเตอร์

### ข้อดีของการใช้ Kubernetes Services:
- ช่วยให้ทราฟฟิกถูกกระจายไปยัง Pods หลายตัว (Load Balancing)
- ทำให้ระบบสามารถปรับขยาย (scaling) ได้ง่าย
- ช่วยจัดการการเชื่อมต่อระหว่าง Pods ภายในคลัสเตอร์
- ง่ายต่อการรวม Kubernetes เข้ากับคลาวด์แพลตฟอร์มต่าง ๆ