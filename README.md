# JournAI GitOps Repository

This repository contains the GitOps configuration for deploying the JournAI application on AWS EKS using ArgoCD and Terraform.

## 🏗️ Infrastructure Components

- **AWS EKS** - Managed Kubernetes clusters
- **AWS RDS** - PostgreSQL database
- **AWS S3** - Object storage for file uploads
- **AWS IAM** - Fine-grained access control
- **Terraform** - Infrastructure as Code
- **ArgoCD** - GitOps continuous delivery

## 📁 Structure

```
GitOps/
├── applicationset.yaml           # ArgoCD ApplicationSet for all environments
└── environments/
    ├── dev/
    │   └── values.yaml          # Dev environment overrides
    ├── staging/
    │   └── values.yaml          # Staging environment overrides
    └── prod/
        └── values.yaml          # Production environment overrides
```

## 🚀 Prerequisites

Before deploying with ArgoCD, you need to create Kubernetes secrets for each environment.

### Create Secrets

The application requires the following secrets for each environment:
- `JWT_SECRET` - JWT token secret
- `GEMINI_API_KEY` - Google Gemini API key
- `OPENAI_API_KEY` - OpenAI API key
- `DB_PASSWORD` - Database password

#### Dev Environment:
```bash
kubectl create secret generic journai-secrets-dev \
  --from-literal=JWT_SECRET='your-dev-jwt-secret' \
  --from-literal=GEMINI_API_KEY='your-gemini-key' \
  --from-literal=OPENAI_API_KEY='your-openai-key' \
  --from-literal=DB_PASSWORD='your-dev-db-password' \
  -n dev-journai
```

#### Staging Environment:
```bash
kubectl create secret generic journai-secrets-staging \
  --from-literal=JWT_SECRET='your-staging-jwt-secret' \
  --from-literal=GEMINI_API_KEY='your-gemini-key' \
  --from-literal=OPENAI_API_KEY='your-openai-key' \
  --from-literal=DB_PASSWORD='your-staging-db-password' \
  -n staging-journai
```

#### Production Environment:
```bash
kubectl create secret generic journai-secrets-prod \
  --from-literal=JWT_SECRET='your-prod-jwt-secret' \
  --from-literal=GEMINI_API_KEY='your-gemini-key' \
  --from-literal=OPENAI_API_KEY='your-openai-key' \
  --from-literal=DB_PASSWORD='your-prod-db-password' \
  -n prod-journai
```

### Generate Strong Secrets:
```bash
# Generate JWT secret
openssl rand -base64 32

# Generate DB password
openssl rand -base64 24
```

## 📝 Deploy with ArgoCD

1. **Install ArgoCD ApplicationSet:**
```bash
kubectl apply -f applicationset.yaml -n argocd
```

2. **Verify Applications Created:**
```bash
kubectl get applications -n argocd
```

You should see:
- `journai-dev`
- `journai-staging`
- `journai-prod`

3. **Sync Applications:**

Dev and Staging will auto-sync. For production, you need to manually sync:
```bash
argocd app sync journai-prod
```

## 🔄 Environment Configuration

### Dev Environment
- **Namespace:** `dev-journai`
- **Branch:** `dev`
- **Replicas:** 1 (backend), 1 (frontend)
- **Auto-sync:** ✅ Enabled
- **Domain:** `dev.journai.local`

### Staging Environment
- **Namespace:** `staging-journai`
- **Branch:** `staging`
- **Replicas:** 2 (backend), 2 (frontend)
- **Auto-sync:** ✅ Enabled
- **Domain:** `staging.journai.local`

### Production Environment
- **Namespace:** `prod-journai`
- **Branch:** `main`
- **Replicas:** 2 (backend), 2 (frontend)
- **Auto-sync:** ❌ Manual sync required
- **Domain:** `journai.com`

## 🔐 Secrets Management

Each environment uses a pre-created Kubernetes secret:
- Dev: `journai-secrets-dev`
- Staging: `journai-secrets-staging`
- Prod: `journai-secrets-prod`

**Important:** Create these secrets BEFORE deploying the application.

## 🛠️ Troubleshooting

### Check Application Status:
```bash
argocd app get journai-dev
argocd app get journai-staging
argocd app get journai-prod
```

### Check Secrets:
```bash
kubectl get secret journai-secrets-dev -n dev-journai
kubectl get secret journai-secrets-staging -n staging-journai
kubectl get secret journai-secrets-prod -n prod-journai
```

### View Application Logs:
```bash
kubectl logs -n dev-journai -l app=backend
kubectl logs -n dev-journai -l app=frontend
```

### Force Sync:
```bash
argocd app sync journai-dev --force
```

## 📚 Related Repositories

- **Application Code:** [JournAI](https://github.com/noylevi/JournAI)
- **Helm Chart:** Located in `JournAI/JournAI-Chart/`
