# kubernetes
## Preparations
### Install
```sh
brew install kubectl
brew install kubectx
```

### [Setup GCP](https://github.com/yumaeda/gcp/blob/main/README.md)

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

## Create GKE cluster
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

## Get a list of GKE cluster
```zsh
gcloud container clusters list
```

## Populate `kubeconfig` file
```zsh
gcloud container clusters get-credentials ${CLUSTER_NAME} --region ${REGION} --project ${PROJECT_ID}
```

## List Clusters
```sh
kubectx
```

## Show Current Context's Cluster
```sh
kubectx -c
```

## Show Current Context's Namespace
```sh
kubens -c
```

## Switch
- To switch your context to one of the clusters displayed by `bubectx`.
```sh
kubectx ${CLUSTER_CONTEXT}
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
