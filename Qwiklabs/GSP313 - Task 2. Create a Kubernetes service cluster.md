##Task 2. Create a Kubernetes service cluster

gcloud config set compute/zone us-east1-b
gcloud container clusters create cluster-1
gcloud container clusters get-credentials cluster-1
kubectl create deployment hello-app --image=gcr.io/google-samples/hello-app:2.0
kubectl expose deployment hello-app --type=LoadBalancer --port 8080
kubectl get service



gcloud container clusters delete cluster-1