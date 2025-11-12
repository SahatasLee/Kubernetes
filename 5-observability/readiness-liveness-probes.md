# Readiness & Liveness Probes

ใน Kubernetes, **Readiness Probes** และ **Liveness Probes** ใช้สำหรับตรวจสอบสถานะของแอปพลิเคชันที่ทำงานอยู่ใน Pods เพื่อให้ Kubernetes สามารถจัดการการทำงานและการเข้าถึงของ Pods ได้อย่างเหมาะสม โดยมีความแตกต่างกันในการตรวจสอบการทำงานดังนี้:

### 1. **Readiness Probes**:
- ใช้เพื่อบอก Kubernetes ว่า Pod พร้อมที่จะรับทราฟฟิกหรือยัง
- หาก **Readiness Probe** ล้มเหลว Kubernetes จะหยุดส่งทราฟฟิกไปยัง Pod นั้น แต่ Pod จะยังคงทำงานอยู่
- เหมาะสำหรับกรณีที่แอปพลิเคชันต้องใช้เวลาในการเริ่มต้นหรือโหลดข้อมูลก่อนที่จะสามารถรับทราฟฟิกได้

### ตัวอย่างการใช้งาน Readiness Probe:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-app
    image: my-app:1.0
    readinessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 5  # รอ 5 วินาทีก่อนตรวจสอบครั้งแรก
      periodSeconds: 10       # ตรวจสอบทุก 10 วินาที
```
ในตัวอย่างนี้ Kubernetes จะตรวจสอบว่า Pod พร้อมที่จะรับทราฟฟิกหรือไม่โดยการส่ง HTTP GET ไปยัง `/healthz` ที่พอร์ต 8080 ทุก 10 วินาที ถ้าตรวจสอบสำเร็จ Pod จะถือว่าพร้อม

### 2. **Liveness Probes**:
- ใช้เพื่อบอก Kubernetes ว่าแอปพลิเคชันใน Pod ยังทำงานปกติหรือไม่
- หาก **Liveness Probe** ล้มเหลว Kubernetes จะทำการรีสตาร์ท Pod นั้น
- เหมาะสำหรับการตรวจสอบว่าแอปพลิเคชันไม่อยู่ในสภาวะค้าง (hang) หรือไม่ทำงาน

### ตัวอย่างการใช้งาน Liveness Probe:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-app
    image: my-app:1.0
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 3  # เริ่มตรวจสอบหลังจากผ่านไป 3 วินาที
      periodSeconds: 5        # ตรวจสอบทุก 5 วินาที
```
ในตัวอย่างนี้ Kubernetes จะส่ง HTTP GET ไปยัง `/healthz` ที่พอร์ต 8080 เพื่อตรวจสอบว่าแอปพลิเคชันยังทำงานอยู่ ถ้าตรวจสอบล้มเหลวหลายครั้งติดต่อกัน Kubernetes จะรีสตาร์ท Pod

### ประเภทของ Probes:
ทั้ง **Readiness** และ **Liveness Probes** มีรูปแบบในการตรวจสอบสถานะได้ 3 แบบ:
1. **HTTP GET**: ส่งคำขอ HTTP ไปยัง URL ที่กำหนดในแอปพลิเคชันและรอรับสถานะ HTTP Code (200-399 ถือว่าสำเร็จ)
   ```yaml
   httpGet:
     path: /healthz
     port: 8080
   ```
   
2. **TCP Socket**: ตรวจสอบว่าพอร์ตที่กำหนดใน Pod สามารถเชื่อมต่อได้หรือไม่
   ```yaml
   tcpSocket:
     port: 8080
   ```

3. **Exec Command**: รันคำสั่งในคอนเทนเนอร์และตรวจสอบรหัสสถานะของคำสั่ง (exit code 0 ถือว่าสำเร็จ)
   ```yaml
   exec:
     command:
     - cat
     - /tmp/healthy
   ```

### ฟิลด์ที่สำคัญใน Probe:
- **initialDelaySeconds**: ระยะเวลาที่จะรอก่อนเริ่มตรวจสอบครั้งแรก
- **periodSeconds**: ระยะเวลาระหว่างการตรวจสอบแต่ละครั้ง
- **timeoutSeconds**: ระยะเวลาที่จะรอให้การตรวจสอบเสร็จสิ้น
- **successThreshold**: จำนวนการตรวจสอบต่อเนื่องที่ต้องสำเร็จจึงจะถือว่าผ่าน
- **failureThreshold**: จำนวนการตรวจสอบต่อเนื่องที่ล้มเหลวจึงจะถือว่าล้มเหลว

### การใช้ทั้ง Readiness และ Liveness Probes ร่วมกัน:
บางแอปพลิเคชันอาจต้องการทั้งการตรวจสอบ Readiness และ Liveness เพื่อให้แน่ใจว่า:
- Pod พร้อมรับทราฟฟิกแล้วก่อนจะเปิดให้รับทราฟฟิก (Readiness Probe)
- แอปพลิเคชันยังคงทำงานปกติ หากไม่ทำงานให้ Kubernetes รีสตาร์ทแอปพลิเคชัน (Liveness Probe)

ตัวอย่างที่ใช้งานทั้ง Readiness และ Liveness Probes:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-app
    image: my-app:1.0
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 3
      periodSeconds: 5
```

### สรุป:
- **Readiness Probe**: ตรวจสอบว่า Pod พร้อมรับทราฟฟิกหรือยัง
- **Liveness Probe**: ตรวจสอบว่าแอปพลิเคชันใน Pod ยังทำงานปกติหรือไม่
- สามารถใช้ร่วมกันได้เพื่อให้ Kubernetes สามารถจัดการทั้งการเข้าถึงและการทำงานของแอปพลิเคชัน
