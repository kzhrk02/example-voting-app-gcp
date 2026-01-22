# Cloud-Native Deployment of Example Voting App (GCP)

This project demonstrates a cloud-native deployment of the Example Voting App using Google Cloud Platform (GCP), Terraform, Kubernetes (GKE Autopilot), and GitHub Actions CI/CD.

## Application Architecture

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

## 1. Cloud Infrastructure Architecture

### Components
- **Cloud Provider:** Google Cloud Platform (GCP)
- **Region:** europe-west1
- **Container Orchestration:** Google Kubernetes Engine (GKE Autopilot)
- **Container Registry:** Google Artifact Registry
- **Infrastructure as Code:** Terraform
- **CI/CD:** GitHub Actions

The solution was deployed on **Google Cloud Platform (GCP)**.

## 2. Infrastructure as Code (Terraform)
The infrastructure is defined in the `terraform/` directory and is fully provisioned using Terraform.

### Managed Resources
Terraform provisions the following resources:

- **GKE Autopilot Cluster**
  - Name: `voting-app-autopilot`
  - Region: `europe-west1`
  - Fully managed Kubernetes control plane and nodes

Terraform configuration files:
- `main.tf` – GKE Autopilot cluster definition
- `variables.tf` – Project and region variables
- `outputs.tf` – Cluster name and location outputs
- `versions.tf` – Terraform and provider versions

The entire infrastructure can be created using a single command:

```bash
tterraform apply -var="project_id=voting-app-gke-123"
```

### Authentication
Terraform authenticates to GCP using **Application Default Credentials (ADC)** provided by the `gcloud` CLI.
No service account keys are stored in the repository.

### Validation
Infrastructure state is validated using:
```bash
terraform init
terraform plan
```

## 3. CI/CD Pipeline (GitHub Actions)
The CI/CD pipeline is implemented using **GitHub Actions** and defined in `.github/workflows/ci-cd.yml`
The pipeline automates application build and deployment while validating infrastructure state.

### Workflow Steps
- **Trigger**  
The pipeline is triggered automatically on:

```yaml
push:
  branches:
    - main
```
Every push to the main branch starts the pipeline without manual intervention.

- **Authentication**  
The pipeline authenticates to GCP using a dedicated service account, performed via:
  - A service account key stored as a GitHub secret: `GCP_PROJECT_ID`, `GCP_SA_KEY`
  - `google-github-actions/auth` action
- **Infrastructure Check**  
  Terraform commands (`init`, `validate`, `plan`) are executed to verify that the existing infrastructure matches the configuration.
- **Docker Build**  
  A Docker image is built for the `vote` service from the `vote/` directory.
- **Docker Push**  
  The image is pushed to Google Artifact Registry using the Git commit SHA as the image tag.
- **Kubernetes Deployment**  
  The Kubernetes deployment manifest is updated with the new image tag and applied using `kubectl apply`.
- **Application Update**  
  Kubernetes performs a rolling update and deploys the new version automatically.

## Deployment Instructions

### Prerequisites
- Google Cloud account with billing enabled
- `gcloud`, `kubectl`, and `terraform` installed
- GCP project created
- GitHub repository secrets configured for CI/CD

The frontend services (vote and result) are exposed using Kubernetes **LoadBalancer** services.
This allows external access to the application through public IP addresses automatically provisioned by GCP. (important for GKE Autopilot)

**Provision Infrastructure**
```bash
cd terraform
terraform init
terraform apply
```

**Verify that the application is running in Kubernetes:**
   ```bash
   kubectl get pods
   kubectl get services
   ```

**All infrastructure can be removed using Terraform:**
```bash
cd terraform
terraform destroy -var="project_id=<GCP_PROJECT_ID>"
```
