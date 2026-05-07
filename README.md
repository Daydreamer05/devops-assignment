# DevOps Assignment: EKS, Hello World, and Monitoring

This project sets up an Amazon EKS cluster using Terraform, deploys a "Hello World" microservice using a custom Helm chart, and automatically installs Prometheus and Grafana for monitoring.

> 🤖 **Note:** This project and its files were created with the assistance of AI.

## ☁️ Terraform State Management
> **Note:** This project is configured to use a remote S3 backend for Terraform state.
> - **S3 Bucket**: `lucidity-devops-assignment`
> - **DynamoDB Table** (for state locking): `lucidity-devops-assigment-lock`
> - **Region**: `us-east-1`

## 🚀 CI/CD with GitHub Actions
This repository includes two independent GitHub Actions workflows:
1. **Deploy Infrastructure**: Deploys the EKS cluster and VPC via Terraform.
2. **Deploy Application**: Builds the Docker image and deploys the Helm chart to EKS.
3. **Destroy Infrastructure**: (Manual only) Destroys the EKS cluster and VPC to save costs.

To use these workflows, configure the following secrets in your GitHub repository:
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `DOCKERHUB_USERNAME`
- `DOCKERHUB_TOKEN`

---

## Prerequisites

Before you begin, ensure you have the following installed on your machine:
- [AWS CLI](https://aws.amazon.com/cli/)
- [Terraform](https://www.terraform.io/downloads.html) (v1.0+)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Helm](https://helm.sh/docs/intro/install/)

---

## Step-by-Step Deployment Guide

### Step 1: Configure AWS Credentials
Configure your AWS CLI with an IAM user that has sufficient permissions to create VPCs, EKS clusters, and related resources.

```bash
aws configure
# Enter your Access Key, Secret Key, and set default region (e.g., us-east-1)
```

### Step 2: Deploy Infrastructure via Terraform
This step creates the VPC, EKS cluster (`t3.small` managed node group), and uses the Helm provider to deploy the `kube-prometheus-stack` into the `monitoring` namespace.

```bash
cd terraform

# Initialize Terraform plugins
terraform init

# Review the execution plan
terraform plan -var-file="global.tfvars"

# Apply the configuration (Type 'yes' when prompted)
terraform apply -var-file="global.tfvars"
```
*(Note: Creating an EKS cluster typically takes about 15-20 minutes.)*

### Step 3: Connect to the EKS Cluster
Once Terraform successfully applies, configure your local `kubectl` to interact with the new cluster:

```bash
aws eks update-kubeconfig --region us-east-1 --name hello-world-cluster
```
Verify the connection by checking the nodes:
```bash
kubectl get nodes
```

### Step 4: Build and Push the Docker Image
A custom Python Uvicorn server is provided in the `app` directory, managed with `uv`. To build and push the image:

```bash
cd ../app
# Build the image
docker build -t kpcharan/devops-assignment:devops-assigment .
# Push the image to Docker Hub
docker push kpcharan/devops-assignment:devops-assigment
```

### Step 5: Deploy the Hello World App
Now deploy the "Hello World" microservice using the provided Helm chart. This chart uses the custom `kpcharan/devops-assignment:devops-assigment` image built in the previous step.

```bash
# Go to the helm folder
cd ../helm

# Install the helm chart
helm install hello-world ./hello-world
```

---

## Accessing the Application & Monitoring

### 1. Access the Hello World App
The Hello World application is exposed via an AWS Classic Load Balancer (ELB). To get the public URL, run:

```bash
kubectl get svc hello-world
```
Look for the `EXTERNAL-IP` value. It will look something like `ad123...elb.us-east-1.amazonaws.com`. 
Wait ~2-3 minutes for AWS to spin up the load balancer, then open that URL in your web browser. You should see a plain text response: **"Hello World"**.

### 2. Access Grafana (Monitoring)
Grafana was deployed automatically by Terraform. To access the Grafana dashboard securely, we will port-forward it to your local machine:

```bash
kubectl port-forward -n monitoring svc/kube-prometheus-stack-grafana 8080:80
```
- Open your browser to: `http://localhost:8080`
- **Username:** `admin`
- **Password:** `admin123` *(configured in `main.tf`)*

---

## Cleanup
To tear down the infrastructure and avoid incurring AWS charges, run the following commands:

```bash
# 1. Uninstall the Helm chart
helm uninstall hello-world

# 2. Destroy the Terraform infrastructure
cd ../terraform
terraform destroy -var-file="global.tfvars"
```
