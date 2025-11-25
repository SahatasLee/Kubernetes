# Headless Services

## บทนำ

Headless Service เป็น Service พิเศษใน Kubernetes ที่ไม่มีการจัดสรร Cluster IP และไม่ทำ Load Balancing แบบปกติ แต่จะให้ DNS ชี้ตรงไปยัง IP ของแต่ละ Pod โดยตรง

**จุดประสงค์หลัก:**
- ใช้สำหรับ StatefulSet เพื่อให้แต่ละ Pod มี DNS name ที่เป็นเอกลักษณ์
- ใช้เมื่อต้องการเชื่อมต่อกับ Pod เฉพาะตัวโดยตรง
- ใช้เมื่อต้องการ Service Discovery แบบที่ไม่ผ่าน Load Balancer

## ความแตกต่างระหว่าง Normal Service และ Headless Service

### Normal Service (ClusterIP)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080
  # มี clusterIP โดยอัตโนมัติ
```

**ลักษณะการทำงาน:**
- มี Cluster IP (เช่น 10.96.0.10)
- DNS จะ resolve ไปยัง Cluster IP
- Cluster IP จะทำ Load Balancing ไปยัง Pod ต่างๆ
- Client ไม่รู้ว่า Pod อยู่ที่ไหน

```bash
# DNS Query
nslookup my-service.default.svc.cluster.local
# Result: 10.96.0.10 (Cluster IP)

# การเชื่อมต่อ
curl http://my-service  # -> Load balanced ไปยัง Pod ใดๆ
```

### Headless Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-headless-service
spec:
  clusterIP: None  # นี่คือจุดสำคัญ!
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080
```

**ลักษณะการทำงาน:**
- **ไม่มี** Cluster IP
- DNS จะ resolve ไปยัง IP ของ Pod โดยตรง (ทุก Pod)
- ไม่มี Load Balancing จาก Service
- Client สามารถเลือกเชื่อมต่อกับ Pod ที่ต้องการได้

```bash
# DNS Query
nslookup my-headless-service.default.svc.cluster.local
# Result: 
# 10.244.0.5 (Pod-0)
# 10.244.0.6 (Pod-1)
# 10.244.0.7 (Pod-2)

# การเชื่อมต่อ
curl http://my-headless-service  
# -> DNS round-robin หรือ application เลือกเอง
```

## สร้าง Headless Service

### วิธีที่ 1: Basic Headless Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-headless-service
  namespace: default
spec:
  clusterIP: None  # สำคัญ! ตั้งเป็น None
  selector:
    app: my-app
  ports:
    - name: http
      port: 80
      targetPort: 8080
    - name: metrics
      port: 9090
      targetPort: 9090
```

### วิธีที่ 2: Headless Service สำหรับ StatefulSet

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-headless
  namespace: database
  labels:
    app: mysql
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
    - name: mysql
      port: 3306
      targetPort: 3306
  # StatefulSet จะใช้ service นี้สร้าง stable network identity
```

### วิธีที่ 3: Headless Service ไม่มี Selector

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-db-headless
spec:
  clusterIP: None
  ports:
    - port: 3306
  # ไม่มี selector
---
# สร้าง Endpoints เอง
apiVersion: v1
kind: Endpoints
metadata:
  name: external-db-headless
subsets:
  - addresses:
      - ip: 192.168.1.100
      - ip: 192.168.1.101
    ports:
      - port: 3306
```

## Headless Service กับ StatefulSet

นี่คือ Use Case ที่สำคัญที่สุดของ Headless Service

### ตัวอย่างแบบสมบูรณ์: MySQL Cluster

```yaml
# Headless Service สำหรับ StatefulSet
apiVersion: v1
kind: Service
metadata:
  name: mysql
  labels:
    app: mysql
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
    - name: mysql
      port: 3306
---
# StatefulSet
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql  # อ้างอิง Headless Service
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
          name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "rootpassword"
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

**ผลลัพธ์:**

