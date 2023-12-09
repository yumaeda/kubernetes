# Deploy
## Create Namespace
```zsh
kubectl apply -f namespace.yaml
```
## Get a list of Namespaces
```zsh
kubectl get namespaces
```
## Change current Namespace
```zsh
kubens sakabas-api
```
## Create Secrets
```zsh
kubectl create secret generic db-pwd --from-literal=db-pwd='${PWD}'
kubectl create secret generic db-host --from-literal=db-host='${DB_HOST}'
kubectl create secret generic db-name --from-literal=db-name='${DB_NAME}'
kubectl create secret generic db-user --from-literal=db-user='${DB_USER}'
kubectl create secret generic s3-id --from-literal=s3-id='${S3_ID}'
kubectl create secret generic s3-secret --from-literal=s3-secret='${S3_SECRET}'
kubectl create secret generic s3-region --from-literal=s3-region='${S3_REGION}'
```
## Get a list of Secrets
```zsh
kubectl get secrets
```
## Create LimitRange
```zsh
kubectl apply -f limitrange.yaml
```
## Get the LimitRange for the sakabas Namespace
```zsh
kubectl describe limitranges --namespace=sakabas-api
```
## Build simplified manifest
```zsh
kustomize build . > deploy.yaml
```
## Create Deployment & Service
```zsh
kubectl apply -f deploy.yaml
```
## List Pods
```zsh
kubectl get pods --namespace=sakabas-api
```
## Create managed certificate
```zsh
kubectl apply -f certificate.yaml
```
## Check the status of the certificate
```zsh
kubectl get managedcertificate --namespace=sakabas-api
```
## Create Ingress
```zsh
kubectl apply -f ingress.yaml
```
## Get External IP of the load balancer
```zsh
kubectl describe ingress api-ingress --namespace=sakabas-api
```

&nbsp;

# Delete
## Delete Ingress
```zsh
kubectl delete -f ingress.yaml
```
## Delete Google-managed certificate
```zsh
kubectl delete -f certificate.yaml
```
## Delete Service
```zsh
kubectl delete service api --namespace=sakabas-api
```
## Delete Deployment
```zsh
kubectl delete deployment api --namespace=sakabas-api
```
## Delete LimitRange
```zsh
kubectl delete -f limitrange.yaml
```
## Delete the Namespace
```zsh
kubectl delete -f namespace.yaml
```
