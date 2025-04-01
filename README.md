# Oracle + Kubernetes + Docker Project (Guide)

This project shows how to run Oracle Database inside Kubernetes using Docker and connect to it like a normal Oracle database. You can upload SQL files, create users, and everything stays running even if you restart. No need to recreate it all every time.

---

## Requirements/Dependancies

- **Docker Desktop** (with WSL2 backend enabled)
- **Minikube** (Docker driver)
- **kubectl**
- **Oracle Account** (to pull the Oracle XE image)
- **PowerShell (Admin)**

---

## Project Folder Structure

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

Make sure the `oracle-deployment.yaml` file is ready. Then:

```powershell
kubectl apply -f oracle-deployment.yaml
kubectl get pods -w
```

Wait until the pod shows `Running`.

---

## Connect to the Registered Oracle Database remotely 

```powershell
kubectl exec -it oracle-db-XXXXX (podname) -- bash
```

---

## Inside Bash

```bash
sqlplus Username/YourPassword@IPAddress/ServiceName (for example, FREEPDB1 in our case)
```

---

## You should see the following snippet

```sql
SQL*Plus: Release 21.0.0.0.0 - Production on Tue Apr 1 21:49:12 2025
Version 21.3.0.0.0

Copyright (c) 1982, 2021, Oracle.  All rights reserved.

Last Successful login time: Sat Mar 22 2025 14:42:26 +00:00

Connected to:
Oracle Database 23ai Free Release 23.0.0.0.0 - Production
Version 23.4.0.24.05
SQL>
```

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
``` Powershell
 bash-4.2$ sqlplus UD_ASHOKK/XXXX@98.84.13.6:1521/FREEPDB1

SQL*Plus: Release 21.0.0.0.0 - Production on Tue Apr 1 21:49:12 2025
Version 21.3.0.0.0

Copyright (c) 1982, 2021, Oracle.  All rights reserved.

Last Successful login time: Sat Mar 22 2025 14:42:26 +00:00

Connected to:
Oracle Database 23ai Free Release 23.0.0.0.0 - Production
Version 23.4.0.24.05
```
---


