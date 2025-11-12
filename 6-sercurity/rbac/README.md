# Role Base Access Control

แน่นอน 😊 มาดูเรื่อง **Kubernetes Role-Based Access Control (RBAC)** กันแบบเข้าใจง่ายและครบถ้วน

---

## 🧩 **ภาพรวม: RBAC คืออะไร**

**RBAC (Role-Based Access Control)** เป็นระบบควบคุมสิทธิ์การเข้าถึง (Access Control) ใน Kubernetes
ใช้เพื่อกำหนดว่า “**ใคร (Who)**” สามารถ “**ทำอะไร (What)**” กับ “**ทรัพยากร (Resource)**” ภายใน cluster ได้บ้าง

👉 RBAC จะทำงานร่วมกับ **API Server** ซึ่งเป็นจุดกลางในการจัดการทุกคำสั่งใน Kubernetes

---

## 🔑 **แนวคิดหลักใน RBAC**

RBAC ประกอบด้วย 4 object หลัก:

| Component              | หน้าที่                                                                          |
| ---------------------- | -------------------------------------------------------------------------------- |
| **Role**               | กำหนดสิทธิ์ (permissions) เช่น อ่าน, สร้าง, ลบ บน resource ภายใน namespace เดียว |
| **ClusterRole**        | เหมือน Role แต่ใช้ได้ทั้ง cluster (ไม่จำกัด namespace)                           |
| **RoleBinding**        | ผูก (bind) Role เข้ากับ user/serviceAccount/group เพื่อให้สิทธิ์ใน namespace     |
| **ClusterRoleBinding** | ผูก ClusterRole เข้ากับ user/serviceAccount/group ให้สิทธิ์ทั่ว cluster          |

---

## 🧠 **โครงสร้างสิทธิ์ใน Role / ClusterRole**

ใน Role จะระบุ:

* `apiGroups` → กลุ่มของ API เช่น `""` (core), `apps`, `batch`
* `resources` → ทรัพยากร เช่น pods, deployments, services
* `verbs` → การกระทำ เช่น get, list, create, delete, update

ตัวอย่าง:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

🔹 Role นี้ให้สิทธิ์อ่านข้อมูล Pod (แต่ไม่สามารถลบหรือสร้างได้)
เฉพาะใน namespace `dev`

---

## 🔗 **RoleBinding**

ใช้เพื่อ “เชื่อม” Role เข้ากับ User หรือ ServiceAccount

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: dev
subjects:
- kind: User
  name: alice
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

🔹 ตัวอย่างนี้ให้ user `alice` สามารถอ่าน Pod ใน namespace `dev`

---

## 🌍 **ClusterRole / ClusterRoleBinding**

ถ้าต้องการให้สิทธิ์ระดับ cluster เช่น

* ดู nodes ทั้งหมด
* เข้าถึง resource ทุก namespace

จะใช้ ClusterRole + ClusterRoleBinding เช่น:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-reader
rules:
- apiGroups: [""]
  resources: ["pods", "nodes"]
  verbs: ["get", "list", "watch"]
```

```yaml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: cluster-reader-binding
subjects:
- kind: User
  name: bob
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-reader
  apiGroup: rbac.authorization.k8s.io
```

---

## 👥 **Subjects ที่สามารถผูกได้**

ใน `RoleBinding` หรือ `ClusterRoleBinding`
เราสามารถผูกกับสิ่งเหล่านี้ได้:

| kind             | ใช้เมื่อ                        |
| ---------------- | ------------------------------- |
| `User`           | ผู้ใช้ภายนอก                    |
| `Group`          | กลุ่มผู้ใช้                     |
| `ServiceAccount` | แอปหรือ Pod ที่รันภายใน cluster |

---

## 💡 **สรุปภาพรวมความสัมพันธ์**

```
User / SA / Group
       ↓ (Binding)
     Role / ClusterRole
       ↓ (Rules)
  Resources + Verbs
```

---

## ⚙️ **ตัวอย่างการใช้งานจริง**

เช่น:

* DevOps ต้องการให้ Jenkins deploy ไป namespace `staging` ได้ → สร้าง Role + RoleBinding ให้ Jenkins SA
* Developer ต้องการดู logs pods ใน namespace ตัวเอง → สร้าง Role ที่มี “get, list” pods/logs
* Admin ต้องจัดการทั้ง cluster → ClusterRole + ClusterRoleBinding

---

## 🔐 **Best Practices**

✅ ใช้ **Least Privilege Principle** — ให้เฉพาะสิทธิ์ที่จำเป็นเท่านั้น
✅ แยก **Role** ตาม function (เช่น read-only, admin, deployer)
✅ ใช้ **ServiceAccount** สำหรับ workloads แทน user จริง
✅ ตรวจสอบสิทธิ์ด้วยคำสั่ง:

```bash
kubectl auth can-i get pods --as alice -n dev
```

---

อยากให้ผมวาด **แผนภาพ RBAC flow** ให้ดูไหมครับ? จะเห็นภาพชัดขึ้นว่า Role/Binding ทำงานกันยังไงใน Cluster 🚀
