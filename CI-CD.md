# CI/CD Setup Guide

This document provides comprehensive instructions for setting up CI/CD pipelines for the Vote App Helm chart using GitHub Actions and ArgoCD.

## Overview

This project supports two CI/CD approaches:

1. **GitHub Actions CI/CD** - Complete CI/CD pipeline using GitHub Actions
2. **ArgoCD GitOps** - GitOps-based continuous deployment using ArgoCD
3. **AKS CI/CD** - Azure Kubernetes Service specific CI/CD workflows

## Architecture

```
┌─────────────────┐
│   GitHub Repo   │
│  (Source Code)  │
└────────┬────────┘
         │
         ├─────────────────┐
         │                 │
         ▼                 ▼
┌─────────────────┐  ┌──────────────┐
│ GitHub Actions  │  │   ArgoCD     │
│   (CI/CD)       │  │  (GitOps)    │
└────────┬────────┘  └──────┬───────┘
         │                  │
         └──────────┬───────┘
                    │
                    ▼
            ┌──────────────┐
            │  AKS Cluster │
            │  (Kubernetes)│
            └──────────────┘
```

## Option 1: GitHub Actions CI/CD

### Features

- Automated linting and testing
- Helm chart validation
- Automated deployment to AKS
- Multi-environment support (staging/production)
- Manual workflow dispatch

### Setup Steps

#### 1. Configure GitHub Secrets

Navigate to your repository: `Settings > Secrets and variables > Actions`

Add the following secrets:

**Azure Credentials:**
```
AZURE_CREDENTIALS
```
Create using:
```bash
az ad sp create-for-rbac --name "github-actions-sp" \
  --role contributor \
  --scopes /subscriptions/<subscription-id> \
  --sdk-auth
```

**AKS Cluster Information:**
```
AKS_RESOURCE_GROUP=<your-resource-group>
AKS_CLUSTER_NAME=<your-cluster-name>
K8S_NAMESPACE=default
```

**Image Registry (Optional):**
```
POSTGRES_IMAGE_REPO=01061875164/postgres15-alpine
POSTGRES_IMAGE_TAG=v3
REDIS_IMAGE_REPO=01061875164/redis
REDIS_IMAGE_TAG=v1
VOTE_IMAGE_REPO=01061875164/vote-app
VOTE_IMAGE_TAG=v1
RESULT_IMAGE_REPO=01061875164/result
RESULT_IMAGE_TAG=v3
WORKER_IMAGE_REPO=01061875164/worker
WORKER_IMAGE_TAG=v2
```

#### 2. Grant AKS Access

```bash
# Get AKS cluster resource ID
AKS_ID=$(az aks show --resource-group <resource-group> --name <cluster-name> --query id -o tsv)

# Get service principal client ID
SP_CLIENT_ID=$(az ad sp list --display-name "github-actions-sp" --query [0].appId -o tsv)

# Grant cluster admin role
az role assignment create \
  --assignee $SP_CLIENT_ID \
  --role "Azure Kubernetes Service Cluster Admin Role" \
  --scope $AKS_ID
```

#### 3. Workflow Files

The following workflows are available:

- `.github/workflows/ci-cd.yml` - Main CI/CD pipeline
- `.github/workflows/aks-cicd.yml` - AKS-specific CI/CD pipeline

#### 4. Trigger Deployment

**Automatic:**
- Push to `main` branch → Deploys to production
- Push to `develop` branch → Deploys to staging

**Manual:**
- Go to `Actions` tab
- Select workflow
- Click `Run workflow`
- Choose branch and environment

### Workflow Details

#### CI Pipeline (`ci-cd.yml`)

1. **Lint** - Validates Helm chart syntax
2. **Test** - Tests chart installation in KinD cluster
3. **Build** - Packages Helm chart
4. **Deploy** - Deploys to AKS cluster

#### AKS Pipeline (`aks-cicd.yml`)

1. **Validate** - Validates chart with environment-specific values
2. **Build** - Packages Helm chart
3. **Deploy Staging** - Deploys to staging environment
4. **Deploy Production** - Deploys to production environment

## Option 2: ArgoCD GitOps

### Features

- GitOps methodology
- Automated sync from Git
- Self-healing capabilities
- Multi-environment management
- Rollback capabilities

### Setup Steps

#### 1. Install ArgoCD

```bash
# Create namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for installation
kubectl wait --for=condition=available --timeout=300s deployment/argocd-server -n argocd
```

