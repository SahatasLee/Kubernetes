# Logs

```sh
kubectl logs -f <pod-name> -c <container-name>
```

คำสั่ง `kubectl logs -f <pod-name>` ใช้เพื่อดู log ของ Pod ใน Kubernetes โดยเฉพาะในโหมด **ติดตาม (follow)** ซึ่งจะแสดง log แบบเรียลไทม์ หากมี log ใหม่ๆ เข้ามา คำสั่งนี้จะอัปเดตอย่างต่อเนื่องจนกว่าจะหยุดคำสั่งเองด้วยการกด `Ctrl+C` หรือปิด terminal

ตัวอย่างการใช้งาน:

```bash
kubectl logs -f my-pod
```

ถ้า Pod มีหลาย container ภายในเดียวกัน คุณต้องระบุชื่อ container ด้วย:

```bash
kubectl logs -f my-pod -c my-container
```

นี่เป็นประโยชน์เมื่อต้องการติดตาม log ของ container ที่รันใน Pod นั้นๆ แบบเรียลไทม์