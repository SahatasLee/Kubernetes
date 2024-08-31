# Taint & Tolerations

taint-effect:
1. NoSchedule
2. PerferNoSchedule
3. NoExecute

```bash
kubectl taint node <node-name> key=value:taint-effect
```

tolerations pod

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: my-app
spec:
    containers:
    - name: nginx-container
      image: nginx
    tolerateions:
    - key: "app"
      operations: "Equal"
      value: "blue"
      effect: "NoSchedule"
```

**Taint Node** เป็นกลไกใน Kubernetes ที่ใช้เพื่อจำกัดหรือตัดสินใจว่าจะอนุญาตให้ Pods ใดสามารถรันบน Node ที่กำหนดได้ โดยการใช้ Taints เพื่อทำเครื่องหมาย Node ว่าไม่สามารถรัน Pods บางตัวได้ เว้นแต่ Pods นั้นจะมี Tolerations ที่ตรงกันกับ Taints ของ Node นั้น

### การทำงานของ Taints และ Tolerations

- **Taints**: เป็นการตั้งค่าให้กับ Node เพื่อบอกว่า Node นั้น "ไม่ยอมรับ" Pods ยกเว้นว่าจะมีการตั้งค่า Tolerations ที่ตรงกันใน Pods นั้น ๆ
- **Tolerations**: เป็นการตั้งค่าภายใน Pods ที่จะช่วยให้ Pods สามารถรันบน Nodes ที่มี Taints ได้ โดยยกเว้น Taints ที่ตรงกันนั้นเอง

### ตัวอย่างการใช้งาน

1. **ตั้งค่า Taint บน Node**:
   ```sh
   kubectl taint nodes <node-name> key=value:taint-effect
   ```
   ตัวอย่าง:
   ```sh
   kubectl taint nodes node1 key=value:NoSchedule
   ```
   ในกรณีนี้ `node1` จะไม่อนุญาตให้ Pods รันบนมัน เว้นแต่ Pods จะมี Toleration ที่ตรงกับ Taint นี้

2. **ตั้งค่า Toleration ใน Pod**:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: mypod
   spec:
     tolerations:
     - key: "key"
       operator: "Equal"
       value: "value"
       effect: "NoSchedule"
   containers:
   - name: mycontainer
     image: myimage
   ```

การใช้ Taints และ Tolerations ช่วยให้คุณสามารถควบคุมการกระจายของ Pods ใน Cluster ได้อย่างยืดหยุ่นมากขึ้น ซึ่งเหมาะสำหรับการตั้งค่า Nodes ที่ต้องการใช้งานเฉพาะด้าน เช่น Node สำหรับงานที่ต้องการทรัพยากรสูงหรือ Node ที่ต้องการใช้งานเฉพาะแอปพลิเคชันบางประเภท.