StatefulSet จะสร้าง Pod ที่มี DNS name ที่ stable:
- `mysql-0.mysql.default.svc.cluster.local`
- `mysql-1.mysql.default.svc.cluster.local`
- `mysql-2.mysql.default.svc.cluster.local`

### ตัวอย่างแบบสมบูรณ์: Kafka Cluster

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kafka-headless
  labels:
    app: kafka
spec:
  clusterIP: None
  selector:
    app: kafka
  ports:
    - name: kafka
      port: 9092
    - name: kafka-internal
      port: 9093
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: kafka
spec:
  serviceName: kafka-headless
  replicas: 3
  selector:
    matchLabels:
      app: kafka
  template:
    metadata:
      labels:
        app: kafka
    spec:
      containers:
      - name: kafka
        image: confluentinc/cp-kafka:7.5.0
        ports:
        - containerPort: 9092
          name: kafka
        - containerPort: 9093
          name: kafka-internal
        env:
        - name: KAFKA_BROKER_ID
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: KAFKA_ZOOKEEPER_CONNECT
          value: "zookeeper:2181"
        - name: KAFKA_ADVERTISED_LISTENERS
          value: "PLAINTEXT://$(POD_NAME).kafka-headless:9092"
        volumeMounts:
        - name: data
          mountPath: /var/lib/kafka/data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 20Gi
```

**การเข้าถึง:**
```bash
# เข้าถึง Kafka broker เฉพาะตัว
kafka-console-producer --broker-list \
  kafka-0.kafka-headless:9092,\
  kafka-1.kafka-headless:9092,\
  kafka-2.kafka-headless:9092 \
  --topic my-topic
```

### ตัวอย่างแบบสมบูรณ์: MongoDB Replica Set

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongodb-headless
  labels:
    app: mongodb
spec:
  clusterIP: None
  selector:
    app: mongodb
  ports:
    - port: 27017
      name: mongodb
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongodb
spec:
  serviceName: mongodb-headless
  replicas: 3
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo:7.0
        command:
          - mongod
          - "--replSet"
          - "rs0"
          - "--bind_ip_all"
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          value: "admin"
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: password
        volumeMounts:
        - name: data
          mountPath: /data/db
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
---
# Init Job สำหรับ Configure Replica Set
apiVersion: batch/v1
kind: Job
metadata:
  name: mongodb-init
spec:
  template:
    spec:
      containers:
      - name: mongo-init
        image: mongo:7.0
        command:
          - /bin/sh
          - -c
          - |
            mongosh --host mongodb-0.mongodb-headless --username admin --password $MONGO_PASSWORD --eval '
            rs.initiate({
              _id: "rs0",
              members: [
                { _id: 0, host: "mongodb-0.mongodb-headless:27017" },
                { _id: 1, host: "mongodb-1.mongodb-headless:27017" },
                { _id: 2, host: "mongodb-2.mongodb-headless:27017" }
              ]
            })'
        env:
        - name: MONGO_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mongodb-secret
              key: password
      restartPolicy: Never
```

**การเชื่อมต่อ:**
```bash
# Connection String
mongodb://admin:password@mongodb-0.mongodb-headless:27017,mongodb-1.mongodb-headless:27017,mongodb-2.mongodb-headless:27017/?replicaSet=rs0
```

## DNS Resolution

### การทำงานของ DNS ใน Headless Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
  namespace: production
spec:
  clusterIP: None
  selector:
    app: web
  ports:
    - port: 80
```

**DNS Records ที่ถูกสร้าง:**

1. **Service DNS (A Record)** - ชี้ไปยังทุก Pod IP:
   ```
   web.production.svc.cluster.local
   -> 10.244.0.5, 10.244.0.6, 10.244.0.7
   ```

2. **Pod DNS (A Record)** - แต่ละ Pod (ใช้กับ StatefulSet):
   ```
   web-0.web.production.svc.cluster.local -> 10.244.0.5
   web-1.web.production.svc.cluster.local -> 10.244.0.6
   web-2.web.production.svc.cluster.local -> 10.244.0.7
   ```

3. **SRV Records** - Service discovery:
   ```
   _http._tcp.web.production.svc.cluster.local
   -> web-0.web.production.svc.cluster.local:80
   -> web-1.web.production.svc.cluster.local:80
   -> web-2.web.production.svc.cluster.local:80
   ```

### ทดสอบ DNS Resolution

```yaml
# สร้าง debug pod
apiVersion: v1
kind: Pod
metadata:
  name: dns-test
