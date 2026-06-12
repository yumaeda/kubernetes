# Sakabas Next.js Deployment

Kubernetes manifests for deploying the Sakabas Next.js application to GKE.

## Overview

This directory contains Kubernetes manifests for deploying a Next.js application to a GKE cluster using GCE ingress.

## Prerequisites

- Kubernetes cluster (GKE recommended)
- kubectl configured to access the cluster
- GCE ingress controller installed in the cluster

## Namespace

The application is deployed to the `sakabas-nextjs` namespace.

```bash
kubectl apply -f namespace.yaml
```

## Deployment

### Apply All Resources

```bash
kubectl apply -f .
```

### Apply Individual Resources

```bash
kubectl apply -f namespace.yaml
kubectl apply -f configmap.yaml
kubectl apply -f secrets.yaml
kubectl apply -f limitrange.yaml
kubectl apply -f networkpolicy.yaml
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
- `NEXT_PUBLIC_API_URL`: API base URL (from ConfigMap)
- `DATABASE_URL`: Database connection string (from Secret)

### Ingress Configuration

The application is accessible at `https://sakabas.com` and `https://www.sakabas.com`.

TLS is automatically configured via GKE Managed Certificates.

### Resource Limits

Current resource allocation:

- **Requests**: 256Mi memory, 250m CPU
- **Limits**: 512Mi memory, 500m CPU

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
```

### View Logs

```bash
kubectl logs -f deployment/sakabas-nextjs -n sakabas-nextjs
```

### Access Application

The application is accessible at `https://sakabas.com`

## Security

- Container runs as non-root user
- Read-only root filesystem enabled
- Privilege escalation disabled
- All capabilities dropped
- Secrets managed via Kubernetes Secrets
- Network policies restrict traffic

## Troubleshooting

### Check Events

```bash
kubectl describe pod -n sakabas-nextjs
kubectl get events -n sakabas-nextjs --sort-by='.lastTimestamp'
```

### Debug Configuration

```bash
kubectl get configmap sakabas-nextjs-config -n sakabas-nextjs -o yaml
kubectl get secrets -n sakabas-nextjs
```

## Uninstall

```bash
kubectl delete -f .
```
