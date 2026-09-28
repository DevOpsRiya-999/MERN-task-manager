# 🚀 MERN Task Manager — Kubernetes Deployment
📌 Project Overview

This project demonstrates the deployment of a MERN Task Manager application using Docker and Kubernetes on an AWS EC2 server.

-The application consists of:

-Frontend — React
-Backend — Node.js / Express
-Database — MongoDB
-Containerization — Docker
-Orchestration — Kubernetes
-Server — AWS EC2
-Configuration — Kubernetes ConfigMaps / Secrets
-Networking — Kubernetes Services
-Reverse Proxy / Web Server — Nginx
=================================================================================================================
# 🏗️ Architecture

                         👤 USER
                           |
                           | HTTP
                           ▼
                  ┌──────────────────┐
                  │    AWS EC2       │
                  │    Server        │
                  └────────┬─────────┘
                           |
                           ▼
                  ┌──────────────────┐
                  │   Kubernetes     │
                  │    Cluster       │
                  └────────┬─────────┘
                           |
             ┌─────────────┴─────────────┐
             │                           │
             ▼                           ▼
   ┌──────────────────┐        ┌──────────────────┐
   │ Frontend Service │        │ Backend Service  │
   │    ClusterIP     │        │    ClusterIP     │
   └────────┬─────────┘        └────────┬─────────┘
            │                           │
            ▼                           ▼
   ┌──────────────────┐        ┌──────────────────┐
   │ React Frontend   │───────▶│ Node.js/Express  │
   │     Pods         │  API   │      Pods        │
   └──────────────────┘        └────────┬─────────┘
                                       │
                                       │ MongoDB
                                       ▼
                              ┌──────────────────┐
                              │     MongoDB      │
                              │      Pod         │
                              └──────────────────┘

                    Kubernetes Namespace
                       task-manager-ns
====================================================================================

# 🔐 Configuration
ConfigMap
   │
   ├── API URL
   ├── Application configuration
   └── Environment configuration

Secret
   │
   ├── Database credentials
   ├── Authentication secrets
   └── Sensitive configuration

====================================================================================
# 🌐 Application Access
``` bash 
kubectl get svc -n task-manager-ns
```

Internet
    │
    ▼
AWS EC2 Public IP
    │
    ▼
Kubernetes Service
    │
    ▼
Frontend Pod
    │
    ▼
Backend Service
    │
    ▼
Backend Pod
    │
    ▼
MongoDB
   
=========================================================================================
# 📁 Kubernetes Project Structure
MERN-task-manager/
│
├── frontend/
│   ├── Dockerfile
│   └── ...
│
├── backend/
│   ├── Dockerfile
│   └── ...
│
├── k8s/
│   ├── namespace.yaml
│   ├── frontend-deployment.yaml
│   ├── frontend-service.yaml
│   ├── backend-deployment.yaml
│   ├── backend-service.yaml
│   ├── mongodb-deployment.yaml
│   ├── mongodb-service.yaml
│   ├── configmap.yaml
│   └── secret.yaml
│
├── docker-compose.yml
└── README.md
===================================================================================================
# ⭐ Project Summary
Successfully deployed a containerized MERN Task Manager application on AWS EC2 using Kubernetes. The project includes separate frontend, backend, and MongoDB components,
with Kubernetes Deployments, Services, ConfigMaps, and Secrets used to manage the application. This project helped me gain practical experience with container orchestration,
service networking, configuration management, and troubleshooting in a real deployment environment.
   

                       
