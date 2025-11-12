# Security Context

**Security Context** ใน Kubernetes เป็นฟีเจอร์ที่ช่วยในการกำหนดและควบคุมนโยบายด้านความปลอดภัยสำหรับ Pods และ Containers โดยการกำหนด Security Context จะช่วยให้สามารถระบุสิทธิ์การเข้าถึงและการดำเนินการที่อนุญาตสำหรับทรัพยากรภายในคลัสเตอร์ ซึ่งมีความสำคัญสำหรับการป้องกันและควบคุมการเข้าถึงข้อมูลและฟังก์ชันต่าง ๆ ในแอปพลิเคชัน

### คุณสมบัติของ Security Context

1. **User and Group ID**: กำหนด UID (User ID) และ GID (Group ID) ที่จะใช้สำหรับ Container หากไม่ระบุ จะใช้ UID และ GID ของผู้ใช้ที่เป็นเจ้าของอิมเมจของ Container

2. **Run As User**: กำหนดว่า Container จะทำงานในฐานะผู้ใช้ที่ระบุหรือไม่ ซึ่งสามารถช่วยในการเพิ่มความปลอดภัย โดยการหลีกเลี่ยงการทำงานในฐานะ root

3. **Privileges**: กำหนดว่าคอนเทนเนอร์จะมีสิทธิ์ที่สูงขึ้นหรือไม่ เช่น การใช้งาน Capabilities ที่สูงขึ้น

4. **SELinux Options**: การกำหนดการเข้าถึงแบบ SELinux เพื่อควบคุมการเข้าถึงทรัพยากรในระดับที่สูงขึ้น

5. **FSGroup**: กำหนดกลุ่มที่ใช้ในการควบคุมการเข้าถึงไฟล์ในระบบไฟล์ร่วมกันระหว่าง Pods

6. **Seccomp**: การใช้ Seccomp profile เพื่อจำกัดระบบเรียกใช้ (system calls) ที่ Container สามารถเรียกใช้ได้

7. **AppArmor**: การกำหนด AppArmor profiles เพื่อควบคุมความปลอดภัยของการทำงานของแอปพลิเคชันใน Container

### ตัวอย่างการกำหนด Security Context ใน YAML

ตัวอย่างการใช้ Security Context ในไฟล์ YAML ของ Pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  securityContext:
    runAsUser: 1000       # ระบุ UID ที่จะใช้
    runAsGroup: 3000      # ระบุ GID ที่จะใช้
    fsGroup: 2000         # ระบุ GID สำหรับการเข้าถึงไฟล์
    seLinuxOptions:
      level: "s0:c123,c456"  # SELinux Level
  containers:
  - name: my-container
    image: nginx
    securityContext:
      allowPrivilegeEscalation: false  # ป้องกันการเพิ่มสิทธิ์
      capabilities:
        drop:
          - ALL  # ลดสิทธิ์การเข้าถึง
```

### วิธีการใช้ Security Context

1. **กำหนดที่ Pod Level**: สามารถกำหนด Security Context ที่ระดับ Pod ซึ่งจะนำไปใช้กับทุก Container ภายใน Pod

2. **กำหนดที่ Container Level**: สามารถกำหนด Security Context เฉพาะสำหรับแต่ละ Container

### ข้อดีของ Security Context

- **เพิ่มความปลอดภัย**: ช่วยในการป้องกันการเข้าถึงและการดำเนินการที่ไม่เหมาะสม
- **ควบคุมสิทธิ์**: ทำให้สามารถกำหนดสิทธิ์ของผู้ใช้และกลุ่มได้อย่างชัดเจน
- **ลดความเสี่ยง**: การทำงานในฐานะผู้ใช้ที่ไม่ใช่ root ช่วยลดความเสี่ยงจากช่องโหว่ด้านความปลอดภัย

### สรุป

**Security Context** เป็นเครื่องมือที่มีประโยชน์ใน Kubernetes ที่ช่วยให้คุณสามารถกำหนดและควบคุมนโยบายด้านความปลอดภัยสำหรับ Pods และ Containers ได้อย่างมีประสิทธิภาพ โดยช่วยเพิ่มความปลอดภัยและควบคุมการเข้าถึงทรัพยากรภายในคลัสเตอร์