# Namespace

**Namespace** ใน Kubernetes คือวิธีการแบ่งแยกหรือจัดการทรัพยากร (resources) ในคลัสเตอร์ Kubernetes ออกเป็นกลุ่ม ๆ เพื่อให้สามารถแยกการทำงานและการจัดการแอปพลิเคชันได้เป็นระเบียบมากขึ้น

### ความสำคัญของ Namespace:
1. **การแยกทรัพยากร**: Namespace ช่วยให้คุณสามารถแยกทรัพยากรออกจากกันได้อย่างชัดเจน เช่น Pod, Service, Deployment ฯลฯ ทำให้สามารถจัดการแอปพลิเคชันหลายตัวในคลัสเตอร์เดียวกันได้ง่ายขึ้น โดยที่ทรัพยากรใน Namespace หนึ่งจะไม่กระทบกับทรัพยากรใน Namespace อื่น
2. **การควบคุมการเข้าถึง**: ใช้สำหรับกำหนดสิทธิ์และนโยบายการเข้าถึงทรัพยากรใน Namespace แต่ละส่วนได้ เช่น การกำหนดสิทธิ์ให้ผู้ใช้หรือทีมที่แตกต่างกันเข้าถึง Namespace ของตัวเองเท่านั้น
3. **การจัดสรรทรัพยากร**: คุณสามารถกำหนด Quotas หรือ Resource Limits ให้กับแต่ละ Namespace เพื่อจัดสรรทรัพยากร เช่น CPU, Memory อย่างเหมาะสม

### ตัวอย่างการใช้งาน Namespace:
ในคลัสเตอร์เดียว คุณสามารถสร้าง Namespace หลาย ๆ อันสำหรับการแบ่งกลุ่มแอปพลิเคชัน เช่น:
- `dev`: สำหรับแอปพลิเคชันที่กำลังพัฒนา
- `prod`: สำหรับแอปพลิเคชันในสภาพแวดล้อมการผลิต

### การทำงานกับ Namespace:

1. **สร้าง Namespace**:
   ใช้คำสั่ง `kubectl` เพื่อสร้าง Namespace ใหม่:
   ```bash
   kubectl create namespace my-namespace
   ```

2. **ตรวจสอบ Namespace ทั้งหมด**:
   คุณสามารถดูรายการ Namespace ที่มีอยู่ในคลัสเตอร์ได้ด้วยคำสั่ง:
   ```bash
   kubectl get namespaces
   ```

3. **สร้างทรัพยากรใน Namespace เฉพาะ**:
   เมื่อคุณสร้างทรัพยากร เช่น Deployment หรือ Service คุณสามารถระบุ Namespace ได้ในไฟล์ YAML:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: my-deployment
     namespace: my-namespace  # ระบุ Namespace ที่ต้องการใช้
   spec:
     replicas: 2
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
           image: nginx
   ```

4. **สลับ Namespace เริ่มต้น**:
   เมื่อทำงานกับ Kubernetes คุณสามารถสลับ Namespace ที่ใช้โดยการตั้งค่า context:
   ```bash
   kubectl config set-context --current --namespace=my-namespace
   ```

5. **ลบ Namespace**:
   การลบ Namespace จะลบทรัพยากรทั้งหมดใน Namespace นั้น:
   ```bash
   kubectl delete namespace my-namespace
   ```

### Namespace เริ่มต้นใน Kubernetes:
- **default**: Namespace เริ่มต้นที่ถูกใช้เมื่อไม่ได้ระบุ Namespace ใด ๆ
- **kube-system**: Namespace ที่ใช้สำหรับเก็บทรัพยากรและบริการที่ถูกสร้างโดย Kubernetes เอง เช่น kube-dns, kube-proxy
- **kube-public**: Namespace พิเศษที่สามารถเข้าถึงได้จากทุกคน โดยทั่วไปจะถูกใช้สำหรับทรัพยากรที่ต้องการให้แชร์กันทั้งคลัสเตอร์
- **kube-node-lease**: ใช้สำหรับจัดการ heartbeat ของ node ในการช่วยตรวจสอบการทำงานของ nodes

### ข้อดีของ Namespace:
- การจัดการทรัพยากรได้เป็นระเบียบ
- แยกการทำงานของทีมพัฒนา (dev, test, prod) หรือโปรเจกต์ต่าง ๆ ได้อย่างชัดเจน
- รองรับการควบคุมสิทธิ์ (RBAC) และการจัดการทรัพยากร (Resource Quota)

### ข้อควรทราบ:
Namespace ไม่ได้แยกทรัพยากรทุกประเภทใน Kubernetes เช่น **Nodes** และ **PersistentVolumes** จะยังคงเป็นทรัพยากรระดับคลัสเตอร์