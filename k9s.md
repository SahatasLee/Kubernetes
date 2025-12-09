# คู่มือการใช้งาน k9s (Kubernetes CLI)

**k9s** เป็นเครื่องมือแบบ Terminal-based UI ที่ช่วยให้เราจัดการและดูสถานะของ Kubernetes Cluster ได้ง่ายและรวดเร็ว โดยไม่ต้องจำคำสั่ง `kubectl` ยาวๆ ช่วยเพิ่มความสะดวกในการ Monitor และแก้ปัญหาต่างๆ ใน Cluster

## 1. การติดตั้ง (Installation)

### macOS (ผ่าน Homebrew)
วิธีที่ง่ายที่สุดสำหรับคนใช้ Mac:
```bash
brew install k9s
```

### Linux
สามารถติดตั้งผ่าน LinuxBrew หรือดาวน์โหลด Binary โดยตรง:
```bash
# ถ้ามี brew
brew install k9s

# หรือติดตั้งผ่าน Pacman (Arch Linux)
pacman -S k9s
```

### Windows
ติดตั้งผ่าน Winget หรือ Scoop:
```powershell
# ผ่าน Scoop
scoop install k9s

# ผ่าน Winget
winget install k9s
```

### ตรวจสอบการติดตั้ง
```bash
k9s version
```

---

## 2. การใช้งานเบื้องต้น (Basic Usage)

เปิดโปรแกรม:
```bash
k9s
```
*ถ้ามีหลาย Context หรือ Kubeconfig สามารถระบุได้ แต่โดยปกติมันจะอ่านจาก `~/.kube/config` ปัจจุบัน*

### การนำทาง (Navigation)

การสั่งงานใน k9s ใช้ **Shortcuts** เป็นหลัก:

| ปุ่ม | การทำงาน |
| :--- | :--- |
| **`:`** (Colon) | เข้าสู่โหมดคำสั่ง (Command Mode) เพื่อเปลี่ยนหน้าจอ Resource |
| **`/`** (Slash) | ค้นหา / กรองข้อมูลในหน้านั้น (Filter) |
| **`?`** | ดูเมนูช่วยเหลือ (Help) แสดงปุ่มลัดทั้งหมดในหน้านั้น |
| **`Esc`** | ย้อนกลับไปหน้าก่อนหน้า หรือยกเลิกการค้นหา |
| **`Ctrl-c`** | ออกจากโปรแกรม k9s |
| **`Enter`** | เข้าไปดูรายละเอียด (Drill down) เช่น เข้าไปดู Container ใน Pod |

### ตัวอย่างคำสั่งเปลี่ยนหน้าจอ (Command Mode)
พิมพ์ `:` แล้วตามด้วยชื่อ Resource (ใช้ตัวย่อได้):

- `:pods` หรือ `:po` -> ไปหน้า Pods
- `:svc` -> ไปหน้า Services
- `:deploy` -> ไปหน้า Deployments
- `:ns` -> ไปหน้าเลือก Namespace
- `:ctx` -> ไปหน้าเลือก Context (Cluster)
- `:nodes` หรือ `:no` -> ไปหน้า Nodes

---

## 3. คำสั่งที่ใช้บ่อย (Common Operations)

เมื่ออยู่ที่หน้า **Pods** (`:pods`):

1. **ดู Logs**
   - กด **`l`** (L เล็ก): ดู logs ของ Pod ที่เลือก
   - กด **`Esc`**: ออกจากหน้า logs
   - **Tips:** ในหน้า logs กด **`s`** เพื่อเปิด/ปิดการเลื่อนอัตโนมัติ (Auto-scroll) / กด **`0`** (เลขศูนย์) เพื่อดู logs ของทุก Container รวมกัน

2. **Remote Shell (Exec)**
   - กด **`s`**: เข้าไป Shell ใน Container นั้น (เหมือน `kubectl exec -it ...`)
   - พิมพ์ `exit` เพื่อออกมา

3. **Port Forward**
   - เลือก Pod แล้วกด **`Shift-f`**: เพื่อทำ Port Forward
   - จะมีหน้าต่างเด้งขึ้นมาให้ระบุ Port

