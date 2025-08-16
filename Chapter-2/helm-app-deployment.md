```bash
oc new-project helm-app
helm repo add 	bitnami https://charts.bitnami.com/bitnami
helm list
helm repo update
```
### 1. Create the Helm Chart Structure
   ```bash
   helm create online-shop
   rm -rf online-shop/templates/*  # Remove default templates
   rmdir charts
   ```
### 2. Chart Structure
Here's the complete structure we'll create:
online-shop/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── namespace.yaml
    ├── deployment.yaml
    ├── service.yaml
    └── route.yaml
```bash
cd online-shop/templates/
touch {namespace,deployment,service,route}.yaml
```
### 3. Create each file
```bash
nano online-shop/chart.yaml
```
#Chart.yaml
apiVersion: v2
name: online-shop
description: A Helm chart for Online Shop application with rolling update strategy
version: 0.1.0
appVersion: 1.0.0

```bash
#Update the below files given as per github repo:
nano online-shop/values.yaml
nano online-shop/templates/namespace.yaml
nano online-shop/templates/deployment.yaml
nano online-shop/templates/service.yaml
nano online-shop/templates/route.yaml
nano online-shop/templates/hpa.yaml
```

### 4. How to Use the Chart
```bash
helm install online-shop ./online-shop
#Upgrade the chart (for example, to change the image):
helm upgrade online-shop ./online-shop --set deployment.image= cloudtrain707/shop_no_footer_v1
helm upgrade online-shop ./online-shop --set deployment.replicas=3
#To customize multiple values:
helm install online-shop ./online-shop \
  --set namespace=custom-namespace \
  --set deployment.replicas=3 \
  --set route.host=custom.host.example.com
```

### 5. Clean Up (Uninstall Helm Release)
```bash
helm list
helm uninstall online-shop
helm list        # Should not show online-shop
oc get all -n rolling-update-strategy #Should show "No resources found"
```
