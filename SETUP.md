# Quick Setup Guide

This guide provides quick setup instructions for CI/CD pipelines.

## 🚀 Quick Start

### Option 1: GitHub Actions Only

1. **Configure Secrets** in GitHub repository:
   - `AZURE_CREDENTIALS` - Azure service principal credentials
   - `AKS_RESOURCE_GROUP` - AKS resource group name
   - `AKS_CLUSTER_NAME` - AKS cluster name
   - `K8S_NAMESPACE` - Kubernetes namespace (default: `default`)

2. **Push to repository** - Workflow will trigger automatically

3. **Monitor deployment** in GitHub Actions tab

### Option 2: ArgoCD Only

1. **Install ArgoCD**:
   ```bash
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

2. **Add repository to ArgoCD**:
   ```bash
   argocd repo add https://github.com/Ahmedhessn/VOTE-APP-V-3 --type git
   ```

3. **Deploy application**:
   ```bash
   kubectl apply -f argocd/application.yaml
   ```

### Option 3: Hybrid (GitHub Actions + ArgoCD)

1. **Setup GitHub Actions** (for CI) - Follow Option 1 steps 1-2
2. **Setup ArgoCD** (for CD) - Follow Option 2 steps 1-3
3. **GitHub Actions** handles testing and validation
4. **ArgoCD** handles deployment automatically

## 📋 Prerequisites Checklist

- [ ] Azure subscription with AKS cluster
- [ ] kubectl configured to access AKS cluster
- [ ] Helm 3.x installed
- [ ] GitHub repository with code
- [ ] Azure service principal created (for GitHub Actions)
- [ ] ArgoCD installed (for ArgoCD option)

## 🔧 Required Secrets

### GitHub Actions Secrets

| Secret Name | Description | Example |
|------------|-------------|---------|
| `AZURE_CREDENTIALS` | Azure service principal JSON | `{"clientId":"...","clientSecret":"..."}` |
| `AKS_RESOURCE_GROUP` | AKS resource group | `my-aks-rg` |
| `AKS_CLUSTER_NAME` | AKS cluster name | `my-aks-cluster` |
| `K8S_NAMESPACE` | Kubernetes namespace | `default` |

### AKS Multi-Environment Secrets

| Secret Name | Description |
|------------|-------------|
| `AKS_STAGING_RESOURCE_GROUP` | Staging resource group |
| `AKS_STAGING_CLUSTER_NAME` | Staging cluster name |
| `AKS_PRODUCTION_RESOURCE_GROUP` | Production resource group |
| `AKS_PRODUCTION_CLUSTER_NAME` | Production cluster name |

## 📁 Project Structure

```
vote-app-chart/
├── .github/
│   └── workflows/
│       ├── ci-cd.yml          # Main CI/CD pipeline
│       └── aks-cicd.yml       # AKS-specific pipeline
├── argocd/
│   ├── application.yaml       # Default ArgoCD app
│   ├── application-staging.yaml
│   ├── application-production.yaml
│   ├── app-of-apps.yaml       # App-of-Apps pattern
│   └── README.md
├── aks/
│   ├── values-staging.yaml    # Staging values
│   ├── values-production.yaml # Production values
│   └── README.md
├── templates/                 # Helm templates
├── Chart.yaml
├── values.yaml
├── CI-CD.md                   # Detailed CI/CD guide
└── SETUP.md                   # This file
```

## 🎯 Deployment Workflows

### GitHub Actions Workflow

```
Push to main/develop
    ↓
Lint & Test
    ↓
Build Chart
    ↓
Deploy to AKS
```

### ArgoCD Workflow

```
Git Repository Change
    ↓
ArgoCD Detects Change
    ↓
Sync to Cluster
    ↓
Deployment Complete
```

## 🔍 Verification

### Check GitHub Actions

1. Go to repository → Actions tab
2. View workflow runs
3. Check job logs

### Check ArgoCD

```bash
# List applications
argocd app list

# Get status
argocd app get vote-app

# View UI
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

### Check Kubernetes

```bash
# Check pods
kubectl get pods -n <namespace>

# Check services
kubectl get svc -n <namespace>

# Check ingress
kubectl get ingress -n <namespace>
```

## 📚 Documentation

- **Detailed CI/CD Guide**: See `CI-CD.md`
- **ArgoCD Setup**: See `argocd/README.md`
- **AKS Configuration**: See `aks/README.md`
- **Helm Chart**: See `README.md`

## 🆘 Troubleshooting

### GitHub Actions Fails

1. Check Azure credentials
2. Verify service principal has AKS access
3. Check workflow logs

### ArgoCD Sync Fails

1. Verify repository access
2. Check application status: `argocd app get vote-app`
3. Review sync logs

### Deployment Issues

1. Check pod status: `kubectl get pods`
2. View pod logs: `kubectl logs <pod-name>`
3. Describe pod: `kubectl describe pod <pod-name>`

## 🔄 Next Steps

1. Customize values files for your environment
2. Configure ingress domains
3. Set up monitoring and alerting
4. Configure backup strategies
5. Review security settings

## 📞 Support

For detailed information, refer to:
- `CI-CD.md` - Complete CI/CD documentation
- `argocd/README.md` - ArgoCD specific guide
- `aks/README.md` - AKS configuration guide

