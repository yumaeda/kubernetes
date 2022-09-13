# kubernetes
## Preparations
### [Setup GCP](https://github.com/yumaeda/gcp/blob/main/README.md)
### Install packages
```sh
brew install \
  kubectl \
  kubectx \
  kustomize
```

### Enable `container.googleapis.com`
```zsh
gcloud services enable container.googleapis.com
```

### Set environment variables
```zsh
export PROJECT_ID=xxxx
export IMG_NAME=xxxx
export IMG_VERSION=x.x.x
export REGION=us-central1
export ZONE=us-central1-a
export CLUSTER_NAME="prod-$REGION-yumaeda"
export REPOSITORY=xxxx
export DEPLOYMENT=xxxx
export SERVICE=xxxx
```

### Create GKE cluster
- From GKE 1.12.0, [`f1-micro` machine type is no longer supported](https://stackoverflow.com/questions/61357217/gcloud-kubernetes-in-f1-micro-results-in-node-pools-of-f1-micro-machines-are-no).

```zsh
gcloud container clusters create \
  --preemptible \
  --project=${PROJECT_ID} \
  --machine-type g1-small \
  --num-nodes 3 \
  --disk-size 10 \
  --zone ${ZONE} \
  --cluster-version latest \
  ${CLUSTER_NAME}
```

### Get a list of GKE cluster
```zsh
gcloud container clusters list
```

### Populate `kubeconfig` file (Add a new kubectl context)
```zsh
gcloud container clusters get-credentials ${CLUSTER_NAME} --region ${ZONE} --project ${PROJECT_ID}
```

### Show list of kubectl contexts
```zsh
kubectx
```

### Show namespace for the current kubectl context
```zsh
kubens -c
```

### Switch kubectl context
```zsh
kubectx ${KUBECTL_CONTEXT}
```

### Build simplified manifest
```zsh
kustomize build . > deploy.yaml
```

## Create Deployment & Service
```zsh
kubectl apply -f deploy.yaml
```

## Update Ingress
```zsh
kubectl apply -f basic-ingress.yaml
```

## Get External IP of the load balancer
```zsh
kubectl get ingress ${INGRESS_NAME}
```

## Rollout restarted Deployment
```zsh
kubectl rollout restart deployment/web
```


## Create Deployment (w/ replicas=2)
```zsh
kubectl create deployment ${DEPLOYMENT} --image=${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPOSITORY}/${IMG_NAME}:${IMG_VERSION}

kubectl scale deployment ${DEPLOYMENT} --replicas=2

kubectl autoscale deployment ${DEPLOYMENT} --cpu-percent=80 --min=1 --max=2
```

## Expose Deployment via LoadBalancer
```zsh
kubectl expose deployment ${DEPLOYMENT} --name=${SERVICE} --type=LoadBalancer --port 80 --target-port 8080
```

## List Pods
```sh
kubectl get pods
```

## Misc
### Delete GKE cluster
```zsh
gcloud container clusters delete ${CLUSTER_NAME}
```
### Delete Deployment
```zsh
kubectl delete deployment ${DEPLOYMENT_NAME}
```
### Delete Service
```zsh
kubectl delete service ${SERVICE_NAME}
```
### Delete Ingress
```zsh
kubectl delete ingress ${INGRESS_NAME}
```

## Reference
- https://qiita.com/Esfahan/items/f7d13d7e80848e9b6963
