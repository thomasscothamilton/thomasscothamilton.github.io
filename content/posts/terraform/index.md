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

## Prerequisites

1. Terraform Installed: Ensure Terraform CLI is installed. Download Terraform.

2. 2 Google Cloud SDK: Install and configure the Google Cloud SDK.

3. Service Account: Create a service account with the necessary permissions for GKE, Networking, and Storage APIs.

4. Enable APIs: Ensure the following APIs are enabled:
   * Kubernetes Engine API
   * Compute Engine API
   * Cloud Storage API 
   
5. Credentials: Export Google Cloud credentials:

```bash
export GOOGLE_APPLICATION_CREDENTIALS="/path/to/your-service-account-key.json"
```

## Module Structure

### Files

1. backend.tf: Configures the backend to store Terraform state in a GCS bucket.

2. versions.tf: Defines Terraform and provider version constraints.

3. provider.tf: Configures the Google Cloud provider.

4. variables.tf: Defines input variables for the module.

5. main.tf: Deploys the GKE cluster using the network module.

6. network.tf: Sets up the VPC and subnet configurations.

7. outputs.tf: Exports outputs like cluster endpoint, CA certificate, and network details.

## Step 1: Initialize Terraform

1. Clone the repository containing the Terraform module.

2. Navigate to the module directory:

```bash
cd /path/to/module
```

3. Initialize Terraform:

```bash
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

