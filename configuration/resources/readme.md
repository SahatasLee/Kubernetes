# Resources

## Limit Range

limit-range-cpu.yaml

```yaml
apiVersion: v1
kind: LimitRange
metadata:
    name: limit-range-cpu
spec:
    limits:
    - default:
        cpu: 500m
      defaultRequest:
        cpu: 500m
      max:
        cpu: 1
      min:
        cpu: 100m
      type: container
```

limit-range-memory.yaml

```yaml
apiVersion: v1
kind: LimitRange
metadata:
    name: limit-range-memory
spec:
    limits:
    - default:
        memory: 1Gi
      defaultRequest:
        memory: 1Gi
      max:
        memory: 1Gi
      min:
        memory: 500Mi
      type: container
```

## Resource Quota

resource-quota.yaml

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
    name: my-resource-quota
spec:
    hard:
        requests.cpu: 4
        requests.memory: 4Gi
        limit.cpu: 10
        limit.memory: 10Gi
```

ใน Kubernetes, **Resource** หมายถึงการจัดการและควบคุมทรัพยากรที่ใช้ในการทำงานของแอปพลิเคชันในคลัสเตอร์ โดยทรัพยากรใน Kubernetes แบ่งออกเป็นหลายประเภท และแต่ละประเภทจะมีวิธีการจัดการที่แตกต่างกัน นี่คือประเภททรัพยากรหลักใน Kubernetes:

### ประเภทของ Resources ใน Kubernetes:

1. **CPU**: หน่วยประมวลผลกลาง ซึ่งใช้ในการคำนวณและประมวลผลข้อมูล
2. **Memory**: หน่วยความจำที่ใช้ในการจัดเก็บข้อมูลและการประมวลผล
3. **Storage**: การเก็บข้อมูลในรูปแบบที่ไม่เปลี่ยนแปลง (Persistent storage) โดยสามารถใช้ Volume ต่าง ๆ ใน Kubernetes เช่น Persistent Volumes (PV) และ Persistent Volume Claims (PVC)

### การกำหนด Resource Requests และ Limits:
ใน Kubernetes, คุณสามารถกำหนด resource requests และ limits สำหรับ Pods และ Containers เพื่อจัดการทรัพยากรอย่างมีประสิทธิภาพ

- **Requests**: เป็นจำนวนทรัพยากรขั้นต่ำที่ Pod ต้องการเพื่อเริ่มทำงาน โดย Kubernetes จะรับประกันว่าทรัพยากรที่ร้องขอจะมีให้
- **Limits**: เป็นจำนวนทรัพยากรสูงสุดที่ Pod สามารถใช้ได้ ถ้า Pod ใช้ทรัพยากรเกินกว่าที่กำหนดไว้ใน limits มันจะถูกจำกัดหรือหยุดการทำงาน

### ตัวอย่างการกำหนด Resource Requests และ Limits ใน YAML:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

### การจัดการ Resources:
Kubernetes ใช้ **Scheduler** ในการจัดการและจัดสรรทรัพยากรให้กับ Pods โดยจะพิจารณาจาก:

- ทรัพยากรที่ร้องขอใน requests
- ทรัพยากรที่มีอยู่ในโหนด (nodes) ต่าง ๆ
- Policy ที่กำหนดไว้ในคลัสเตอร์

### Resource Quotas:
Kubernetes ยังมีคุณสมบัติ **Resource Quotas** ที่ช่วยในการจำกัดการใช้ทรัพยากรใน namespace หนึ่ง ๆ เช่น จำนวน Pods, จำนวน CPU หรือ Memory ที่สามารถใช้งานได้ใน namespace นั้น ๆ

### ตัวอย่าง Resource Quotas:
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-quota
spec:
  hard:
    requests.cpu: "4"
    requests.memory: "8Gi"
    limits.cpu: "10"
    limits.memory: "16Gi"
    count/pods: "10"
```

### สรุป:
**Resources** ใน Kubernetes มีความสำคัญสำหรับการจัดการทรัพยากรในคลัสเตอร์ เพื่อให้แน่ใจว่าแอปพลิเคชันทำงานได้อย่างมีประสิทธิภาพและไม่ขัดแย้งกัน โดยการกำหนด resource requests และ limits ช่วยให้การควบคุมและจัดการทรัพยากรเป็นไปอย่างมีระเบียบและเป็นระบบ