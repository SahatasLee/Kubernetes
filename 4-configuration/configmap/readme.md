# ConfigMap

Share Values

```bash
# Create from literal
kubectl create configmap <configmap-name> --from-literal=<key>=<value>
# Example
kubectl create configmap app-config --from-literal=APP_COLOR=blue --from-literal=APP_MODE=prod
# Create from file
kubectl create configmap <configmap-name> --from-file=<path-to-file>
# Example
kubectl create configmap <configmap-name> --from-file=app_config.properties
# 
kubectl create -f <path-to-file>
```

Env

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
    name: app-config
data:
    APP_COLOR: blue
    APP_MODE: prod
```

Configmap in Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: webapp
    labels:
        name: webapp
spec:
    container:
    - name: webapp
      image: 
      ports:
        - containerPort: 8080
      envFrom:
        - configMapRef:
            name: app-config
```

Single Env

```yaml

```

Volume

```yaml
volumes:
    - name: app-volume
      configMap:
        name: app-config
```

**ConfigMap** ใน Kubernetes เป็นทรัพยากรที่ใช้สำหรับเก็บข้อมูลการตั้งค่า (configuration data) ที่ไม่เป็นความลับ โดยสามารถใช้งานได้กับ Pods และบริการต่าง ๆ ในคลัสเตอร์ ConfigMap ช่วยให้คุณสามารถแยกการตั้งค่าที่อาจมีการเปลี่ยนแปลงบ่อยออกจากโค้ดของแอปพลิเคชัน ทำให้การปรับเปลี่ยนการตั้งค่าทำได้ง่ายขึ้นโดยไม่ต้องสร้างหรืออัปเดตภาพของคอนเทนเนอร์

### คุณสมบัติหลักของ ConfigMap:
1. **การจัดเก็บข้อมูลการตั้งค่า**: ConfigMap สามารถเก็บค่าการตั้งค่าหรือข้อมูลคอนฟิกในรูปแบบ key-value pairs
2. **การเข้าถึงข้อมูลได้หลายรูปแบบ**: ConfigMap สามารถถูกใช้ใน Pods ผ่าน environment variables, command-line arguments, หรือในไฟล์ที่ถูก mount เป็น volumes
3. **การแยกการตั้งค่าออกจากโค้ด**: ช่วยให้คุณจัดการการตั้งค่าของแอปพลิเคชันได้อย่างสะดวกและทำให้การปรับเปลี่ยนค่าต่าง ๆ ทำได้ง่ายขึ้น

### โครงสร้างของ ConfigMap:
ConfigMap ประกอบด้วย metadata และ data ที่ระบุค่าการตั้งค่าต่าง ๆ

### ตัวอย่างไฟล์ YAML ของ ConfigMap:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  DATABASE_URL: "mysql://user:password@hostname:3306/dbname"
  LOG_LEVEL: "info"
```

### การทำงานกับ ConfigMap:

1. **สร้าง ConfigMap**:
   ใช้คำสั่ง `kubectl` เพื่อนำไฟล์ ConfigMap ขึ้นสู่คลัสเตอร์:
   ```bash
   kubectl apply -f configmap.yaml
   ```

2. **ตรวจสอบ ConfigMap**:
   สามารถตรวจสอบ ConfigMap ที่มีอยู่ได้ด้วยคำสั่ง:
   ```bash
   kubectl get configmaps
   ```

3. **ดูรายละเอียดของ ConfigMap**:
   สามารถดูรายละเอียดของ ConfigMap ได้ด้วยคำสั่ง:
   ```bash
   kubectl describe configmap my-config
   ```

4. **ลบ ConfigMap**:
   การลบ ConfigMap สามารถทำได้ง่าย ๆ ด้วยคำสั่ง:
   ```bash
   kubectl delete configmap my-config
   ```

### การใช้งาน ConfigMap ใน Pods:
ConfigMap สามารถนำไปใช้งานใน Pods ได้หลายวิธี:

1. **ผ่าน Environment Variables**:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: my-pod
   spec:
     containers:
     - name: my-container
       image: nginx
       env:
       - name: DATABASE_URL
         valueFrom:
           configMapKeyRef:
             name: my-config
             key: DATABASE_URL
   ```

2. **ผ่าน Command-Line Arguments**:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: my-pod
   spec:
     containers:
     - name: my-container
       image: nginx
       command: ["nginx", "-g", "daemon off;"]
       args: ["--database-url", "$(DATABASE_URL)"]
       env:
       - name: DATABASE_URL
         valueFrom:
           configMapKeyRef:
             name: my-config
             key: DATABASE_URL
   ```

3. **Mount เป็น Volume**:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: my-pod
   spec:
     containers:
     - name: my-container
       image: nginx
       volumeMounts:
       - name: config-volume
         mountPath: /etc/config
     volumes:
     - name: config-volume
       configMap:
         name: my-config
   ```

### ข้อดีของ ConfigMap:
- **ความยืดหยุ่น**: การเปลี่ยนแปลงการตั้งค่าทำได้ง่ายโดยไม่ต้องสร้างภาพใหม่
- **การจัดการที่ง่าย**: ช่วยให้สามารถจัดการการตั้งค่าที่ไม่ใช่ความลับในที่เดียว
- **การแชร์การตั้งค่า**: สามารถใช้ ConfigMap เดียวกันในหลาย Pods ได้

### ข้อควรระวัง:
- ConfigMap ไม่ควรใช้ในการเก็บข้อมูลที่เป็นความลับ เช่น รหัสผ่าน หรือคีย์ API เนื่องจาก ConfigMap จะถูกเก็บในฐานข้อมูลของ Kubernetes โดยไม่เข้ารหัส ควรใช้ **Secrets** สำหรับข้อมูลที่เป็นความลับ

### สรุป:
**ConfigMap** เป็นทรัพยากรที่มีประโยชน์ใน Kubernetes สำหรับการจัดการการตั้งค่าของแอปพลิเคชัน ทำให้การปรับเปลี่ยนค่าและการจัดการข้อมูลการตั้งค่าสามารถทำได้อย่างมีประสิทธิภาพและยืดหยุ่น
