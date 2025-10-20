# Task 2 – Kubernetes Deployment and Pod Creation

## Objective
Use the application created in **Task #1**, containerize it with Docker, deploy it on Kubernetes along with MongoDB, and modify the `PUT TaskExecution` endpoint to dynamically create and execute commands in the new pod that is created.

---

## Overview

In this task, we:
- Created **Dockerfile** for the Spring Boot application with MongoDB.
- Built and pushed Docker images to **Docker Hub**.
- Created **Kubernetes manifests** for:
  - Application Deployment & Service
  - MongoDB Deployment, Service, Persistent Volume and RBAC for creation of Pod
- Deployed images of Task application and MongoDB on **Docker Desktop (Kubernetes Cluster)**.
- Modified the `PUT TaskExecution` API endpoint to programmatically create a **Kubernetes pod** (using Kubernetes Java Client) and execute command inside the pod.

---

## Architecture

```
+-----------------------+
|   Host Machine        |
|   (Docker Desktop)    |
|                       |
|  +-----------------+  +-----------------+
|  |  MongoDB Pod    |  |  App Pod        |
|  | (with PVC)      |  | (Spring Boot)   |
|  |                 |  |                 |
|  +--------+--------+  +--------+--------+
|           |                    |
|     Cluster Service         NodePort Service
|           |                    |
|     Persistent Volume       Access via Host:NodePort
+-----------------------------------------------+
```

---

## Technologies Used

- **Spring Boot** (Java)
- **MongoDB**
- **Docker**
- **Kubernetes (Docker Desktop)**


---

## Docker Setup

### 1. Dockerfile 
```dockerfile
FROM openjdk:17-jdk-slim
WORKDIR /app
COPY target/poc-1.0.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### 2. Build and Push Docker Image
```bash
# Build the image
docker build -t arjunamar5/poc-app-task:latest .

# Push to Docker Hub
docker push arjunamar5/poc-app-task:latest
```

---

## Kubernetes Setup
### Used Kubernetes deployments and services to create the pods and expose the services following are the commands/files:
```bat
kubectl apply -f mongo-pv.yaml
kubectl apply -f mongo-pvc.yaml
kubectl apply -f mongo-deployment.yaml
kubectl apply -f mongo-service.yaml
kubectl apply -f rbac.yaml
kubectl apply -f app-deployment.yaml
kubectl apply -f app-service.yaml

kubectl get all
```


---

## ⚡ Commands Used

```bash
# Check running pods
kubectl get pods

# Check services
kubectl get svc

# Describe pods for debug info, check pod logs and logging into the pod
kubectl describe pod <pod-name>
kubectl logs -f <pod-name>
kubectl exec -it <pod-name> -- <command>


```

---

## 🧾 Verification

### ✅ Check pods
```bash
kubectl get pods
```

### ✅ Application and MongoDB pods running
```
NAME                                     READY     STATUS    RESTARTS      AGE
pod/command-pod-6df07a63                  0/1     Completed    0          139m
pod/mongodb-6bd45f4756-nnqpr              1/1     Running      0          141m
pod/poc-app-deployment-69f74f494d-clp6w   1/1     Running      0          141m
```

### ✅ Access application using GET 
```bash
http://localhost:30081/api/tasks
```
---

## 📸 Screenshots to Include

1. Output of `kubectl get all`  #includes the details of pods, services, deployments and replicas

![Get-all](https://github.com/user-attachments/assets/9e44d112-53c4-4800-b016-2964e68136bd)
--- 

2. Output of New pod that is created as part of command execution

![new pod](https://github.com/user-attachments/assets/ea39fca7-9e2a-47d9-8064-01727c99c0a3)


---

3. Execution of command using Postman using http://localhost:30081/api/tasks/130/executions (POST request)

![execution-of-cmd](https://github.com/user-attachments/assets/93ae157d-d8c2-4114-aa3f-c7359b9fc06d)


---
4. Execution output of the command shown inside the busy box pod

<img width="611" height="53" alt="image" src="https://github.com/user-attachments/assets/1c42c9bf-920b-42d1-b6e4-f257998d8219" />


## Summary

- Application and MongoDB deployed in **separate pods**
- MongoDB uses **PVC** for persistence
- Environment variable `MONGO_URI` used in app deployment
- Application accessible from host machine via **NodePort**
- PUT endpoint spawns a **new BusyBox pod** dynamically

