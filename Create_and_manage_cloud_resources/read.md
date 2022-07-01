# Set the default region and zone for all resources
```
ZONE="us-east1-b"

REGION="us-east1"

gcloud config set compute/zone $ZONE

gcloud config set compute/region $REGION
```

# Create a instance for your project
```
INSTANCE_NAME="*****"
MACHINE_TYPE="f1-micro"

gcloud compute instances create $INSTANCE_NAME --machine-type=$MACHINE_TYPE
```

# Create a Kubernetes Service cluster
```
CLUSTER_NAME="my-cluster"

CONTAINER_NAME="hello-app"

CONTAINER_IMAGE="gcr.io/google-samples/hello-app:2.0"

PORT=*****

gcloud container clusters create $CLUSTER_NAME

gcloud container clusters get-credentials $CLUSTER_NAME

kubectl create deployment $CONTAINER_NAME --image=$CONTAINER_IMAGE

kubectl expose deployment $CONTAINER_NAME --type=LoadBalancer --port $PORT

kubectl get service
```

# Set up an HTTP load balancer

1. Create an instance template
```
INSTANCE_TEMPLATE_NAME="lb-backend-template"

gcloud compute instance-templates create $INSTANCE_TEMPLATE_NAME \
   --network=default \
   --subnet=default \
   --tags=allow-health-check \
```

2. Create a target pool and 2 nginx instances
```
TARGET_POOL_NAME="www-pool"

gcloud compute target-pools create $TARGET_POOL_NAME

INSTANCE1_NAME="www1"

INSTANCE2_NAME="www2"

gcloud compute instances create $INSTANCE1_NAME \
  --image-family debian-9 \
  --image-project debian-cloud \
  --tags network-lb-tag \
  --metadata startup-script="#! / bin / bash
    apt-get update
    apt-get install -y nginx
    service nginx start"
    
gcloud compute instances create $INSTANCE2_NAME \
  --image-family debian-9 \
  --image-project debian-cloud \
  --tags network-lb-tag \
  --metadata startup-script="#! / bin / bash
    apt-get update
    apt-get install -y nginx
    service nginx start"
    
gcloud compute target-pools add-instances $TARGET_POOL_NAME --instances $INSTANCE1_NAME,$INSTANCE2_NAME
```

3. Create a managed instance group.
```
INSTANCE_GROUP_NAME="lb-backend-group"

gcloud compute instance-groups managed create $INSTANCE_GROUP_NAME \
   --template=$INSTANCE_TEMPLATE_NAME --size=2
```

4. Firewall rule Create a firewall rule named to allow traffic (TCP port 80)
```
FIREWALL_RULE_NAME="*****"

gcloud compute firewall-rules create $FIREWALL_RULE_NAME \
    --network=default \
    --action=allow \
    --direction=ingress \
    --source-ranges=130.211.0.0/22,35.191.0.0/16 \
    --target-tags=allow-health-check \
    --rules=tcp:80
```

5. Create a health check
```
HEALTH_CHECK_NAME="basic-check"

gcloud compute health-checks create http $HEALTH_CHECK_NAME --port 80
```

6. Create a backend service and attach a managed instance group with a named port (HTTP: 80).
```
BACKEND_SERVICE_NAME="web-backend-service"

gcloud compute backend-services create $BACKEND_SERVICE_NAME \
    --protocol=HTTP \
    --port-name=http \
    --health-checks=$HEALTH_CHECK_NAME \
    --global
    
gcloud compute backend-services add-backend $BACKEND_SERVICE_NAME \
    --instance-group=$INSTANCE_GROUP_NAME \
    --global
```

7. Create a URL map and set up a target HTTP proxy to route requests to your URL map.
```
URL_MAP_NAME="web-map-http"

gcloud compute url-maps create $URL_MAP_NAME \
    --default-service $BACKEND_SERVICE_NAME


TARGET_HTTP_PROXY_NAME="http-lb-proxy"

gcloud compute target-http-proxies create $TARGET_HTTP_PROXY_NAME \
    --url-map $URL_MAP_NAME
```

8. Create a forwarding rule.
```
FORWARDING_RULE_NAME=http-content-rule

gcloud compute forwarding-rules create $FORWARDING_RULE_NAME \
    --address=lb-ipv4-1\
    --global \
    --target-http-proxy=$TARGET_HTTP_PROXY_NAME \
    --ports=80
```