4. **ลบ Pod**
   - กด **`Ctrl-d`** หรือปุ่ม **`Delete`** (Mac บางเครื่องต้องกด `Fn+Delete`)
   - จะมีหน้าต่างยืนยัน ให้กด `Enter` เพื่อยืนยันการลบ

5. **ดู YAML / Describe**
   - กด **`y`**: ดู YAML ของ resource
   - กด **`d`**: ดู Describe (รายละเอียด Events, Status)

---

## 4. ฟีเจอร์ขั้นสูง (Advanced Features)

### X-Ray View (`:xray`)
ช่วยดูความสัมพันธ์ของ Resource (Dependencies) ว่ามันเกี่ยวข้องกันอย่างไร
```
:xray deploy
```
*จะเห็น Tree view ว่า Deployment นี้สร้าง ReplicaSet และ Pods อะไรบ้าง*

### Pulse Mode (`:pulse`)
หน้า Dashboard สรุปสถานะสุขภาพของ Cluster แบบกราฟิก ดูง่ายๆ ว่ามีอะไรพังไหม
```
:pulse
```

### Popeye Integration
ถ้าติดตั้ง Popeye ไว้ k9s จะสแกนหาปัญหา Best Practices ให้ด้วย แต่ปกติใช้แค่ดูสถานะทั่วไปก็เพียงพอ

### การเปลี่ยน Namespace
- กด **`:`** พิมพ์ `ns` แล้ว Enter
- เลือก Namespace ที่ต้องการ แล้วกด **`Enter`** เพื่อ Switch
- หรือกด **`Ctrl-n`** เพื่อดู Namespace ทั้งหมดที่ Pods นั้นอยู่ (ในบางเวอร์ชัน)

---

## 5. Tips & Tricks

- **Filter ไวๆ**: ในหน้า Pods ที่เยอะๆ กด `/` แล้วพิมพ์ชื่อบางส่วน เช่น `error` หรือชื่อ service เพื่อกรองเฉพาะตัวที่สนใจ
- **เรียงลำดับ**: กด `Shift+i` (เรียงตาม IP), `Shift+s` (สถานะ), `Shift+a` (อายุ) *shortcut อาจต่างกันในแต่ละ version ลองกด `?` เพื่อเช็ค*
- **ดู Resource Usage**: ถ้า Cluster ลง Metrics Server ไว้ จะเห็น Column CPU/MEM ของแต่ละ Pod เลย ช่วย debug เรื่อง Performance ได้ดีมาก
- **Benchmarks**: k9s มี tool ยิง load test ง่ายๆ ในตัว (hey) แต่ต้องระวังการใช้บน Production

ดูข้อมูลเพิ่มเติมและ Shortcut ทั้งหมดได้ที่ [K9s Repository](https://github.com/derailed/k9s)

---

## 6. การปรับแต่งหน้าตา (Customization)

เราสามารถเปลี่ยนสีและ Theme ของ k9s ได้ผ่านไฟล์ **Skin** ครับ

### วิธีเปลี่ยน Theme (Skins)

1. **หาไฟล์ Config**:
   - **macOS / Linux**: อยู่ที่ `~/.config/k9s/config.yml`
   - **Windows**: อยู่ที่ `%LOCALAPPDATA%\k9s\config.yml`

2. **เลือก Skin**:
   ไปดูตัวอย่าง Theme สวยๆ ได้ที่ [K9s Skins Gallery](https://github.com/derailed/k9s/tree/master/skins) (เช่น Dracula, Nord, OneDark)

3. **ติดตั้ง**:
   - สร้างโฟลเดอร์ `skins` ใน `~/.config/k9s/`
   - ดาวน์โหลดไฟล์ `.yml` ของ theme ที่ชอบไปใส่ในโฟลเดอร์นั้น (เช่น `dracula.yml`)
   - แก้ไขไฟล์ `config.yml` ของ k9s ให้ชี้ไปที่ skin นั้น:
     ```yaml
     k9s:
       ui:
         skin: dracula  # ไม่ต้องใส่นามสกุล .yml
     ```
