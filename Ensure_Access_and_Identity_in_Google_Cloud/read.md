# 1. Create a custom security role
create <b>role-definition.yaml</b> file
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
create custom rule
```
gcloud iam roles create [CUSTOM_ROLE_NAME] --project $DEVSHELL_PROJECT_ID \
--file role-definition.yaml
```
