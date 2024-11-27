# Kubernetes Production Deployment Guide

This guide provides a step-by-step process for deploying and managing a production environment using Kubernetes. It covers creating namespaces, setting up `ConfigMaps` and `Secrets`, deploying a production application, and exposing it via a service.

## Steps to Set Up the Production Environment

### 1. **Create the Production Namespace**

```bash
kubectl create namespace production
```
### 2. **Configure ConfigMaps**

```bash
kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --namespace=production
```
### 3. **Define sensitive information**
```bash
kubectl create secret generic app-secret \
  --from-literal=DB_PASSWORD=prodpassword123 \
  --namespace=production
```
### 4. **View secrets**
```bash
kubectl get secrets -n production
```
### 5. **View metadata of secrets**
```bash
kubectl describe secret app-secret -n production
```
### 6. **Decode password**
```bash
kubectl get secret app-secret -n production -o jsonpath="{.data.DB_PASSWORD}" | base64 --decode
```
### 7. **Create a generic deployment YAML template. Save it as prod-deployment-template.yaml**
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
      - name: nginx
        image: nginx
        env:
        - name: APP_ENV
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_ENV
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: DB_PASSWORD
```
### 8. **Deploy the production environment**
```bash
sed 's/production/production/' prod-deployment-template.yaml > prod-deployment.yaml 
kubectl apply -f prod-deployment.yaml
```
### 9. **Expose the production environment**
```bash
kubectl expose deployment nginx-app \
  --type=NodePort \
  --name=nginx-service \
  --port=80 \
  --target-port=80 \
  --namespace=production
```
### 10. **Verify the deployment**
```bash
kubectl get pods -n production
kubectl get service -n production
```
### 11. **Access the production app**
```bash
Pod Names:
- nginx-app-65968468b6-cv9f8 
- nginx-app-65968468b6-jbtdq 
- nginx-app-65968468b6-jqpc8 
kubectl exec -it <pod—name> n production -- /bin/bash
echo $APP_ENV
echo $DB_PASSWORD
```

