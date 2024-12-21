---
title: "Setting Up a Kubernetes Cluster on GKE Using Terraform"
date: 2024-12-11
tags: ["GKE", "Kubernetes", "Terraform", "DevOps"]
repository: "https://github.com/thomasscothamilton/terraform-google-kubernetes-engine"
---

# Setting Up a Kubernetes Cluster on GKE Using Terraform

This guide describes how to configure and deploy a Terraform module to set up a GCP network and an autopilot 
private GKE cluster with a structured and reusable design.

## Repository

[Terraform GKE Cluster Repository](https://github.com/your-username/terraform-gke-cluster)

```shell
git clone git@github.com:thomasscothamilton/terraform-google-kubernetes-engine.git
```

```shell
cd terraform-google-kubernetes-engine
```

## Prerequisites

1. Install Terraform:

```shell
# Download and install Terraform
brew tap hashicorp/tap
brew install hashicorp/tap/terraform

# Verify installation
terraform -v
```

2. Install and Configure Google Cloud SDK:

```shell
# Download and install Google Cloud SDK
curl https://sdk.cloud.google.com | bash

# Restart your shell
exec -l $SHELL

# Initialize the SDK
gcloud init

# Verify installation
gcloud version
```

3. Service Account: Create a service account with the necessary permissions for GKE, Networking, and Storage APIs.

```shell
# Set your project ID
PROJECT_ID="your-gcp-project-id"

# Create the service account
gcloud iam service-accounts create terraform-sa --display-name "Terraform Service Account"

# Assign roles to the service account
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member "serviceAccount:terraform-sa@$PROJECT_ID.iam.gserviceaccount.com" \
  --role "roles/container.admin"
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member "serviceAccount:terraform-sa@$PROJECT_ID.iam.gserviceaccount.com" \
  --role "roles/compute.admin"
gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member "serviceAccount:terraform-sa@$PROJECT_ID.iam.gserviceaccount.com" \
  --role "roles/storage.admin"

# Create a key for the service account
gcloud iam service-accounts keys create ~/terraform-key.json \
  --iam-account terraform-sa@$PROJECT_ID.iam.gserviceaccount.com
```

4. Enable APIs: Ensure the following APIs are enabled:
```shell
 
# Enable Kubernetes Engine API
gcloud services enable container.googleapis.com

# Enable Compute Engine API
gcloud services enable compute.googleapis.com

# Enable Cloud Storage API
gcloud services enable storage.googleapis.com
```
   
5. Credentials: Export Google Cloud credentials:

```bash
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/google-application-credentials.json"
```

## Module Structure

### Files

1. **backend.tf**: Configures the backend to store Terraform state in a GCS bucket.

2. **versions.tf**: Defines Terraform and provider version constraints.

3. **provider.tf**: Configures the Google Cloud provider.

4. **variables.tf**: Defines input variables for the module.

5. **main.tf**: Deploys the GKE cluster using the network module.

6. **network.tf**: Sets up the VPC and subnet configurations.

7. **outputs.tf**: Exports outputs like cluster endpoint, CA certificate, and network details.

## Step 1: Initialize Terraform

1. Clone the repository containing the Terraform module.

2. Navigate to the module directory:

```bash
cd environments/dev
```

3. Initialize Terraform:

```shell
export TF_VAR_project_id="my-gcp-project"
export TF_VAR_region="us-central1"
export TF_VAR_network="my-vpc-network"
export TF_VAR_subnetwork="my-subnet"
```

```bash
# Copy this command:
terraform init
```

## Step 2: Define Input Variables

Create a terraform.tfvars file to provide values for the required variables (project_id and region).

Example terraform.tfvars

```hcl
project_id = "your-gcp-project-id"
region     = "your-region"
```

## Step 3: Plan the Deployment

Generate a plan to review the changes Terraform will apply:

```bash
terraform plan -out main.tfplan
```

## Step 4: Apply the Plan

Apply the configuration to deploy the resources:

```bash
terraform apply main.tfplan
```

## Step 5: Review Outputs

After deployment, Terraform will display outputs defined in outputs.tf.

Example Outputs
    * Cluster Endpoint: Used to connect to the Kubernetes cluster.
    * CA Certificate: Used to authenticate with the cluster.
    * Network Name: The name of the created VPC.

## Key Features

### GCP Network Configuration (network.tf)
 
#### VPC Creation:
* Creates a custom VPC network.
* Configures subnets with secondary IP ranges for pods and services.

#### Subnets:
* A primary subnet for cluster nodes.
* A master subnet for GKE private endpoint authentication.

### Autopilot Private GKE Cluster (main.tf)
* Creates a GKE cluster with the following features:
    * Autopilot mode.
    * Private nodes and private endpoints.
    * Regional clusters for high availability.
    * IP ranges for pods and services.

