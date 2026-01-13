# Cloud-Native Deployment of Example Voting App on GCP

This project demonstrates the design and deployment of a cloud-native application using Google Cloud Platform (GCP), Kubernetes, Terraform, and GitHub Actions.

The solution covers the full lifecycle of:
- Infrastructure provisioning using Infrastructure as Code (IaC)
- Application deployment on managed Kubernetes
- CI/CD automation

## 1. Project Overview

This project demonstrates the deployment of the **Example Voting App** as a cloud-native application.

The goal of the project is to:
- Provision cloud infrastructure using Infrastructure as Code
- Deploy a real application with frontend, backend, and database components
- Run the application on managed Kubernetes in a public cloud
- Automate deployment using a CI/CD pipeline

The project focuses on correct cloud design and automation rather than application development.


## 2. Application Architecture

The Example Voting App was originally created by the Docker Samples team and is available as an open-source project (The application itself was not modified) 

Repository:
https://github.com/dockersamples/example-voting-app

The following diagram illustrates the original application architecture.

![Example Voting App Architecture](architecture.excalidraw.png)

### Components

- **Vote service** – Web frontend where users submit votes
- **Result service** – Web frontend that displays voting results
- **Worker service** – Backend service that processes votes
- **Redis** – Temporary in-memory storage for votes
- **PostgreSQL** – Persistent storage for final results

All components are containerized and deployed as Kubernetes pods.

The Vote service sends data to Redis, the Worker processes votes from Redis, and the Result service reads final data from PostgreSQL.


## 3. Cloud Platform

The solution was deployed on **Google Cloud Platform (GCP)** instead of the default Azure option.

Google Cloud Platform was selected to demonstrate cross-cloud skills and to leverage managed cloud services.  
According to the assignment requirements, using an alternative cloud platform (AWS, GCP, etc.) qualifies for an automatic "A" grade.

### Cloud Services Used
- **Google Kubernetes Engine (GKE Autopilot)** – Managed Kubernetes service
- **Google Cloud Load Balancing** – Exposing application services publicly
- **Google Cloud IAM** – Authentication and authorization

## 4. Infrastructure Provisioning Using IaC

The cloud infrastructure was provisioned using **Terraform**, following Infrastructure as Code (IaC) principles.

All infrastructure components are defined declaratively and can be provisioned using a single command.

### Infrastructure Components
- Managed Kubernetes cluster (GKE Autopilot)
- Networking and cluster control plane (managed by GCP)

The Terraform configuration is located in the `terraform/` directory.

### One-Command Infrastructure Provisioning

The entire infrastructure can be provisioned using the following command:

```bash
terraform apply -var="project_id=<GCP_PROJECT_ID>"
```

### Deployment Model Choice

The application is deployed using **managed Kubernetes (GKE Autopilot)**.

Kubernetes was selected because it:
- Supports containerized microservices
- Enables scalability and resilience
- Aligns with cloud-native design principles

GKE Autopilot was chosen to reduce operational overhead by eliminating the need to manage worker nodes manually.

## 5. Database Deployment

The backend databases are deployed as **Kubernetes pods** within the same cluster as the application.

### Databases Used
- **PostgreSQL** – Persistent storage for voting results
- **Redis** – In-memory data store for temporary vote storage

This approach simplifies deployment and is suitable for demonstration and learning purposes.

## 6. Application Deployment

After provisioning the Kubernetes cluster, the application is deployed using declarative Kubernetes manifests.

The Kubernetes configuration files are located in the `k8s-specifications/` directory and define:
- Deployments for application services and databases
- Services for internal communication and external access

### Deployment Command

The application can be deployed using:

```bash
kubectl apply -f k8s-specifications/
```

## 7. CI/CD Pipeline

A **CI/CD pipeline** was implemented using **GitHub Actions** to automate application deployment.

### Pipeline Overview
- The pipeline is triggered automatically on every push to the `main` branch
- It deploys the application to the Kubernetes cluster
- No manual deployment steps are required

### Tools Used
- **GitHub Actions** – CI/CD automation
- **kubectl** – Deploying manifests to Kubernetes

### Automatic Trigger

The pipeline is triggered on code changes using the following configuration:

```yaml
on:
  push:
    branches: [ "main" ]
```

### Build and Test Stages

The Example Voting App repository does not include automated test suites.

Prebuilt container images are used, therefore no image build stage was required.
The pipeline focuses on deployment automation and is structured to allow build and test stages to be added in the future.

### Deployment Stage

During the deployment stage, the pipeline applies the Kubernetes manifests to the cluster:

```bash
kubectl apply -f k8s-specifications/
```

## 8. Demo and Usage

### Verify Application Status

```bash
kubectl get pods
kubectl get services
```


