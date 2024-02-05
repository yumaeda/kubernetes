# kubernetes
## Preparations
### [Setup GCP](https://github.com/yumaeda/gcp/blob/main/README.md)
### Install packages
```sh
brew install \
  helm \
  kubectl \
  kubectx \
  kustomize
```
### Enable `container.googleapis.com`
```zsh
gcloud services enable container.googleapis.com
```

&nbsp;

## Create GKE cluster
### Set environment variables
```zsh
export PROJECT_ID=xxxx
```
```zsh
export REGION=asia-northeast1 &&
export ZONE=asia-northeast1-b &&
export CLUSTER_NAME="prod-$REGION-sakabas"
```
### Execute below command
```zsh
gcloud container clusters create \
  --preemptible \
  --project=${PROJECT_ID} \
  --machine-type=e2-small \
  --num-nodes=3 \
  --max-nodes=4 \
  --disk-size=10 \
  --network=sakabas-tokyo-vpc \
  --subnetwork=subnet-asia-northeast-172 \
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

### Delete GKE cluster
```zsh
gcloud container clusters delete ${CLUSTER_NAME}
```

&nbsp;

## Trouble-shooting
### Increase the number of nodes
- Create a new node-pool with more nodes
```zsh
gcloud container node-pools create ${NEW_POOL_NAME} \
  --cluster ${CLUSTER_NAME} --zone ${ZONE} \
  --machine-type=e2-small --num-nodes=3 \
  --disk-size=10 --preemptible
```
- Delete the old node-pool
```zsh
gcloud container node-pools delete ${OLD_POOL_NAME} --cluster ${CLUSTER_NAME} --zone ${ZONE}
```
### Execute command within pods
```zsh
kubectl exec --stdin --tty ${POD_NAME} -- /bin/sh
```
### Show logs for the pod
```zsh
kubectl logs ${POD_NAME}
```
### See current resource usage
```zsh
kubectl top pod --all-namespaces
```
### Delete the current pod
```zsh
kubectl delete pod ${POD_NAME}
```
### Delete all pods
```zsh
kubectl delete --all pods --all-namespaces
```

&nbsp;

## Misc
```zsh
kubectl get pod --all-namespaces
```
### Port-forward service to localhost
```zsh
kubectl port-forward svc/web 8080:8080
```
- Access http://localhost:8080/ to make sure that port-forwarding works.
- New pod with different name is automatically created.
### Rollout restarted Deployment
```zsh
kubectl rollout restart deployment/web
```
- Old pod is replaced by new pod.

&nbsp;

## Reference
- https://docs.datadoghq.com/containers/kubernetes/installation/?tab=operator
- https://istio.io/latest/docs/setup/install/helm/
- https://cloud.google.com/load-balancing/docs/ssl-certificates/troubleshooting?&_ga=2.28635196.-1210616006.1659620185#certificate-managed-status
- https://cloud.google.com/kubernetes-engine/docs/how-to/managed-certs
- https://qiita.com/tontoko/items/33faead6bb14370ecb17
