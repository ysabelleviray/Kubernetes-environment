1. Create namespace: kubectl create namespace production
2. Configure configmaps:
kubectl create configmap app-config \
  --from-literal=APP_ENV=production \
  --namespace=production
3. Define sensitive information:
kubectl create secret generic app-secret \
  --from-literal=DB_PASSWORD=prodpassword123 \
  --namespace=production
4. View secrets:
kubectl get secrets -n production
5. View metadata of secrets:
kubectl describe secret app-secret -n production
6. Decode password:
kubectl get secret app-secret -n production -o jsonpath="{.data.DB_PASSWORD}" | base64 --decode
7. Create a generic deployment YAML template. Save it as prod-deployment-template.yaml:
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
8. Deploy the production environment:
sed 's/production/production/' prod-deployment-template.yaml > prod-deployment.yaml 
kubectl apply -f prod-deployment.yaml
9. Expose the production environment:
kubectl expose deployment nginx-app \
  --type=NodePort \
  --name=nginx-service \
  --port=80 \
  --target-port=80 \
  --namespace=production
10. Verify the deployment:
kubectl get pods -n production
kubectl get service -n production
11. Access and verify the production app:
Pod Names:
- nginx-app-65968468b6-cv9f8 
- nginx-app-65968468b6-jbtdq 
- nginx-app-65968468b6-jqpc8 
kubectl exec -it <pod—name> n production -- /bin/bash
echo $APP_ENV
echo $DB_PASSWORD






