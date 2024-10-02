# Jobs

ใน Kubernetes, **Jobs** เป็นทรัพยากรที่ใช้เพื่อจัดการการทำงานแบบ batch หรือการรันงานที่ต้องการให้ทำเสร็จสิ้นเพียงครั้งเดียว ซึ่งแตกต่างจากการทำงานของ **Pods** หรือ **Deployments** ที่มุ่งเน้นการให้บริการอย่างต่อเนื่อง (long-running services)

### คอนเซปต์ของ Jobs:
- **Job** สร้างและจัดการ **Pod** ที่รันจนเสร็จสิ้น (ไม่เหมือน Deployment ที่ต้องรันตลอดเวลา)
- Job จะตรวจสอบให้แน่ใจว่า **Pod** ที่รันนั้นเสร็จสมบูรณ์ตามที่กำหนด (จำนวนการรันหรือ task ที่สำเร็จ)
- หาก Pod ล้มเหลวระหว่างการรัน Job จะสร้าง Pod ใหม่มาทำงานทดแทนจนกว่างานจะเสร็จ

### การทำงานของ Jobs:
- Job สามารถกำหนดให้รันได้หลายลักษณะ เช่น การรันเพียงครั้งเดียว หรือรันซ้ำในหลาย task
- เมื่อ Pod ภายใต้ Job เสร็จสิ้นการทำงาน (ไม่ว่าจะสำเร็จหรือล้มเหลว) Job จะถือว่าหน้าที่นั้นเสร็จสิ้น

### ประเภทของ Jobs:
1. **Non-parallel Job**: รันเพียง Pod เดียว และรอจน Pod นั้นทำงานเสร็จ
2. **Parallel Job with a fixed number of completions**: กำหนดให้สร้างหลาย Pod พร้อมกัน และแต่ละ Pod ต้องรันจนเสร็จ โดยจะเสร็จสิ้นเมื่อทุก Pod ทำงานครบตามจำนวนที่ตั้งไว้
3. **Parallel Job with a work queue**: ใช้ในกรณีที่ต้องการกระจายงานที่มีจำนวนไม่แน่นอนให้หลาย Pod ทำงานร่วมกัน

### ตัวอย่างการใช้งาน Jobs:

#### 1. **Simple Job**:
ตัวอย่างการรัน Pod เพียงหนึ่งครั้งโดย Job
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: simple-job
spec:
  template:
    spec:
      containers:
      - name: myjob
        image: busybox
        command: ["echo", "Hello, Kubernetes!"]
      restartPolicy: Never   # ไม่ให้ Pod รีสตาร์ทเมื่อทำงานสำเร็จ
```

#### 2. **Parallel Job**:
Job ที่รัน Pod หลาย ๆ ครั้งพร้อมกัน
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: parallel-job
spec:
  completions: 3            # จำนวนครั้งที่ต้องการให้ Pod ทำงานเสร็จ
  parallelism: 3            # จำนวน Pod ที่จะรันพร้อมกัน
  template:
    spec:
      containers:
      - name: myjob
        image: busybox
        command: ["echo", "Parallel Task"]
      restartPolicy: Never
```

### คุณสมบัติที่สำคัญของ Jobs:
- **completions**: จำนวนครั้งที่ต้องการให้ Pod ทำงานสำเร็จ (เช่น การรัน task)
- **parallelism**: จำนวน Pod ที่สามารถรันได้พร้อมกัน
- **restartPolicy**: กำหนดว่าต้องการให้ Pod รีสตาร์ทเมื่อมันล้มเหลวหรือไม่ โดยทั่วไปจะใช้ค่า `Never` หรือ `OnFailure`

### การตรวจสอบสถานะของ Jobs:
- ตรวจสอบสถานะของ Job (เช่น จำนวนการรันที่สำเร็จ):
  ```bash
  kubectl describe job simple-job
  ```
  
- ดู logs ของ Pod ที่รันภายใต้ Job:
  ```bash
  kubectl logs <pod-name>
  ```

### CronJobs:
สำหรับงานที่ต้องการรันเป็นช่วงเวลาที่กำหนด (เช่น ทุกวัน, ทุกชั่วโมง) Kubernetes ยังมี **CronJob** ที่ทำงานเหมือนกับ Jobs แต่สามารถกำหนดเวลาได้ตามรูปแบบของ cron

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cron-job-example
spec:
  schedule: "*/5 * * * *"    # กำหนดให้รันทุก 5 นาที
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: myjob
            image: busybox
            command: ["echo", "Scheduled Task"]
          restartPolicy: OnFailure
```

**CronJob** มีประโยชน์สำหรับการรันงานเป็นช่วงเวลาที่แน่นอน เช่น งาน backup, การเก็บ log, หรือการทำ maintenance อัตโนมัติ