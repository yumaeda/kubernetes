# Deploy
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
kubectl get pods --namespace=sakabas
```
## Create managed certificate
```zsh
kubectl apply -f certificate.yaml
```
## Check the status of the certificate
```zsh
gcloud beta compute ssl-certificates list
```
## Create a ServiceAccount
```zsh
kubectl apply -f serviceaccount.yaml
```
## Get a list of ServiceAccounts
```zsh
kubectl get serviceaccounts --namespace=ramen-mania
```
## Create CronJob
```zsh
kubectl apply -f cronjob.yaml
```
## Check a CronJob's configuration
```zsh
kubectl describe cronjob terminated-pod-cleaner --namespace=ramen-mania
```

&nbsp;

# Delete
## Delete Google-managed certificate
```zsh
kubectl delete -f certificate.yaml
```
## Delete Service
```zsh
kubectl delete service web --namespace=sakabas
```
## Delete Deployment
```zsh
kubectl delete deployment web --namespace=sakabas
```
## Delete a CronJob
```zsh
kubectl delete -f cronjob.yaml
```
## Delete the ServiceAccount
```zsh
kubectl delete -f serviceaccount.yaml
```
