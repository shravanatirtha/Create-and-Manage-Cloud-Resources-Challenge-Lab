# Create and Manage Cloud Resources Challenge Lab
Create and Manage Cloud Resources: Challenge Lab of Google Cloud

### Task 1. Create a project jumphost instance 

Navigation menu > Compute engine > VM Instance

1. Name for the VM instance :<b> [specified instance name]</b>

2. Region : <b>leave default region</b>

3. Zone : <b>leave default zone</b>

4. Machine Type : <b>f1-micro (Series — N1)</b>

5. Boot Disk : use the default image type <b>(Debian Linux)</b>

6. Leave other everything else as "default"

7. Click -> <b> Create</b>

### Task 2. Create a Kubernetes service cluster
1. Activate Cloud Shell 

2. Paste the commands one by one

> gcloud auth list

> gcloud config set compute/zone us-east1-b

> gcloud container clusters create nucleus-webserver1

> gcloud container clusters get-credentials nucleus-webserver1

> kubectl create deployment hello-app --image=gcr.io/google-samples/hello-app:2.0

> kubectl expose deployment hello-app --type=LoadBalancer --port [specified port]

> kubectl get service

### Task 3. Set up an HTTP load balancer
<b>Paste the following commands in Cloud Shell</b><br />
cat << EOF > startup.sh<br />
#! /bin/bash<br />
apt-get update<br />
apt-get install -y nginx<br />
service nginx start<br />
sed -i -- 's/nginx/Google Cloud Platform - '"\$HOSTNAME"'/' /var/www/html/index.nginx-debian.html<br />
EOF<br />

#### 1 .Create an instance template :
> gcloud compute instance-templates create nginx-template --metadata-from-file startup-script=startup.sh
#### 2 .Create a target pool :
> gcloud compute target-pools create nginx-pool

( if the location is other than us-east1, then press n)
choose the location number that says us-east1
#### 3 .Create a managed instance group :
> gcloud compute instance-groups managed create nginx-group --base-instance-name nginx --size 2 --template nginx-template --target-pool nginx-pool 

> gcloud compute instances list
#### 4 .Create a firewall rule to allow traffic (80/tcp) :
> gcloud compute firewall-rules create [specified firewall name] --allow tcp:80

> gcloud compute forwarding-rules create nginx-lb --region us-east1 --ports=80 --target-pool nginx-pool

> gcloud compute forwarding-rules list
#### 5 .Create a health check :
> gcloud compute http-health-checks create http-basic-check

> gcloud compute instance-groups managed set-named-ports nginx-group --named-ports http:80
#### 6 .Create a backend service and attach the manged instance group :
> gcloud compute backend-services create nginx-backend --protocol HTTP --http-health-checks http-basic-check --global

> gcloud compute backend-services add-backend nginx-backend --instance-group nginx-group --instance-group-zone us-east1-b --global

#### 7 .Create a URL map and target HTTP proxy to route requests to your URL map :
> gcloud compute url-maps create web-map --default-service nginx-backend

> gcloud compute target-http-proxies create http-lb-proxy --url-map web-map

#### 8 .Create a forwarding rule :
> gcloud compute forwarding-rules create http-content-rule --global --target-http-proxy http-lb-proxy --ports 80

> gcloud compute forwarding-rules list

Wait for 5 minutes as the progress takes time to update.You’ve successfully completed the challenge lab. Hope it helped. 

