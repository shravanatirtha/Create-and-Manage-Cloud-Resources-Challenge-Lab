# Create-and-Manage-Cloud-Resources-Challenge-Lab
Create and Manage Cloud Resources: Challenge Lab of Google Cloud

### Task 1. Create a project jumphost instance
Navigation menu > Compute engine > VM Instance

Name for the VM instance : [specified instance name]

Region : leave Default Region

Zone : leave Default Zone

Machine Type : f1-micro (Series — N1)

Boot Disk : use the default image type (Debian Linux)

Leave other everything else as "default"

Click -> Create

### Task 2. Create a Kubernetes service cluster
Activate Cloud Shell 

Paste the commands one by one

gcloud auth list

gcloud config set compute/zone us-east1-b

gcloud container clusters create nucleus-webserver1

gcloud container clusters get-credentials nucleus-webserver1

kubectl create deployment hello-app --image=gcr.io/google-samples/hello-app:2.

kubectl expose deployment hello-app --type=LoadBalancer --port [specified port]

kubectl get service

### Task 3. Set up an HTTP load balancer
cat << EOF > startup.sh
#! /bin/bash
apt-get update
apt-get install -y nginx
service nginx start
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html
EOF

#### 1 .Create an instance template :
gcloud compute instance-templates create nginx-template \
--metadata-from-file startup-script=startup.sh
#### 2 .Create a target pool :
gcloud compute target-pools create nginx-pool

( if the location is other than us-east1, then press n)
choose the location number that says us-east1

gcloud compute instance-groups managed create nginx-group \
--base-instance-name nginx \
--size 2 \
--template nginx-template \
--target-pool nginx-pool

gcloud compute instances list

gcloud compute firewall-rules create [specified firewall name] --allow tcp:80

gcloud compute forwarding-rules create nginx-lb \
--region us-east1 \
--ports=80 \
--target-pool nginx-pool

gcloud compute forwarding-rules list

gcloud compute http-health-checks create http-basic-check

gcloud compute instance-groups managed \
set-named-ports nginx-group \
--named-ports http:80

gcloud compute backend-services create nginx-backend \
--protocol HTTP --http-health-checks http-basic-check --global

gcloud compute backend-services add-backend nginx-backend \
--instance-group nginx-group \
--instance-group-zone us-east1-b \
--global

#### 7 .Create a URL map and target HTTP proxy to route requests to your URL map :

gcloud compute url-maps create web-map \
--default-service nginx-backend

gcloud compute target-http-proxies create http-lb-proxy \
--url-map web-map

#### 8 .Create a forwarding rule :

gcloud compute forwarding-rules create http-content-rule \
--global \
--target-http-proxy http-lb-proxy \
--ports 80

gcloud compute forwarding-rules list

Wait for 5 minutes as the progress takes time to update.You’ve successfully completed the challenge lab. Hope it helped. 

