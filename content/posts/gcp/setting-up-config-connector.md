---
title: "How to Install GCP Config Connector"
date: 2024-12-12
description: "A step-by-step guide to installing and configuring the GCP Config Connector on your Kubernetes cluster."
keywords: ["GCP", "Config Connector", "Kubernetes", "Cloud Resource Management"]
tags: ["gcp", "config connector", "installation", "kubernetes"]
draft: false
---

## Overview

Google Cloud Config Connector (Config Connector) is a Kubernetes add-on that allows you to manage Google Cloud Platform (GCP) resources using Kubernetes-style configuration. By installing Config Connector on your cluster, you can provision and manage GCP services—like Cloud SQL instances, Pub/Sub topics, and Cloud Storage buckets—through standard Kubernetes manifests.

In this guide, we’ll walk you through the prerequisites, the installation process, and how to verify a successful installation.

## Prerequisites

Before you begin, ensure you have:

1. **A GCP Project:** You’ll need a Google Cloud project with appropriate billing and permissions. Make a note of your project ID, as you’ll use it during installation.

2. **kubectl:** Make sure `kubectl` is installed and configured to communicate with your Kubernetes cluster.

   ```bash
   kubectl version
Cluster Admin Permissions: You must have cluster-admin permissions on your Kubernetes cluster.

GCP Credentials: You need a GCP service account key or Workload Identity configured, so that Config Connector can authenticate to your GCP project and manage resources.

Step 1: Enable the Config Connector API
First, you’ll need to ensure that the Config Connector API is enabled on your GCP project:

bash
Copy code
gcloud services enable containerregistry.googleapis.com \
--project <YOUR_PROJECT_ID>
(Replace <YOUR_PROJECT_ID> with your actual project ID.)

This command enables the required API so the connector can pull images and resources.

Step 2: Assign or Create a Service Account
You must provide Config Connector a service account that has the necessary permissions to create and manage GCP resources. For example, you might create a service account with an editor role for a proof-of-concept (not recommended for production):

bash
Copy code
gcloud iam service-accounts create config-connector-sa \
--display-name="Config Connector Service Account" \
--project <YOUR_PROJECT_ID>

gcloud projects add-iam-policy-binding <YOUR_PROJECT_ID> \
--member="serviceAccount:config-connector-sa@<YOUR_PROJECT_ID>.iam.gserviceaccount.com" \
--role="roles/editor"
Important: For production environments, follow the principle of least privilege and assign only the necessary roles.

Step 3: Download the Config Connector Manifest
The Config Connector installation manifests are hosted on GitHub. Download the latest manifest bundle:

bash
Copy code
curl -o configconnector-operator.yaml \
https://raw.githubusercontent.com/GoogleCloudPlatform/k8s-config-connector/main/install-bundle/namespace-injection/0-operator.yaml
This operator YAML file contains the operator for Config Connector. You’ll apply this operator to your cluster to manage the installation and lifecycle of the connector.

Step 4: Install the Config Connector Operator
Apply the operator manifest to your cluster:

bash
Copy code
kubectl apply -f configconnector-operator.yaml
This command installs the Config Connector operator into the configconnector-operator-system namespace. The operator will manage the lifecycle of Config Connector instances within the cluster.

Step 5: Configure a ConfigConnector Instance
Next, create a ConfigConnector custom resource to enable Config Connector functionality in your cluster. You can do this by creating a manifest file, such as configconnector.yaml:

yaml
Copy code
apiVersion: core.cnrm.cloud.google.com/v1beta1
kind: ConfigConnector
metadata:
name: configconnector.core.cnrm.cloud.google.com
spec:
mode: cluster
googleServiceAccount: "config-connector-sa@<YOUR_PROJECT_ID>.iam.gserviceaccount.com"
credentialSecretName: gcp-credentials
Update the googleServiceAccount field with your service account. If you are using a credential file, you will need to create a Kubernetes secret containing your service account key:

bash
Copy code
kubectl create secret generic gcp-credentials \
--from-file=key.json=</path/to/your/service-account-key.json>
Then, apply the Config Connector resource:

bash
Copy code
kubectl apply -f configconnector.yaml
Step 6: Verify the Installation
Check that the Config Connector components are running correctly:

bash
Copy code
kubectl get pods -n configconnector-operator-system
You should see one or more Pods running for the Config Connector operator.

Additionally, verify that the Config Connector resource status is Ready:

bash
Copy code
kubectl get configconnector configconnector.core.cnrm.cloud.google.com -o yaml
Look for status.state: READY or any conditions that indicate a successful setup.

Step 7: Test the Installation
To confirm everything is working properly, try creating a simple GCP resource using a Kubernetes manifest. For example, create a Pub/Sub topic:

yaml
Copy code
apiVersion: pubsub.cnrm.cloud.google.com/v1beta1
kind: PubSubTopic
metadata:
name: my-test-topic
spec:
name: my-test-topic
Apply this manifest:

bash
Copy code
kubectl apply -f test-topic.yaml
Check the status of your resource:

bash
Copy code
kubectl describe pubsubtopic my-test-topic
If the resource is created successfully, you should see it reflected in the GCP Console as well.

Conclusion
You have successfully installed and configured the GCP Config Connector on your Kubernetes cluster. From here, you can manage a wide variety of GCP services directly from Kubernetes manifests, streamlining your infrastructure operations and maintaining a single source of truth for both application and infrastructure resources.