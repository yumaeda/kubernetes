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
export REGION=us-central1 &&
export ZONE=us-central1-a &&
export CLUSTER_NAME="prod-$REGION-yumaeda"
```
### Execute below command
```zsh
gcloud container clusters create \
  --preemptible \
  --project=${PROJECT_ID} \
  --machine-type=e2-micro \
  --num-nodes=2 \
  --max-nodes=3 \
  --disk-size=10 \
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

&nbsp;

## Deploy
### Move to the manifests directory
```zsh
cd manifests
```
### Create Namespace
```zsh
kubectl apply -f namespace.yaml
```
### Get a list of Namespaces
```zsh
kubectl get namespaces
```
### Create LimitRange
```zsh
kubectl apply -f limitrange.yaml
```
### Get the LimitRange for the sakaba-link Namespace
```zsh
kubectl describe limitranges --namespace=sakaba-link
```
### Build simplified manifest
```zsh
kustomize build . > deploy.yaml
```
### Create Deployment & Service
```zsh
kubectl apply -f deploy.yaml
```
### List Pods
```zsh
kubectl get pods --namespace=sakaba-link
```
### Create managed certificate
```zsh
kubectl apply -f web-cert.yaml
```
### Create Ingress
```zsh
kubectl apply -f basic-ingress.yaml
```
### Get External IP of the load balancer
```zsh
kubectl describe ingress basic-ingress --namespace=sakaba-link
```
### Check the status of the certificate
```zsh
gcloud beta compute ssl-certificates list
```
### Create a ServiceAccount
```zsh
kubectl apply -f serviceaccount.yaml
```
### Get a list of ServiceAccounts
```zsh
kubectl get serviceaccounts --namespace=ramen-mania
```
### Create CronJob
```zsh
kubectl apply -f cronjob.yaml
```
### Check a CronJob's configuration
```zsh
kubectl describe cronjob terminated-pod-cleaner --namespace=ramen-mania
```

&nbsp;

## Delete
### Move to the manifests directory
```zsh
cd manifests
```
### Delete Ingress
```zsh
kubectl delete -f basic-ingress.yaml
```
### Delete Google-managed certificate
```zsh
kubectl delete -f web-cert.yaml
```
### Delete Service
```zsh
kubectl delete service web --namespace=ramen-mania
```
### Delete Deployment
```zsh
kubectl delete deployment web --namespace=ramen-mania
```
### Delete a CronJob
```zsh
kubectl delete -f cronjob.yaml
```
### Delete the ServiceAccount
```zsh
kubectl delete -f serviceaccount.yaml
```
### Delete the LimitRange for the ramen-mania Namespace
```zsh
kubectl delete -f limitrange.yaml --namespace=ramen-mania
```
### Delete the Namespace
```zsh
kubectl delete -f namespace.yaml
```
### Delete GKE cluster
```zsh
gcloud container clusters delete ${CLUSTER_NAME}
```

&nbsp;

## Misc
### Reduce the number of nodes in the k8s cluster
```zsh
gcloud container node-pools create ramen-mania-pool \
  --cluster ${CLUSTER_NAME} --zone ${ZONE} \
  --machine-type=e2-small --num-nodes=2 --max-nodes=3 \
  --disk-size=10 --preemptible
```
```zsh
gcloud container node-pools delete default-pool --cluster ${CLUSTER_NAME} --zone ${ZONE}
```
```zsh
kubectl get pod --all-namespaces
```
### Port-forward service to localhost
```zsh
kubectl port-forward svc/web 8080:8080
```
- Access http://localhost:8080/ to make sure that port-forwarding works.
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
kubectl top pod -n ramen-mania
```
### Delete the current pod
```zsh
kubectl delete pod ${POD_NAME}
```
- New pod with different name is automatically created.
### Rollout restarted Deployment
```zsh
kubectl rollout restart deployment/web
```
- Old pod is replaced by new pod.
### Delete all pods
```zsh
kubectl delete --all pods --namespace=default
kubectl delete --all pods --namespace=ramen-mania
```

&nbsp;

## Datadog
### Set environment variable for API Key
```zsh
export DATADOG_API_KEY=xxxx
```
### Install Datadog Agent
```zsh
helm repo add datadog https://helm.datadoghq.com
helm repo update
helm install datadog-agent -f datadog-values.yaml -n ramen-mania --set datadog.site='datadoghq.com' --set datadog.apiKey=${DATADOG_API_KEY} datadog/datadog
```
### Warning
```zsh
###################################################################################
####   WARNING: Cluster-Agent should be deployed in high availability mode     ####
###################################################################################

The Cluster-Agent should be in high availability mode because the following features
are enabled:
* Admission Controller

To run in high availability mode, our recommendation is to update the chart
configuration with:
* set clusterAgent.replicas value to 2 replicas .
* set clusterAgent.createPodDisruptionBudget to true.
```
### Uinstall Datadog Operator
```zsh
helm uninstall datadog-agent -n ramen-mania
kubectl delete configmap datadog-agent-leader-election -n ramen-mania
kubectl delete configmap datadog-agenttoken -n ramen-mania
kubectl delete configmap datadog-cluster-id -n ramen-mania
```

&nbsp;

## Istio
### Preparation
```zsh
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo update
```
### Install
```zsh
kubectl create namespace istio-system
```
```zsh
helm install istio-base istio/base -n istio-system
```
```zsh
helm install istiod istio/istiod -n istio-system --wait
```
```zsh
kubectl create namespace istio-ingress
```
```zsh
kubectl label namespace istio-ingress istio-injection=enabled
```
```zsh
helm install istio-ingress istio/gateway -n istio-ingress --wait
```
```zsh
helm list --namespace=istio-system
```
```zsh
kubectl get pods --namespace=istio-system
```
### Uinstall
```zsh
helm uninstall istio-ingress --namespace=istio-system
```
```zsh
helm uninstall istiod --namespace=istio-system
```
```zsh
helm uninstall istio-base --namespace=istio-system
```
```zsh
kubectl delete namespace istio-system
```
```zsh
kubectl delete namespace istio-ingress
```
```zsh
kubectl get crd -oname | grep --color=never 'istio.io' | xargs kubectl delete
```

&nbsp;

## Reference
- https://docs.datadoghq.com/containers/kubernetes/installation/?tab=operator
- https://istio.io/latest/docs/setup/install/helm/
- https://cloud.google.com/load-balancing/docs/ssl-certificates/troubleshooting?&_ga=2.28635196.-1210616006.1659620185#certificate-managed-status
- https://cloud.google.com/kubernetes-engine/docs/how-to/managed-certs
- https://qiita.com/tontoko/items/33faead6bb14370ecb17
