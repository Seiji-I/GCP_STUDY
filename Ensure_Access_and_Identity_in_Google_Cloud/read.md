# 1. Create a custom security role
First, create <b>role-definition.yaml</b> file

```
vi role-definition.yaml
```
Paste the code below and save this file
```
title: my custom rule
description: my custom rule description
stage: "ALPHA"
includedPermissions:
- storage.buckets.get
- storage.objects.get
- storage.objects.list
- storage.objects.update
- storage.objects.create
```
And then run this command to create a custom rule
```
$CUSTOM_ROLE_NAME=[CUSTOM_ROLE_NAME]
gcloud iam roles create $CUSTOM_ROLE_NAME --project $DEVSHELL_PROJECT_ID \
--file role-definition.yaml
```

# 2. Create a service account
Define service account name and create it
```
SERVICE_ACCOUNT_NAME=[SERVICE_ACCOUNT_NAME]
gcloud iam service-accounts create $SERVICE_ACCOUNT_NAME --display-name "my service account"
```
Binding these roles to control minimal GKE
- logging.logWriter
- monitoring.metricWriter
- monitoring.viewer
- stackdriver.resourceMetadata.writer
```
gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID \
    --member serviceAccount:$SERVICE_ACCOUNT_NAME@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --role roles/logging.logWriter
    
gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID \
    --member serviceAccount:$SERVICE_ACCOUNT_NAME@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --role roles/monitoring.metricWriter
    
gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID \
    --member serviceAccount:$SERVICE_ACCOUNT_NAME@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --role roles/monitoring.viewer
    
gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID \
    --member serviceAccount:$SERVICE_ACCOUNT_NAME@$DEVSHELL_PROJECT_ID.iam.gserviceaccount.com --role roles/stackdriver.resourceMetadata.writer
```


```
gcloud container clusters create orca-cluster-389 \
	--zone ***** \
	--network ***** \
    --subnetwork ***** \
    --enable-master-authorized-networks \
	--enable-ip-alias \
	--master-ipv4-cidr 172.16.0.32/28 \
    --enable-private-nodes \
    --enable-private-endpoint
```
