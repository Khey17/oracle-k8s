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

## Pull the Oracle XE Docker Image

```powershell
docker login container-registry.oracle.com
# (use your Oracle credentials)

docker pull container-registry.oracle.com/database/express:21.3.0-xe
```

---

## Start Minikube with Docker Driver

```powershell
minikube start --driver=docker
minikube image load container-registry.oracle.com/database/express:21.3.0-xe
```

This will start the Kubernetes cluster and load the Oracle image.

---

## Deploy Oracle in Kubernetes

Make sure your `oracle-deployment.yaml` file is ready. Then:

```powershell
kubectl apply -f oracle-deployment.yaml
kubectl get pods -w
```

Wait until the pod shows `Running`.

---

## Create a Custom User (Example: C##UD_ASH)

```powershell
kubectl exec deploy/oracle-db -- bash -c "echo -e 'CREATE USER C##UD_ASH IDENTIFIED BY UdAsh2025 CONTAINER=ALL;\nGRANT CONNECT, RESOURCE TO C##UD_ASH CONTAINER=ALL;' | /opt/oracle/product/21c/dbhomeXE/bin/sqlplus sys/Oracle2025@localhost:1521/XE as sysdba"
```

---

## Upload Your SQL File (e.g. PS1.sql)

```powershell
kubectl cp "C:/path/to/oracle-k8s/PS1.sql" <your-pod-name>:/home/oracle/PS1.sql
```

To get your pod name:
```powershell
kubectl get pods
```

---

## Run SQL File as Your Custom User

```powershell
kubectl exec <your-pod-name> -- \
  bash -c "/opt/oracle/product/21c/dbhomeXE/bin/sqlplus C##UD_ASH/UdAsh2025@localhost:1521/XE @/home/oracle/PS1.sql"
```

---

## Check If Tables Were Created

```powershell
kubectl exec <your-pod-name> -- \
  bash -c "/opt/oracle/product/21c/dbhomeXE/bin/sqlplus -s C##UD_ASH/UdAsh2025@localhost:1521/XE <<< 'SELECT table_name FROM user_tables;'"
```

---

## SQL + Interactive Mode (Just Like SQL Developer)

```powershell
kubectl exec -it <your-pod-name> -- bash

# Inside the pod:
/opt/oracle/product/21c/dbhomeXE/bin/sqlplus C##UD_ASH/UdAsh2025@localhost:1521/XE
```

Then you can run whatever SQL you want.

---

## Things to Remember

| Thing              | Value Example                           |
|-------------------|------------------------------------------|
| Pod Name          | `oracle-db-5469757c8-kht7c`              |
| DB Name           | `XE`                                     |
| Custom Username   | `C##UD_ASH`                              |
| Password          | `UdAsh2025`                              |
| Deployment Name   | `oracle-db`                              |
| SQL File Path     | `/home/oracle/PS1.sql` inside the pod    |

---

## If You Restart Your PC

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

 # Progress

 kubectl exec oracle-db-5469757c8-kht7c -- bash -c "/opt/oracle/product/21c/dbhomeXE/bin/sqlplus C##UD_ASH/UdAsh2025@localhost:1521/XE @/home/oracle/PS2.sql"
>>

SQL*Plus: Release 21.0.0.0.0 - Production on Sat Mar 29 21:09:01 2025
Version 21.3.0.0.0

Copyright (c) 1982, 2021, Oracle.  All rights reserved.

Last Successful login time: Sat Mar 29 2025 20:38:38 +00:00

Connected to:
Oracle Database 21c Express Edition Release 21.0.0.0.0 - Production
Version 21.3.0.0.0


MSG
--------------------------------------------------------------------------------
Begin Executing 01@ /home/oracle/PS2.sql


Procedure created.


Procedure created.


Procedure created.

BEGIN
*
ERROR at line 1:
ORA-01031: insufficient privileges
ORA-06512: at "C##UD_ASH.PRC_CREATE_TRG01_TRIGGERS", line 19
ORA-06512: at "C##UD_ASH.PRC_CREATE_TRIGGERS", line 27
ORA-06512: at "C##UD_ASH.PRC_CREATE_TRIGGERS", line 27
ORA-06512: at line 2


Disconnected from Oracle Database 21c Express Edition Release 21.0.0.0.0 - Production
Version 21.3.0.0.0
command terminated with exit code 7
 
---


