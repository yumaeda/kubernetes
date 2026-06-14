# Sakabas Next.js Deployment

Kubernetes manifests for deploying the Sakabas Next.js application to GKE.

## Overview

This directory contains Kubernetes manifests for deploying a Next.js application to a GKE cluster using GCE ingress.

## Prerequisites

- Kubernetes cluster (GKE recommended)
- kubectl configured to access the cluster
- GCE ingress controller installed in the cluster

## Files

The following manifests are included:

- `namespace.yaml` - Namespace definition
- `service-account.yaml` - Service account for pod identity
- `deployment.yaml` - Deployment configuration (2 replicas)
- `service.yaml` - Service definition
- `managed-certificate.yaml` - GCP managed TLS certificate
- `ingress.yaml` - Ingress with GCP load balancer
- `limitrange.yaml` - Resource limits for the namespace

## Deployment

### Apply All Resources

```bash
kubectl apply -f .
```

### Apply Individual Resources

```bash
kubectl apply -f namespace.yaml
kubectl apply -f service-account.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f managed-certificate.yaml
kubectl apply -f ingress.yaml
```

## Configuration

### Environment Variables

Configure environment variables in the deployment:

- `NODE_ENV`: Set to `production`
- `PORT`: Application port (default: 3000)

### Ingress Configuration

The application is accessible at `https://sakabas.com`.

TLS is automatically configured via GKE Managed Certificates.

### Resource Limits

Current resource allocation:

- **Requests**: 256Mi memory, 250m CPU
- **Limits**: 512Mi memory, 500m CPU

### Security Context

The container runs with the following security settings:

- `runAsNonRoot: true` - Container must run as non-root user
- `readOnlyRootFilesystem: true` - Read-only root filesystem
- `allowPrivilegeEscalation: false` - Privilege escalation disabled
- `capabilities.drop: [ALL]` - All capabilities dropped
- `runAsUser: 1001` - Run as specific non-root user
- `seccompProfile.type: RuntimeDefault` - Default seccomp profile

## Scaling

To scale the deployment:

```bash
kubectl scale deployment sakabas-nextjs -n sakabas-nextjs --replicas=3
```

## Monitoring

### Check Deployment Status

```bash
kubectl get pods -n sakabas-nextjs
kubectl get deployment -n sakabas-nextjs
kubectl get svc -n sakabas-nextjs
kubectl get ingress -n sakabas-nextjs
```

### View Logs

```bash
kubectl logs -f deployment/sakabas-nextjs -n sakabas-nextjs
```

### Access Application

The application is accessible at `https://sakabas.com`

## Troubleshooting

### Check Events

```bash
kubectl describe pod -n sakabas-nextjs
kubectl get events -n sakabas-nextjs --sort-by='.lastTimestamp'
```

### Debug Configuration

```bash
kubectl get serviceaccount sakabas-nextjs -n sakabas-nextjs -o yaml
kubectl get secrets -n sakabas-nextjs
```

## Uninstall

```bash
kubectl delete -f .
```
