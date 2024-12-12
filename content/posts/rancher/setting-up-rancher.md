Prerequisites
Google Cloud SDK (gcloud) installed and authenticated
kubectl installed and configured
Helm (v3+) installed
Ensure that you are logged into your Google Cloud account and have set the desired project and compute zone:

bash
Copy code
gcloud auth login
gcloud config set project <YOUR_GCP_PROJECT_ID>
gcloud config set compute/zone <YOUR_COMPUTE_ZONE>
Step 1: Create a GKE Cluster
Create a GKE cluster with a sufficient number of nodes and appropriate machine types:

bash
Copy code
gcloud container clusters create rancher-cluster \
--num-nodes=3 \
--machine-type=e2-standard-4 \
--enable-ip-alias
After the cluster is created, get the cluster credentials:

bash
Copy code
gcloud container clusters get-credentials rancher-cluster
Step 2: Install cert-manager
Rancher requires cert-manager to manage certificates. Install the JetStack Helm repository and then install cert-manager:

bash
Copy code
helm repo add jetstack https://charts.jetstack.io
helm repo update
bash
Copy code
kubectl create namespace cert-manager
helm install cert-manager jetstack/cert-manager \
--namespace cert-manager \
--version v1.11.0 \
--set installCRDs=true
Wait for cert-manager pods to be in a Running state:

bash
Copy code
kubectl get pods -n cert-manager
Step 3: Add the Rancher Helm Chart Repository
Add the official Rancher Helm repository:

bash
Copy code
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
helm repo update
Step 4: Install Rancher
Create a namespace for Rancher and then install it using Helm:

bash
Copy code
kubectl create namespace cattle-system
bash
Copy code
helm install rancher rancher-latest/rancher \
--namespace cattle-system \
--set hostname=<RANCHER_FQDN> \
--set replicas=3
Replace <RANCHER_FQDN> with a valid DNS name pointed to your cluster’s load balancer.

Step 5: Validate the Installation
Check the Rancher deployment status:

bash
Copy code
kubectl -n cattle-system rollout status deploy/rancher
kubectl -n cattle-system get pods
Once all pods are running, you can retrieve the external IP address of the Rancher service:

bash
Copy code
kubectl -n cattle-system get svc rancher
Point your browser to the <RANCHER_FQDN> or the LoadBalancer IP address, then proceed with the initial Rancher setup steps via the web UI.