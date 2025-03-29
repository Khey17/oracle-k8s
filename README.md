# Oracle + Kubernetes + Docker Project (Guide)

This project shows how to run Oracle Database inside Kubernetes using Docker and connect to it like a normal Oracle database. You can upload SQL files, create users, and everything stays running even if you restart. No need to recreate it all every time.

---

## 🧰 What You Need Installed

- **Docker Desktop** (with WSL2 backend enabled)
- **Minikube** (Docker driver)
- **kubectl**
- **Oracle Account** (to pull the Oracle XE image)
- **PowerShell (Admin)**

---

## 📁 Project Folder Structure

```bash
/oracle-k8s
  ├── PS1.sql   # Schema creation
  ├── PS2.sql   # Data + constraints (optional)
  └── oracle-deployment.yaml
```

---

## 🐳 Step 1: Pull the Oracle XE Docker Image

```powershell
docker login container-registry.oracle.com
# (use your Oracle credentials)

docker pull container-registry.oracle.com/database/express:21.3.0-xe
```

---

## 🧠 Step 2: Start Minikube with Docker Driver

```powershell
minikube start --driver=docker
minikube image load container-registry.oracle.com/database/express:21.3.0-xe
```

This will start the Kubernetes cluster and load the Oracle image.

---

## 📦 Step 3: Deploy Oracle in Kubernetes

Make sure your `oracle-deployment.yaml` file is ready. Then:

```powershell
kubectl apply -f oracle-deployment.yaml
kubectl get pods -w
```

Wait until the pod shows `Running`.

---

## 👤 Step 4: Create a Custom User (Example: C##UD_ASH)

```powershell
kubectl exec deploy/oracle-db -- bash -c "echo -e 'CREATE USER C##UD_ASH IDENTIFIED BY UdAsh2025 CONTAINER=ALL;\nGRANT CONNECT, RESOURCE TO C##UD_ASH CONTAINER=ALL;' | /opt/oracle/product/21c/dbhomeXE/bin/sqlplus sys/Oracle2025@localhost:1521/XE as sysdba"
```

---

## 📂 Step 5: Upload Your SQL File (e.g. PS1.sql)

```powershell
kubectl cp "C:/path/to/oracle-k8s/PS1.sql" <your-pod-name>:/home/oracle/PS1.sql
```

To get your pod name:
```powershell
kubectl get pods
```

---

## ▶️ Step 6: Run SQL File as Your Custom User

```powershell
kubectl exec <your-pod-name> -- \
  bash -c "/opt/oracle/product/21c/dbhomeXE/bin/sqlplus C##UD_ASH/UdAsh2025@localhost:1521/XE @/home/oracle/PS1.sql"
```

---

## 🔍 Step 7: Check If Tables Were Created

```powershell
kubectl exec <your-pod-name> -- \
  bash -c "/opt/oracle/product/21c/dbhomeXE/bin/sqlplus -s C##UD_ASH/UdAsh2025@localhost:1521/XE <<< 'SELECT table_name FROM user_tables;'"
```

---

## 🧠 SQL+ Interactive Mode (Just Like SQL Developer)

```powershell
kubectl exec -it <your-pod-name> -- bash

# Inside the pod:
/opt/oracle/product/21c/dbhomeXE/bin/sqlplus C##UD_ASH/UdAsh2025@localhost:1521/XE
```

Then you can run whatever SQL you want.

---

## 🧾 Info You’ll Want to Remember

| Thing              | Value Example                           |
|-------------------|------------------------------------------|
| Pod Name          | `oracle-db-5469757c8-kht7c`              |
| DB Name           | `XE`                                     |
| Custom Username   | `C##UD_ASH`                              |
| Password          | `UdAsh2025`                              |
| Deployment Name   | `oracle-db`                              |
| SQL File Path     | `/home/oracle/PS1.sql` inside the pod    |

---

## 🧯 If You Restart Your PC

```powershell
minikube start
kubectl get pods
```

If the pod isn’t running, use:
```powershell
kubectl rollout restart deployment oracle-db
```

Then reconnect using the same commands.

---

## 🚫 Do You Need to Docker Login Again?
Only if you:
- Wipe Docker
- Delete Docker config
- Pull the Oracle image again

Else: **you’re good** 👍

---

That’s it! You now have a real Oracle XE instance inside Kubernetes with your own schema, tables, and SQL files. No Oracle SQL Developer needed. You’re a wizard 🧙‍♂️