spec:
  containers:
  - name: dnsutils
    image: tutum/dnsutils
    command:
      - sleep
      - "3600"
```

```bash
# เข้าไปใน Pod
kubectl exec -it dns-test -- /bin/bash

# ทดสอบ DNS lookup
nslookup web.production.svc.cluster.local

# ดู A records ทั้งหมด
dig web.production.svc.cluster.local

# ดู SRV records
dig SRV _http._tcp.web.production.svc.cluster.local

# ทดสอบเชื่อมต่อไปยัง Pod เฉพาะตัว (StatefulSet)
ping web-0.web.production.svc.cluster.local
```

## Use Cases และตัวอย่างการใช้งาน

### 1. Database Clustering

#### PostgreSQL with Patroni

```yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres-headless
spec:
  clusterIP: None
  selector:
    app: postgres
  ports:
    - port: 5432
      name: postgres
    - port: 8008
      name: patroni
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres-headless
  replicas: 3
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:15-alpine
        ports:
        - containerPort: 5432
          name: postgres
        - containerPort: 8008
          name: patroni
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: PATRONI_NAME
          value: "$(POD_NAME)"
        - name: PATRONI_KUBERNETES_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: PATRONI_KUBERNETES_LABELS
          value: "{app: postgres}"
        - name: PATRONI_SCOPE
          value: "postgres-cluster"
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

### 2. Distributed Cache (Redis Cluster)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis-headless
spec:
  clusterIP: None
  selector:
    app: redis
  ports:
    - port: 6379
      name: client
    - port: 16379
      name: gossip
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
spec:
  serviceName: redis-headless
  replicas: 6
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:7-alpine
        command:
          - redis-server
        args:
          - "--cluster-enabled"
          - "yes"
          - "--cluster-config-file"
          - "/data/nodes.conf"
          - "--cluster-node-timeout"
          - "5000"
          - "--appendonly"
          - "yes"
        ports:
        - containerPort: 6379
          name: client
        - containerPort: 16379
          name: gossip
        volumeMounts:
        - name: data
          mountPath: /data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```

**สร้าง Cluster:**
```bash
# รวม nodes เป็น cluster
kubectl exec -it redis-0 -- redis-cli --cluster create \
  redis-0.redis-headless:6379 \
  redis-1.redis-headless:6379 \
  redis-2.redis-headless:6379 \
  redis-3.redis-headless:6379 \
  redis-4.redis-headless:6379 \
  redis-5.redis-headless:6379 \
  --cluster-replicas 1
```

### 3. Elasticsearch Cluster

```yaml
apiVersion: v1
kind: Service
metadata:
  name: elasticsearch-headless
spec:
  clusterIP: None
  selector:
    app: elasticsearch
  ports:
    - port: 9200
      name: http
    - port: 9300
      name: transport
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: elasticsearch
spec:
  serviceName: elasticsearch-headless
  replicas: 3
  selector:
    matchLabels:
      app: elasticsearch
  template:
    metadata:
      labels:
        app: elasticsearch
    spec:
      initContainers:
      - name: increase-vm-max-map
        image: busybox
        command: ["sysctl", "-w", "vm.max_map_count=262144"]
        securityContext:
          privileged: true
      containers:
      - name: elasticsearch
        image: docker.elastic.co/elasticsearch/elasticsearch:8.10.0
        ports:
        - containerPort: 9200
          name: http
        - containerPort: 9300
          name: transport
        env:
        - name: cluster.name
          value: "my-cluster"
        - name: node.name
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: discovery.seed_hosts
          value: "elasticsearch-0.elasticsearch-headless,elasticsearch-1.elasticsearch-headless,elasticsearch-2.elasticsearch-headless"
        - name: cluster.initial_master_nodes
          value: "elasticsearch-0,elasticsearch-1,elasticsearch-2"
        - name: ES_JAVA_OPTS
          value: "-Xms512m -Xmx512m"
        - name: xpack.security.enabled
          value: "false"
        volumeMounts:
        - name: data
          mountPath: /usr/share/elasticsearch/data
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

