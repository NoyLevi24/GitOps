# JournAI GitOps Repository

This repository contains the GitOps configuration for deploying the JournAI application on AWS EKS using ArgoCD and Terraform with AWS Secrets Manager integration.

## 🏗️ Infrastructure Components

- **AWS EKS** - Managed Kubernetes clusters
- **AWS RDS** - PostgreSQL database
- **AWS S3** - Object storage for file uploads
- **AWS Secrets Manager** - Secure secrets storage and rotation
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

Before deploying with ArgoCD, ensure the following are in place:

### AWS Secrets Manager Setup

1. **Create AWS Secrets Manager secrets** for each environment:
   - Database credentials
   - API keys (Gemini, OpenAI)
   - JWT secrets
   - Application-specific secrets

2. **Secret Naming Convention:**
   ```
   journai/dev/config
   journai/staging/config
   journai/prod/config
   ```

3. **IAM Permissions:**
   Ensure the EKS node role has permissions to access AWS Secrets Manager:
   ```json
   {
     "Effect": "Allow",
     "Action": [
       "secretsmanager:GetSecretValue",
       "secretsmanager:DescribeSecret"
     ],
     "Resource": "arn:aws:secretsmanager:*:*:secret:journai/*"
   }
   ```

### Generate Strong Secrets:
```bash
# Generate JWT secret
openssl rand -base64 32

# Generate DB password
openssl rand -base64 24
```

### ArgoCD Installation

Ensure ArgoCD is installed and configured in your EKS cluster.

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
- **Auto-sync:** ✅ Enabled (with self-heal)
- **Domain:** `journai.com`

## 🔐 Secrets Management

JournAI uses **AWS Secrets Manager** for secure secrets storage and management. Secrets are automatically injected into Kubernetes pods using the Secrets Store CSI Driver.

### AWS Secrets Manager Integration

The application automatically retrieves the following secrets from AWS Secrets Manager:
- **Database credentials** (RDS username/password)
- **API keys** (Gemini, OpenAI)
- **JWT secrets**
- **Application secrets**

### Secret Names in AWS Secrets Manager

Each environment uses prefixed secret names:
- Dev: `journai/dev/*`
- Staging: `journai/staging/*`
- Production: `journai/prod/*`

### Kubernetes Secret Integration

The Helm chart is configured to use existing Kubernetes secrets:
```yaml
secrets:
  useExistingSecret: true
  existingSecretName: "journai-secrets"
```

The Secrets Store CSI Driver automatically creates and syncs these Kubernetes secrets from AWS Secrets Manager.

**Important:** Ensure AWS Secrets Manager secrets are created BEFORE deploying the application.

## 🛠️ Troubleshooting

### Check Application Status:
```bash
argocd app get journai-dev
argocd app get journai-staging
argocd app get journai-prod
```

### Check Secrets:
```bash
# Check Kubernetes secrets (auto-synced from AWS Secrets Manager)
kubectl get secret journai-secrets -n dev-journai
kubectl get secret journai-secrets -n staging-journai
kubectl get secret journai-secrets -n prod-journai

# Check AWS Secrets Manager secrets
aws secretsmanager describe-secret --secret-id journai/dev/config
aws secretsmanager describe-secret --secret-id journai/staging/config
aws secretsmanager describe-secret --secret-id journai/prod/config
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

- **Application Code:** [JournAI](https://github.com/NoyLevi24/JournAI.git)
- **Terraform Infrastructure:** [Terraform](https://github.com/NoyLevi24/Terraform.git)
- **Helm Chart:** Located in `JournAI/JournAI-Chart/` within the application repository

## 🔗 Repository Structure

This GitOps repository references:
- **Source Repository:** `https://github.com/NoyLevi24/JournAI.git`
- **Target Branch:** `main`
- **Chart Path:** `JournAI-Chart`

The ApplicationSet automatically deploys the Helm chart from the source repository using environment-specific values from this GitOps repository.
