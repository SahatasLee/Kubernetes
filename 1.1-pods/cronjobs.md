# Cronjobs

**CronJobs** ใน Kubernetes คือทรัพยากรที่ใช้เพื่อจัดการงานที่ต้องการรันเป็นระยะเวลาที่กำหนด (ตามรูปแบบ cron) เหมือนกับการใช้ cron jobs ในระบบปฏิบัติการ Linux แต่ CronJobs ใน Kubernetes สามารถจัดการ Pod ที่สร้างขึ้นเพื่อทำงานนั้น ๆ โดยอัตโนมัติ

### การทำงานของ CronJobs:
- **CronJob** จะสร้างและจัดการ **Jobs** ตามตารางเวลาที่กำหนด (เช่นทุกวัน, ทุกสัปดาห์, หรือทุก 5 นาที)
- Jobs ที่ถูกสร้างโดย CronJob จะทำงานตามที่กำหนดและจบการทำงานเมื่อเสร็จสิ้น
- CronJob ใช้สำหรับงานที่ต้องการรันเป็นช่วงเวลาที่แน่นอน เช่น การสำรองข้อมูล, การลบ log, หรือการรันกระบวนการ maintenance

### คำสั่ง **schedule**:
- **schedule** ใช้สำหรับกำหนดช่วงเวลาที่งานต้องถูกรัน โดยใช้รูปแบบ cron
- รูปแบบคือ: `* * * * *`
  - ตัวอย่าง:
    - `"*/5 * * * *"` = ทุก ๆ 5 นาที
    - `"0 0 * * *"` = ทุกวันเวลาเที่ยงคืน
    - `"0 0 * * 0"` = ทุกวันอาทิตย์เวลาเที่ยงคืน

### ตัวอย่างการใช้งาน CronJob:

#### ตัวอย่าง CronJob ที่รันทุก 5 นาที:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: my-cronjob
spec:
  schedule: "*/5 * * * *"     # รันทุก 5 นาที
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: myjob
            image: busybox
            command: ["echo", "Hello from the CronJob!"]
          restartPolicy: OnFailure
```

### ฟิลด์สำคัญใน CronJob:
1. **schedule**: ระบุเวลาที่ต้องการให้รันงานตามรูปแบบ cron
2. **jobTemplate**: เป็นเทมเพลตของ Job ที่จะสร้างขึ้นทุกครั้งตามกำหนดเวลา
3. **spec** (ของ Job): กำหนดพฤติกรรมของ Pods ที่จะถูกรัน เช่น คำสั่ง, คอนเทนเนอร์, และ restart policy

### การตั้งค่าพิเศษใน CronJob:
1. **startingDeadlineSeconds**: กำหนดเวลาที่ Job สามารถเริ่มงานได้ หากเกิดความล่าช้าในการรัน (เช่น Server ล่ม) หากพ้นเวลานี้ไป Job จะไม่ถูกสร้าง
   ```yaml
   startingDeadlineSeconds: 100
   ```

2. **concurrencyPolicy**: กำหนดว่าจะทำอย่างไรกับ Jobs ที่ซ้ำซ้อนกัน (เช่น Job ใหม่ถูกรันในขณะที่ Job เก่ายังไม่เสร็จ)
   - **Allow**: รัน Jobs พร้อมกันได้
   - **Forbid**: ไม่อนุญาตให้รัน Jobs ซ้อนกัน
   - **Replace**: ยกเลิก Job เก่าแล้วรัน Job ใหม่แทน
   ```yaml
   concurrencyPolicy: Forbid
   ```

3. **successfulJobsHistoryLimit**: กำหนดจำนวน Job ที่สำเร็จแล้วที่ต้องเก็บประวัติไว้ (ค่าปกติคือ 3)
   ```yaml
   successfulJobsHistoryLimit: 3
   ```

4. **failedJobsHistoryLimit**: กำหนดจำนวน Job ที่ล้มเหลวแล้วที่ต้องเก็บประวัติไว้ (ค่าปกติคือ 1)
   ```yaml
   failedJobsHistoryLimit: 1
   ```

### การตรวจสอบสถานะของ CronJob:
- ดูรายการ CronJobs ที่มีอยู่:
  ```bash
  kubectl get cronjob
  ```

- ตรวจสอบสถานะของ CronJob:
  ```bash
  kubectl describe cronjob my-cronjob
  ```

- ดูประวัติการทำงานของ Job ที่ CronJob สร้าง:
  ```bash
  kubectl get jobs
  ```

### ตัวอย่างการใช้งาน CronJob ที่ซับซ้อนขึ้น:

#### กำหนดให้รัน Job ทุกวันเวลาเที่ยงคืน และให้จัดการกับการซ้ำซ้อน:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: daily-backup
spec:
  schedule: "0 0 * * *"        # รันทุกวันเวลาเที่ยงคืน
  concurrencyPolicy: Replace    # ยกเลิก Job เก่าถ้ามีงานซ้ำซ้อน
  successfulJobsHistoryLimit: 5 # เก็บประวัติ 5 Jobs ที่สำเร็จ
  failedJobsHistoryLimit: 2     # เก็บประวัติ 2 Jobs ที่ล้มเหลว
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: backup-image
            command: ["/bin/sh", "-c", "backup.sh"]
          restartPolicy: OnFailure
```

### ข้อดีของการใช้ CronJob:
- ช่วยจัดการงานที่ต้องการรันตามกำหนดเวลาได้อัตโนมัติ
- มีความยืดหยุ่นในการตั้งค่าและการจัดการเมื่อเกิดปัญหา เช่น ความล่าช้า หรือการซ้ำซ้อน