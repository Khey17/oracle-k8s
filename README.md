# 🐘 Oracle XE + Docker + Kubernetes Deployment (Step-by-Step Guide)

This guide walks you through deploying Oracle XE locally using **Docker**, and then deploying the same setup into **Kubernetes using Minikube**, with a clean custom schema (`psuser`) and SQL script (`PS1.sql`).

---

## 📦 Prerequisites

- Docker installed and running
- Minikube installed (with Docker driver)
- kubectl installed
- Oracle account to access Oracle Container Registry
- PowerShell (Admin)

---

## 🚀 Phase 1: Oracle XE in Docker (Local Setup)

### ✅ Step 1: Login to Oracle Container Registry
```powershell
docker login container-registry.oracle.com
```
Enter your Oracle SSO username + password.

### ✅ Step 2: Pull the Oracle XE Docker Image
```powershell
docker pull container-registry.oracle.com/database/express:21.3.0-xe
```

### ✅ Step 3: Start Oracle XE Container
```powershell
docker run -d --name oracle-xe ^
  -p 1521:1521 -p 5500:5500 ^
  -e ORACLE_PWD=Oracle2025 ^
  -v oracle-data:/opt/oracle/oradata ^
  container-registry.oracle.com/database/express:21.3.0-xe
```

### ✅ Step 4: Wait for DB Initialization
```powershell
docker logs -f oracle-xe
```
Wait for:
```
DATABASE IS READY TO USE!
```

### ✅ Step 5: Create a Custom User
```powershell
docker exec -it oracle-xe bash
```
Then inside the container:
```bash
sqlplus sys/Oracle2025@localhost:1521/XEPDB1 as sysdba
```
```sql
CREATE USER psuser IDENTIFIED BY PsUser2025;
GRANT CONNECT, RESOURCE TO psuser;
```

### ✅ Step 6: Upload and Run PS1.sql
```powershell
docker cp "C:\Users\Karth\PS\PS1.sql" oracle-xe:/home/oracle/PS1.sql
```
Back in the container:
```bash
sqlplus psuser/PsUser2025@localhost:1521/XEPDB1
```
```sql
@/home/oracle/PS1.sql
SELECT table_name FROM user_tables;
```

---

## ☸️ Phase 2: Oracle XE in Kubernetes (Minikube)

### ✅ Step 1: Start Minikube with Docker Driver
```powershell
minikube start --driver=docker --memory=6000 --cpus=2
```

### ✅ Step 2: Prepare YAML Deployment File
Create `oracle-deployment.yaml`:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: oracle-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: oracle-db
spec:
  replicas: 1
  selector:
    matchLabels:
      app: oracle
  template:
    metadata:
      labels:
        app: oracle
    spec:
      containers:
        - name: oracle
          image: container-registry.oracle.com/database/express:21.3.0-xe
          ports:
            - containerPort: 1521
            - containerPort: 5500
          env:
            - name: ORACLE_PWD
              value: "Oracle2025"
          volumeMounts:
            - mountPath: /opt/oracle/oradata
              name: oracle-storage
      volumes:
        - name: oracle-storage
          persistentVolumeClaim:
            claimName: oracle-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: oracle-service
spec:
  type: NodePort
  selector:
    app: oracle
  ports:
    - name: db-port
      port: 1521
      targetPort: 1521
      nodePort: 31521
    - name: web-port
      port: 5500
      targetPort: 5500
      nodePort: 32500
```

### ✅ Step 3: Load Oracle Image into Minikube
```powershell
minikube image load container-registry.oracle.com/database/express:21.3.0-xe
```

> ⏳ This may take 10–20 mins. It silently transfers ~7GB.

### ✅ Step 4: Deploy to Kubernetes
```powershell
kubectl apply -f oracle-deployment.yaml
```

### ✅ Step 5: Monitor Pod
```powershell
kubectl get pods -w
```
Wait for:
```
STATUS: Running
READY: 1/1
```

---

## 🧑‍💻 Inside Kubernetes: Setup Schema

### ✅ Step 6: Exec Into the Pod
```powershell
kubectl exec -it <pod-name> -- bash
```

### ✅ Step 7: Create `psuser`
```bash
sqlplus sys/Oracle2025@localhost:1521/XEPDB1 as sysdba
```
```sql
CREATE USER psuser IDENTIFIED BY PsUser2025;
GRANT CONNECT, RESOURCE TO psuser;
```

### ✅ Step 8: Copy and Run `PS1.sql`
```powershell
kubectl cp "C:\Users\Karth\PS\PS1.sql" <pod-name>:/home/oracle/PS1.sql
```
```bash
sqlplus psuser/PsUser2025@localhost:1521/XEPDB1
@/home/oracle/PS1.sql
```

### ✅ Step 9: Verify
```sql
SELECT table_name FROM user_tables;
```

---

## 🎉 You Did It!

You now have:
- Oracle XE in Docker ✅
- Oracle XE in Kubernetes ✅
- A clean schema (`psuser`) for your class assignments ✅
- Fully documented, repeatable setup ✅

Use this guide as your README for GitHub or any future projects.

---

Feel free to automate later using Helm or CI/CD. For now — this is a rock-solid manual foundation. 🚀