### 4. Service Mesh / Peer-to-Peer Applications

```yaml
# ตัวอย่าง: Distributed Application ที่ต้องการรู้จัก peers ทั้งหมด
apiVersion: v1
kind: Service
metadata:
  name: distributed-app
spec:
  clusterIP: None
  selector:
    app: distributed-app
  ports:
    - port: 8080
      name: http
    - port: 7946
      name: gossip
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: distributed-app
spec:
  serviceName: distributed-app
  replicas: 5
  selector:
    matchLabels:
      app: distributed-app
  template:
    metadata:
      labels:
        app: distributed-app
    spec:
      containers:
      - name: app
        image: myapp:latest
        ports:
        - containerPort: 8080
          name: http
        - containerPort: 7946
          name: gossip
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: SERVICE_NAME
          value: "distributed-app"
        - name: NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        # Application จะใช้ DNS เพื่อค้นหา peers อื่นๆ
        - name: CLUSTER_MEMBERS
          value: "distributed-app.$(NAMESPACE).svc.cluster.local"
```

**Application Code ตัวอย่าง (Go):**
```go
package main

import (
    "fmt"
    "net"
    "os"
)

func discoverPeers() ([]string, error) {
    serviceName := os.Getenv("CLUSTER_MEMBERS")
    
    // DNS lookup จะได้ IP ของ peers ทั้งหมด
    ips, err := net.LookupIP(serviceName)
    if err != nil {
        return nil, err
    }
    
    var peers []string
    for _, ip := range ips {
        peers = append(peers, ip.String())
    }
    
    return peers, nil
}

func main() {
    peers, _ := discoverPeers()
    fmt.Printf("Found peers: %v\n", peers)
    // เชื่อมต่อกับ peers เพื่อสร้าง cluster
}
```

### 5. Custom Service Discovery

```yaml
# ตัวอย่าง: Microservices ที่ต้องการ custom load balancing
apiVersion: v1
kind: Service
metadata:
  name: api-headless
  labels:
    app: api
spec:
  clusterIP: None
  selector:
    app: api
  ports:
    - port: 8080
      name: http
  # publishNotReadyAddresses: true  # เปิดถ้าต้องการ include Pods ที่ยัง not ready
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
        version: v1
    spec:
      containers:
      - name: api
        image: myapi:v1
        ports:
        - containerPort: 8080
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 10
```

**Client Application:**
```python
import socket
import random
import requests

def get_all_api_endpoints():
    """ดึง IP ของ API pods ทั้งหมดจาก DNS"""
    service_name = "api-headless.default.svc.cluster.local"
    
    # DNS lookup
    _, _, ips = socket.gethostbyname_ex(service_name)
    
    return [f"http://{ip}:8080" for ip in ips]

def custom_load_balance(endpoints, request_data):
    """Custom load balancing logic"""
    # ตัวอย่าง: เลือกตาม hash ของ user_id
    user_id = request_data.get('user_id')
    index = hash(user_id) % len(endpoints)
    return endpoints[index]

# ใช้งาน
endpoints = get_all_api_endpoints()
print(f"Available endpoints: {endpoints}")

request = {'user_id': '12345', 'action': 'get_profile'}
selected_endpoint = custom_load_balance(endpoints, request)
response = requests.post(f"{selected_endpoint}/api", json=request)
```

## publishNotReadyAddresses

เป็น option พิเศษที่ทำให้ Headless Service รวม Pod ที่ยัง not ready ด้วย

