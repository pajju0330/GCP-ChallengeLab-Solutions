# GCP Challenge 1: Create and Manage Cloud Resources

## task1 : Create an Instance
Command - 
```
gcloud compute instances create nucleus-jumphost-512 --machine-type ec2-micro
```

## task2: create kubernetes service cluster
Commands -
# 1] Create Cluster - 
```gcloud config set compute/zone us-west1-c
gcloud container clusters create nucleus-webserver1
```
# 2] Get creds - 
```
gcloud container clusters get-credentials nucleus-webserver1
```

# 3]Deploy image to server - 
```kubectl create deployment hello-app --image=gcr.io/google-samples/hello-app:2.0
kubectl expose deployment hello-app --type=LoadBalancer --port 8081
```

## task3: Setting up HTTP LoadBalancer

# 1] nano startup.sh
***paste the following code given in the lab***
# 2] Create instance template - 
```gcloud compute instance-templates create web-server-template \
--metadata-from-file startup-script=startup.sh \
--network nucleus-vpc \
--machine-type g1-small \
--region us-east1
```

# 2] create managed groups - 
```
gcloud compute instance-groups managed create web-server-group \
--base-instance-name web-server \
--size 2 \
--template web-server-template \
--region us-east1
```

# 3] Create firewall - 
```gcloud compute firewall-rules create grant-tcp-rule-632 \
--allow tcp:80 \
--network nucleus-vpc`
```

# 4] Create health Check - 
```
gcloud compute http-health-checks create http-basic-check
```

# 5] Create a backend service:
```gcloud compute instance-groups managed \
set-named-ports web-server-group \
--named-ports http:80 \
--region us-east1
```
-
```
gcloud compute backend-services create web-server-backend \
--protocol HTTP \
--http-health-checks http-basic-check \
--global
```
-
```
gcloud compute backend-services add-backend web-server-backend \
--instance-group web-server-group \
--instance-group-region us-east1 \
--global
```

# 6] Create a URL MAP
```
gcloud compute url-maps create web-server-map \
--default-service web-server-backend
gcloud compute target-http-proxies create http-lb-proxy \
--url-map web-server-map
```

# 7] Create forwarding rule
```
gcloud compute forwarding-rules create http-content-rule \
--global \
--target-http-proxy http-lb-proxy \
--ports 80
```

***DONE !! [ wait 10 minutes before clicking on verify ]***