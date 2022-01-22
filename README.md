<p align="center">
    <img src="files/gcp.png" height="150">
</p>

# Secure Workloads in Google Kubernetes Engine: Challenge Lab


# Task 1

Replace <b>[Cluster Name]</b> with <b>Cluster Name</b> from your project.

```md
gcloud container clusters create [Cluster Name] \
   --zone us-central1-c \
   --machine-type n1-standard-4 \
   --num-nodes 2 \
   --enable-network-policy
```
# Task 2

Replace <b>[Cloud SQL Instance] & [Service Account]</b> with <b>Cloud SQL Instance & Service Account </b> names from your project.
```md
gcloud sql instances create [Cloud SQL Instance] --region us-central1
```
* Create a Database in [Cloud SQL Instance] from console
  <p>Name : wordpress</p>
* Create a User for the Database wordpress from the console  
  
```md
gcloud iam service-accounts create [Service Account]
```
```md
gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID \
   --member="serviceAccount:[Service Account]@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com" \
   --role="roles/cloudsql.client"
```
```md
gcloud iam service-accounts keys create key.json --iam-account=[Service Account]@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com
```
```md
kubectl create secret generic cloudsql-instance-credentials --from-file key.json
```
```md
kubectl create secret generic cloudsql-db-credentials \
   --from-literal username=wordpress \
   --from-literal password=''
```
# Task 3

```md
helm version
helm repo add stable https://charts.helm.sh/stable
helm repo update
```
```md
helm install nginx-ingress stable/nginx-ingress --set rbac.create=true
```
```md
kubectl get service nginx-ingress-controller
```
```md
. add_ip.sh
```
The script will output your DNS name
<p>Note down your DNS name -- Format : <b>YOUR_LAB_USERNAME.labdns.xyz</b>

```md
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v0.16.0/cert-manager.yaml
```
```md
kubectl create clusterrolebinding cluster-admin-binding \
   --clusterrole=cluster-admin \
   --user=$(gcloud config get-value core/account)
```
Open Code Editor and edit these files <br>
* <b>issuer.yaml</b> and set the email address to the email address of your wordpress service account. 
* <b>ingress.yaml</b> and set HOST_NAME with your DNS name
  
# Task 4

Edit <b>network-policy.yaml</b> and add these lines at the end

```md
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-nginx-access-to-internet
spec:
  podSelector:
    matchLabels:
      app: nginx-ingress
  policyTypes:
  - Ingress
  ingress:
  - {}
```
Apply Network Policy
```md
kubectl apply -f network-policy.yaml
```
# Task 5
From the console
* Go to Security➠Binary Authorization and Enable it
* Click Edit Policy
* Change Default Rule to Disallow all images
* Create Specific Rule to GKE Cluster
* Add Specific Rule to Your Cluster ID
* Add image pattern and add given patterns
* Save Policy
  
Go to your Kubernetes Cluster and enable Binary Authorization from it's Security

# Task 6

Simply apply yaml files

```md
  kubectl apply -f psp-restrictive.yaml
  kubectl apply -f psp-role.yaml
  kubectl apply -f psp-use.yaml
```