```yaml
apiVersion: v1
kind: Service
metadata:
  name: db-cluster
spec:
  clusterIP: None
  publishNotReadyAddresses: true  # เพิ่มบรรทัดนี้
  selector:
    app: database
  ports:
    - port: 5432
```

**Use Cases:**
- Database clustering ที่ต้องการให้ nodes ติดต่อกันตั้งแต่เริ่มต้น
- StatefulSet ที่ Pod ต้องรู้จักกันก่อนจะ ready
- Application ที่มี custom health check logic

**ตัวอย่าง: Cassandra Cluster**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: cassandra
spec:
  clusterIP: None
  publishNotReadyAddresses: true  # สำคัญสำหรับ Cassandra
  selector:
    app: cassandra
  ports:
    - port: 9042
      name: cql
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: cassandra
spec:
  serviceName: cassandra
  replicas: 3
  selector:
    matchLabels:
      app: cassandra
  template:
    metadata:
      labels:
        app: cassandra
    spec:
      containers:
      - name: cassandra
        image: cassandra:4.1
        ports:
        - containerPort: 9042
          name: cql
        env:
        - name: CASSANDRA_SEEDS
          value: "cassandra-0.cassandra,cassandra-1.cassandra"
        - name: CASSANDRA_CLUSTER_NAME
          value: "MyCluster"
        volumeMounts:
        - name: data
          mountPath: /var/lib/cassandra
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

## Debugging และ Troubleshooting

### 1. ตรวจสอบ Service

```bash
# ดูรายละเอียด Service
kubectl get svc my-headless-service -o yaml

# ตรวจสอบว่า clusterIP เป็น None
kubectl get svc my-headless-service -o jsonpath='{.spec.clusterIP}'

# ดู endpoints
kubectl get endpoints my-headless-service
```

### 2. ตรวจสอบ DNS

```bash
# สร้าง debug pod
kubectl run -it --rm debug --image=nicolaka/netshoot --restart=Never -- /bin/bash

# ใน debug pod:
# ดู A records
nslookup my-headless-service.default.svc.cluster.local

# ดูทุก record
dig my-headless-service.default.svc.cluster.local ANY

# ดู SRV records
dig SRV _http._tcp.my-headless-service.default.svc.cluster.local

# ทดสอบ Pod-specific DNS (StatefulSet)
nslookup pod-0.my-headless-service.default.svc.cluster.local
```

### 3. ตรวจสอบ Network Connectivity

```bash
# ทดสอบ ping ไปยัง Pod
kubectl run -it --rm debug --image=busybox --restart=Never -- \
  ping pod-0.my-headless-service.default.svc.cluster.local

# ทดสอบ TCP connection
kubectl run -it --rm debug --image=busybox --restart=Never -- \
  nc -zv pod-0.my-headless-service.default.svc.cluster.local 3306

# ใช้ curl
kubectl run -it --rm debug --image=curlimages/curl --restart=Never -- \
  curl http://pod-0.my-headless-service.default.svc.cluster.local
```

### 4. ดู Logs

```bash
# CoreDNS logs (สำหรับ DNS issues)
kubectl logs -n kube-system -l k8s-app=kube-dns

# Application logs
kubectl logs -f pod-0

# ดู events
kubectl get events --sort-by='.lastTimestamp'
```

### 5. ปัญหาที่พบบ่อย

#### ปัญหา: DNS ไม่ resolve

```bash
# ตรวจสอบว่า Pod มี label ที่ตรงกับ selector
kubectl get pods --show-labels

# ตรวจสอบ endpoints
kubectl describe endpoints my-headless-service

# ถ้าไม่มี endpoints แสดงว่า selector ไม่ match
```

#### ปัญหา: StatefulSet Pods ไม่มี individual DNS

```bash
# ตรวจสอบว่า StatefulSet ใช้ serviceName ที่ถูกต้อง
kubectl get statefulset my-statefulset -o yaml | grep serviceName

# ตรวจสอบว่า Service เป็น Headless (clusterIP: None)
kubectl get svc my-service -o yaml | grep clusterIP
```

