# CLAUDE Configuration for Kubernetes Project

## Project Overview

This repository contains Kubernetes manifests for deploying the Sakabas application to Google Kubernetes Engine (GKE). The application consists of two main services:

1. **sakabas-api** - Backend API service
2. **sakabas-nextjs** - Next.js frontend application

## Environment Setup

### Prerequisites

- **GKE Cluster**: `prod-asia-northeast1-sakabas` in `asia-northeast1` region
- **Tools**: `kubectl`, `kubectx`, `kubens`, `helm`
- **Auth**: `gke-gcloud-auth-plugin` for cluster authentication

### Cluster Access

```bash
gcloud container clusters get-credentials prod-asia-northeast1-sakabas \
  --region asia-northeast1 \
  --project <PROJECT_ID>
```

## Manifest Structure

Each service has its own directory under `manifests/`:

- `manifests/sakabas_api/` - API deployment (2 replicas)
- `manifests/sakabas_nextjs/` - Next.js app (2 replicas)

Each directory contains:
- `namespace.yaml` - Namespace definition
- `deployment.yaml` - Application deployment
- `service.yaml` - Service definition
- `ingress.yaml` - Ingress configuration
- `managed-certificate.yaml` - TLS certificates
- `limitrange.yaml` - Resource limits for the namespace
- `service-account.yaml` - Service account for pod identity

## Deployment Commands

```bash
# Deploy all resources in a service directory
kubectl apply -f manifests/<service>/

# Check deployment status
kubectl get pods -n <namespace>
kubectl get deployments -n <namespace>

# View logs
kubectl logs -f deployment/<deployment-name> -n <namespace>

# Restart deployment
kubectl rollout restart deployment/<deployment-name> -n <namespace>
```

## Resource Allocation

| Service | Replicas | Memory Request | CPU Request | Memory Limit | CPU Limit |
|---------|----------|----------------|-------------|--------------|-----------|
| sakabas-api | 2 | 100Mi (LimitRange default) | 3m (LimitRange default) | 100Mi (LimitRange default) | 3m (LimitRange default) |
| sakabas-nextjs | 2 | 128Mi | 125m | 128Mi | 125m |

## Service-Specific Guidelines

### sakabas-api

- **Image**: `us-central1-docker.pkg.dev/hello-world-352201/sakabas-api/sakabas-api:latest`
- **Port**: 8080 (container) / 80 (service, NodePort)
- **Dependencies**: TiDB, AWS S3
- **Secrets Required**: `tidb-pwd`, `tidb-host`, `tidb-name`, `tidb-user`, `s3-id`, `s3-secret`, `s3-region`

### sakabas-nextjs

- **Image**: `us-central1-docker.pkg.dev/hello-world-352201/sakabas-nextjs/sakabas-nextjs:latest`
- **Port**: 3000
- **Environment**: NODE_ENV=production
- **Service Account**: `sakabas-nextjs`
- **Ingress**: GCE load balancer with managed certificate

## Security Context

All containers run with:
- Non-root user (`runAsNonRoot: true`)
- Read-only root filesystem (`readOnlyRootFilesystem: true`)
- Privilege escalation disabled (`allowPrivilegeEscalation: false`)
- Seccomp profile (`RuntimeDefault`)

## Troubleshooting

### Check pod status
```bash
kubectl describe pod <pod-name> -n <namespace>
```

### View events
```bash
kubectl get events -n <namespace> --sort-by='.lastTimestamp'
```

### Port-forward for local testing
```bash
kubectl port-forward svc/<service-name> <local-port>:<service-port> -n <namespace>
```

## GCP Resources

- **Managed Certificates**: Google-managed TLS certificates for domains
- **Ingress**: Cloud Load Balancing with SSL termination
- **Artifacts**: Docker images stored in GCR (`us-central1-docker.pkg.dev`)

## Reference

- [GKE Managed Certificates](https://cloud.google.com/kubernetes-engine/docs/how-to/managed-certs)
- [Kubernetes Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- [GCE Ingress](https://cloud.google.com/kubernetes-engine/docs/concepts/ingress)
