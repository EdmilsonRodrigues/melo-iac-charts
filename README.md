# melo-iac Charts & Manifests

This repository contains the Helm charts, Custom Resource Definitions (CRDs), and RBAC manifests required to deploy the **melo-iac** platform onto a Kubernetes cluster.

---

## Directory Structure

```text
melo-iac-charts/
├── crds/
│   └── melo-iac.io_terraformapps.yaml    # Custom Resource Definition
├── helm/
│   └── melo-iac/                         # Helm chart for melo-iac core engine
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
│           ├── controller-deployment.yaml
│           ├── api-deployment.yaml
│           ├── minio-deployment.yaml
│           └── rbac.yaml
└── examples/
    └── sample-terraform-app.yaml        # Example TerraformApp CR manifest
```

---

## Custom Resource Definition (`TerraformApp`)

The central resource managed by `melo-iac` is the `TerraformApp` CRD:

```yaml
apiVersion: melo-iac.io/v1alpha1
kind: TerraformApp
metadata:
  name: production-vpc
  namespace: melo-system
spec:
  repository: "https://github.com/my-org/terraform-infra.git"
  branch: "main"
  path: "environments/prod/vpc"
  syncInterval: "5m"
  autoApply: false # If false, generates plan and waits for approval
  terraformVersion: "1.16.0"
  backendConfig:
    bucket: "my-company-tf-states"
    key: "prod/vpc.tfstate"
    region: "us-east-1"
```

---

## Installation Guide

### Option 1: Quick Install via Helm

1. **Install CRDs manually:**
   ```bash
   kubectl apply -f crds/
   ```

2. **Deploy the Melo Platform using Helm:**
   ```bash
   helm install melo-iac ./helm/melo-iac \
     --namespace melo-system \
     --create-namespace \
     --set api.ingress.enabled=true \
     --set api.ingress.host="melo-api.yourdomain.com"
   ```

### Option 2: Deploy Sample Application

To test the controller with a sample repository, apply the example manifest:

```bash
kubectl apply -f examples/sample-terraform-app.yaml
```

Check reconciliation status:

```bash
kubectl get terraformapps -n melo-system
```

