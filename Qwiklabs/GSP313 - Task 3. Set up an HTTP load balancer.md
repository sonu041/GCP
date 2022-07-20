#Create and Manage Cloud Resources: Challenge Lab
##Task 3. Set up an HTTP load balancer
https://partner.cloudskillsboost.google/focuses/11604?parent=catalog

gcloud config set compute/zone us-east1-b
gcloud config set compute/region us-east1

## Create the startup script
cat << EOF > startup.sh
```#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF```


## Create an instance template.
gcloud compute instance-templates create nginx-template \
   --metadata-from-file startup-script=startup.sh
	 
## Create Target Pool 
gcloud compute target-pools create nginx-pool --region=us-east1

## Create a managed instance group.
gcloud compute instance-groups managed create nginx-group \
   --base-instance-name=nginx \
   --template=nginx-template --size=2 \
   --zone=us-east1-b \
   --target-pool=nginx-pool
   
## Create Firewall Rule
gcloud compute firewall-rules create accept-tcp-rule-918 \
    --action=allow \
    --rules=tcp:80

## Create a forwarding rule.
gcloud compute forwarding-rules create nginx-lb \
	--region=us-east1 \
    --ports=80 \
	--target-pool=nginx-pool
    
## Create a health check.
gcloud compute http-health-checks create http http-basic-check

gcloud compute instance-groups managed set-named-ports nginx-group --named-ports http:80

## Create a backend service, and attach the managed instance group with named port (http:80).
gcloud compute backend-services create nginx-backend \
    --protocol=HTTP \
    --port-name=http \
    --health-checks=http-basic-check \
    --global

gcloud compute backend-services add-backend nginx-backend \
    --instance-group=nginx-group \
    --instance-group-zone=us-east1-b \
    --global

## Create a URL map, and target the HTTP proxy to route requests to your URL map.
gcloud compute url-maps create web-map-http \
    --default-service nginx-backend

gcloud compute target-http-proxies create http-lb-proxy \
    --url-map web-map-http
	
## Create another firewall rule
gcloud compute forwarding-rules create http-conent-rule \
	--global \
	--target-http-proxy=http-lb-proxy \
	--port=80