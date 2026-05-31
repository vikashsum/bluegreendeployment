Blue-Green Deployment on AWS EKS using Jenkins
🚀 Project Overview

This project demonstrates a CI/CD pipeline using Jenkins to deploy a containerized application on an AWS EKS Kubernetes cluster using a Blue-Green deployment strategy.

It includes:

Dockerized Python application
Jenkins pipeline for CI/CD
Amazon EKS cluster provisioning
Kubernetes Blue-Green deployment
Automatic image build and push to Docker Hub
Traffic switching using Kubernetes Service
🏗️ Architecture
Developer pushes code to GitHub
Jenkins pipeline triggers automatically
Docker image is built
Image is pushed to Docker Hub
EKS cluster is created / validated
Green deployment is applied
Service switches traffic from Blue → Green
📁 Project Structure
BlueGreen-Deployment/
│
├── app/
│   ├── Dockerfile
│   ├── app.py
│   ├── requirements.txt
│
├── k8s/
│   ├── namespace.yaml
│   ├── blue/
│   ├── green/
│
├── Jenkinsfile
└── README.md
⚙️ Prerequisites

Make sure you have:

AWS Account
EKS IAM permissions
Jenkins installed
Docker installed on Jenkins server
kubectl installed
eksctl installed
AWS CLI configured
DockerHub account
🔐 Jenkins Credentials Required

Add these in Jenkins → Manage Credentials:

Credential ID	Type	Purpose
dockerhub-creds	Username/Password	DockerHub login
aws-creds	AWS credentials	EKS access
🐳 Build & Run Locally
Build Docker image
docker build -t sample-app .
Run container
docker run -p 8080:8080 sample-app

Test:

curl http://localhost:8080/
curl http://localhost:8080/health
☸️ Kubernetes Deployment
Apply namespace
kubectl apply -f k8s/namespace.yaml
Deploy Blue version
kubectl apply -f k8s/blue/
Deploy Green version
kubectl apply -f k8s/green/
🔄 Blue-Green Switching

Traffic switching is done by updating the Kubernetes Service:

Blue active
selector:
  version: blue
Switch to Green
selector:
  version: green

Apply:

kubectl apply -f k8s/green/service.yaml
🚀 Jenkins Pipeline Stages
Checkout code
Build Docker image
Push to Docker Hub
Create/verify EKS cluster
Configure kubectl
Deploy Green version
Validate deployment
Switch traffic (Blue → Green)
📊 Health Checks

The application exposes:

/ → Main endpoint
/health → Readiness/Liveness probe
🧠 Key Concepts Used
CI/CD with Jenkins
Containerization with Docker
Kubernetes Deployments
Blue-Green Deployment Strategy
AWS EKS Managed Kubernetes
⚠️ Common Issues
Pods stuck in Pending
Node group not scaled properly
Insufficient CPU/memory
Deployment timeout
Missing /health endpoint
Missing readinessProbe
Node group issues
eksctl scale nodegroup ...
🎯 Author

DevOps Learning Project – Jenkins + EKS + Docker + Kubernetes
