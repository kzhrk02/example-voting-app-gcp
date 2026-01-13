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

The solution was deployed on **Google Cloud Platform (GCP)**.

GCP was selected to demonstrate cross-cloud deployment skills and to use a fully managed Kubernetes service that simplifies cluster operations.

### Cloud Services Used

- **Google Kubernetes Engine (GKE Autopilot)** – Managed Kubernetes cluster
- **Google Cloud Load Balancer** – Automatically created by Kubernetes Services
- **Google Cloud IAM** – Authentication and access control


## 4. Infrastructure as Code (Terraform)

The cloud infrastructure was provisioned using **Terraform**.

Terraform was chosen to ensure that the infrastructure is:
- Reproducible
- Version-controlled
- Created without manual configuration

All Terraform files are located in the `terraform/` directory.


Terraform provisions the following main component:

- **Google Kubernetes Engine (GKE) Autopilot cluster**

Networking, control plane, and node management are handled automatically by the managed service.

The entire infrastructure can be created using a single command:

```bash
tterraform apply -var="project_id=voting-app-gke-123"
```

### Kubernetes Deployment Model

A managed Kubernetes cluster (GKE Autopilot) was selected because it:
- Removes the need to manage worker nodes
- Automatically handles scaling and resource allocation
- Allows focus on application deployment rather than infrastructure maintenance

### Key Terraform Resource

The core infrastructure is defined using the following Terraform resource:

```hcl
resource "google_container_cluster" "autopilot" {
  enable_autopilot = true
}
```
## 5. Database Deployment

The application uses two backend databases:

- **PostgreSQL** – persistent storage for voting results
- **Redis** – temporary in-memory storage for votes

Both databases are deployed as **Kubernetes pods** within the same cluster as the application.

This approach was chosen because:
- It avoids additional cloud services for this project
- It simplifies deployment and teardown
- It is sufficient for demonstration and learning purposes

### Kubernetes Configuration

Database deployments are defined using Kubernetes manifests located in the `k8s-specifications/` directory:

- `db-deployment.yaml` and `db-service.yaml`
- `redis-deployment.yaml` and `redis-service.yaml`

These files define how the database containers are started and accessed by other services.


## 6. Application Deployment

After the Kubernetes cluster was provisioned, the application was deployed using **Kubernetes manifests**.

All deployment and service definitions are located in the `k8s-specifications/` directory. The following components are deployed to the cluster:

- Application services: vote, result, worker
- Backend services: redis, postgres

Each component is deployed as a Kubernetes **Deployment** and exposed using a **Service** where required.

The application can be deployed using:

```bash
kubectl apply -f k8s-specifications/
```

### Service Exposure
The frontend services (vote and result) are exposed using Kubernetes **LoadBalancer** services.

This allows external access to the application through public IP addresses automatically provisioned by GCP. (important for GKE Autopilot)

Application status can be verified using:

```bash
kubectl get pods
kubectl get services
```

## 7. CI/CD Pipeline

The repository contains existing CI workflows provided by the original application authors.
These workflows focus on building Docker images for application components.

For this project, an additional CI/CD process was used to automate **deployment to the Kubernetes cluster**.

### Existing CI Workflows

The following workflows are part of the original repository:
- Docker image build workflows for vote, result, and worker services

These workflows were not modified and are outside the scope of this project.


## 8. Demo and Cleanup

Verify that the application is running in Kubernetes:

   ```bash
   kubectl get pods
   kubectl get services
   ```

All infrastructure can be removed using Terraform:

```bash
cd terraform
terraform destroy -var="project_id=<GCP_PROJECT_ID>"
```
