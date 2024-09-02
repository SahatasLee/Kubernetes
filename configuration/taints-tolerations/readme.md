# Taint & Tolerations

[taints management blog](https://www.baeldung.com/ops/kubernetes-taints-managementhttps://www.baeldung.com/ops/kubernetes-taints-management)

Taint Node

```bash
kubectl taint node <node-name> key=value:taint-effect
```

taint-effect:
1. NoSchedule
2. PerferNoSchedule
3. NoExecute

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

## การทำงานของ Taints และ Tolerations

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

## Remove taint node

ในการลบ taint จาก Node ใน Kubernetes คุณสามารถใช้คำสั่ง `kubectl taint` โดยระบุชื่อ Node และชื่อของ taint ที่ต้องการลบออกได้ ดังนี้:

```bash
kubectl taint nodes <node-name> <key>:<value>:<effect>-
```

### ตัวอย่าง

สมมติว่าคุณต้องการลบ taint ชื่อ `key=value:NoSchedule` จาก Node ชื่อ `node1`:

```bash
kubectl taint nodes node1 key=value:NoSchedule-
```

เครื่องหมาย `-` ที่ท้ายคำสั่งใช้ในการลบ taint ออกจาก Node นั้นๆ

หากคุณต้องการลบ taint ทั้งหมดจาก Node คุณสามารถใช้คำสั่ง:

```bash
kubectl taint nodes <node-name> --all
```

วิธีนี้จะลบ taint ทั้งหมดที่อยู่ใน Node ที่ระบุออกไปทั้งหมด.

คำสั่ง

```bash
kubectl taint nodes foo dedicated-
```

คำสั่งนี้จะลบ taints ทั้งหมดที่มี key ชื่อ `dedicated` ออกจาก Node ชื่อ `foo` โดยไม่สนใจค่า (`value`) หรือผลกระทบ (`effect`) ของ taints นั้น ๆ.

### ความหมายของคำสั่ง:

- `kubectl taint nodes` - คำสั่งที่ใช้เพื่อจัดการกับ taints ของ Node ใน Kubernetes
- `foo` - ชื่อของ Node ที่คุณต้องการลบ taints ออก
- `dedicated-` - ระบุว่าให้ลบ taints ที่มี key ชื่อ `dedicated` ออก

โดยใช้คำสั่งนี้ คุณจะลบ taints ทั้งหมดที่มี key ชื่อ `dedicated` จาก Node `foo` ซึ่งทำให้ Pod สามารถถูกจัดสรรไปยัง Node นั้นได้โดยไม่มีข้อจำกัดที่เกิดจาก taints เหล่านั้น.

## Using kubectl to List Node Taints

```bash
# JSON and JSONPath Output
$ kubectl get nodes -o jsonpath="{range .items[*]}{.metadata.name}:{' '}{range .spec.taints[*]}{.key}={.value}:{.effect},{' '}{end}{'\n'}{end}"
node1: app=blue:NoSchedule,
node2: gpu=exclusive:NoSchedule,
node3:

# Using JSON and jq
$ kubectl get nodes -o=json | jq -r '.items[] | .metadata.name + "\t" + (if .spec.taints then (.spec.taints | map(.key + "=" + (.value // "") + ":" + .effect) | join(", ")) else "No taints" end)'
node1   app=blue:NoSchedule, gpu=exclusive:PreferNoSchedule
node2   No taints
node3   storage=high:NoExecute

# Listing All Taints Across All Nodes
$ kubectl get nodes -o=jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.taints[*].key}{"\n"}{end}'
node1	app
node2	gpu
node3	

# Custom Queries for Complex Environments
$ kubectl get nodes -o custom-columns=NAME:.metadata.name,ARCH:.status.nodeInfo.architecture,KERNEL:.status.nodeInfo.kernelVersion,TAINTS:.spec.taints
NAME        ARCH       KERNEL         TAINTS
node1       amd64      4.19.0-6-amd64 [{"key":"app","value":"blue","effect":"NoSchedule"}]
node2       amd64      4.19.0-6-amd64 [{"key":"gpu","value":"exclusive","effect":"NoSchedule"}]
node3       amd64      4.19.0-6-amd64 []
```