#### 2. Access ArgoCD

```bash
# Port forward
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Get initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Login
argocd login localhost:8080
```

#### 3. Add Repository

**Public Repository:**
```bash
argocd repo add https://github.com/Ahmedhessn/VOTE-APP-V-3 --type git
```

**Private Repository:**
```bash
# Using SSH
argocd repo add git@github.com:Ahmedhessn/VOTE-APP-V-3.git \
  --ssh-private-key-path ~/.ssh/id_rsa

# Using HTTPS
argocd repo add https://github.com/Ahmedhessn/VOTE-APP-V-3 \
  --username <github-username> \
  --password <github-token>
```

#### 4. Deploy Applications

**Using App-of-Apps Pattern:**
```bash
kubectl apply -f argocd/app-of-apps.yaml
```

**Individual Applications:**
```bash
# Default
kubectl apply -f argocd/application.yaml

# Staging
kubectl apply -f argocd/application-staging.yaml

# Production
kubectl apply -f argocd/application-production.yaml
```

#### 5. Verify Deployment

```bash
# List applications
argocd app list

# Get application status
argocd app get vote-app

# Sync application
argocd app sync vote-app
```

### Application Manifests

- `argocd/application.yaml` - Default application
- `argocd/application-staging.yaml` - Staging environment
- `argocd/application-production.yaml` - Production environment
- `argocd/app-of-apps.yaml` - App-of-Apps pattern

## Option 3: Hybrid Approach (Recommended)

Combine GitHub Actions for CI and ArgoCD for CD:

### Workflow

1. **GitHub Actions (CI):**
   - Lint Helm chart
   - Test chart installation
   - Build and package chart
   - Push chart to registry or update Git

2. **ArgoCD (CD):**
   - Monitor Git repository
   - Automatically sync changes
   - Deploy to Kubernetes cluster

### Benefits

- **CI**: Automated testing and validation
- **CD**: GitOps-based deployment
- **Separation**: Clear separation of concerns
- **Flexibility**: Can use either approach independently

## Environment-Specific Configurations

### Staging (`aks/values-staging.yaml`)

- Lower resource limits
- Single replica for most services
- Staging domain configuration
- Development-friendly settings

### Production (`aks/values-production.yaml`)

- Higher resource limits
- Multiple replicas
- Production domain configuration
- Autoscaling enabled
- Premium storage class

## Monitoring and Troubleshooting

### GitHub Actions

**View Workflow Runs:**
- Go to `Actions` tab in GitHub
- Click on workflow run
- View logs for each job

**Common Issues:**
- Authentication failures → Check Azure credentials
- Cluster access issues → Verify service principal permissions
- Deployment failures → Check pod logs in AKS

### ArgoCD

**Check Application Status:**
```bash
argocd app get vote-app
```

**View Sync Status:**
```bash
argocd app sync vote-app --dry-run
```

**Common Issues:**
- Sync failures → Check Git repository access
- Health issues → Check pod status in cluster
- Out of sync → Review differences and sync

## Best Practices

1. **Use separate namespaces** for each environment
2. **Enable resource limits** to prevent resource exhaustion
3. **Use secrets management** for sensitive data
4. **Enable monitoring** and alerting
5. **Regular backups** of persistent volumes
6. **Test in staging** before production deployment
7. **Use semantic versioning** for chart versions
8. **Document changes** in commit messages

## Security Considerations

1. **Secrets Management:**
   - Use Azure Key Vault or Kubernetes Secrets
   - Never commit secrets to Git
   - Rotate credentials regularly

2. **Network Policies:**
   - Implement network policies for pod-to-pod communication
   - Restrict ingress access

3. **RBAC:**
   - Use least privilege principle
   - Separate service accounts per application

4. **Image Security:**
   - Scan images for vulnerabilities
   - Use trusted image registries
   - Implement image signing

## Rollback Procedures

### GitHub Actions

```bash
# Rollback using Helm
helm rollback vote-app <revision-number> -n <namespace>
```

### ArgoCD

```bash
# Rollback to previous sync
argocd app rollback vote-app

# Rollback to specific revision
argocd app rollback vote-app <revision-id>
```

## Additional Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [Helm Documentation](https://helm.sh/docs/)
- [AKS Documentation](https://docs.microsoft.com/azure/aks/)

## Support

For issues or questions:
1. Check workflow logs in GitHub Actions
2. Review ArgoCD application status
3. Check Kubernetes pod logs
4. Review this documentation