#### ปัญหา: Pods ไม่สามารถเชื่อมต่อกันได้

```bash
# ตรวจสอบ Network Policy
kubectl get networkpolicy

# ตรวจสอบว่า Pods อยู่บน node เดียวกันหรือไม่
kubectl get pods -o wide

# ทดสอบจาก Pod หนึ่งไปอีก Pod
kubectl exec -it pod-0 -- ping pod-1.my-service
```

## Best Practices

### 1. ใช้กับ StatefulSet เสมอ

```yaml
# ✅ ดี - มี stable identity
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: database
spec:
  serviceName: database-headless  # อ้างอิง Headless Service

# ❌ ไม่ดี - Deployment กับ Headless Service
# (ใช้ได้แต่ไม่มี stable DNS names)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
```

### 2. ตั้งชื่อ Service ให้สื่อความหมาย

```yaml
# ✅ ดี
metadata:
  name: mysql-headless
  
# ✅ ดี
metadata:
  name: kafka-hs

# ❌ ไม่ดี
metadata:
  name: service1
```

### 3. ใช้ publishNotReadyAddresses เมื่อจำเป็น

```yaml
# ใช้สำหรับ clustered databases
spec:
  clusterIP: None
  publishNotReadyAddresses: true  # Pods ต้องติดต่อกันตั้งแต่เริ่ม
```

### 4. กำหนด Ports อย่างชัดเจน

```yaml
spec:
  ports:
    - name: mysql        # ใส่ชื่อ
      port: 3306
      targetPort: 3306
    - name: metrics
      port: 9104
      targetPort: 9104
```

### 5. ใช้ Readiness Probe

```yaml
# แม้จะเป็น Headless Service ก็ควรมี readiness probe
apiVersion: apps/v1
kind: StatefulSet
spec:
  template:
    spec:
      containers:
      - name: app
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
```

### 6. Resource Limits

```yaml
# กำหนด resources สำหรับ stateful applications
spec:
  template:
    spec:
      containers:
      - name: database
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
```

## เปรียบเทียบ Service Types

| Feature | ClusterIP | NodePort | LoadBalancer | Headless |
|---------|-----------|----------|--------------|----------|
| Cluster IP | ✅ มี | ✅ มี | ✅ มี | ❌ ไม่มี (None) |
| External Access | ❌ | ✅ | ✅ | ❌ |
| Load Balancing | ✅ ใช่ | ✅ ใช่ | ✅ ใช่ | ❌ ไม่มี (DNS round-robin) |
| DNS ชื้ไปยัง | Cluster IP | Cluster IP | External IP | Pod IPs |
| Pod-specific DNS | ❌ | ❌ | ❌ | ✅ (with StatefulSet) |
| Use Case | Internal services | Development/Testing | Production external | StatefulSets, DB clusters |

## สรุป

### Headless Service คืออะไร
- Service ที่ไม่มี Cluster IP (`clusterIP: None`)
- DNS จะชี้ตรงไปยัง Pod IPs
- ไม่มี built-in load balancing

### เมื่อไหร่ควรใช้
- ✅ StatefulSet applications (databases, message queues)
- ✅ ต้องการเชื่อมต่อกับ Pod เฉพาะตัว
- ✅ Clustered applications ที่ต้อง peer discovery
- ✅ Custom load balancing logic
- ❌ Stateless applications ทั่วไป (ใช้ ClusterIP แทน)

### ข้อดี
- Stable network identity สำหรับ StatefulSet Pods
- Fine-grained control ในการเลือก Pod
- เหมาะกับ distributed systems
- Support service discovery แบบ DNS

### ข้อควรระวัง
- ไม่มี automatic load balancing
- Application ต้องจัดการ failover เอง
- ต้องใช้คู่กับ StatefulSet เพื่อให้เกิดประโยชน์สูงสุด

Headless Service เป็นเครื่องมือสำคัญสำหรับการทำ stateful applications บน Kubernetes ช่วยให้ Pod มี identity ที่ชัดเจนและสามารถค้นหากันได้ผ่าน DNS!
