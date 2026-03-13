# Hello World DevOps on EKS

End-to-end CI/CD pipeline deploying a Flask microservice to AWS EKS using 
Terraform, GitHub Actions, and Helm — with failure simulation built in.

## Architecture
```Developer → GitHub → GitHub Actions → Amazon ECR
                          ↓
                      Terraform (VPC + EKS)
                          ↓
                       Helm deploy
                          ↓
              ┌─────── EKS Cluster ────────────┐
              │  Flask app → Prometheus → Grafana │
              └────────────────────────────────┘
```

## Pipeline flow

1. Developer pushes code to GitHub
2. GitHub Actions triggers — runs tests
3. Docker image is built and pushed to Amazon ECR
4. Terraform provisions the VPC and EKS cluster
5. Helm pulls the image from ECR and deploys to EKS
6. Prometheus scrapes metrics from the Flask app
7. Grafana visualises dashboards and alerts

## Project structure
```
.
├── app/                        # Flask hello-world microservice
├── Dockerfile                  # Container image build
├── .github/workflows/cicd.yml  # GitHub Actions CI/CD workflow
├── terraform/                  # AWS infrastructure (VPC + EKS module)
└── helm/hello-world/           # Helm chart for Kubernetes deployment
```

## Prerequisites

- AWS CLI configured with appropriate permissions
- Terraform >= 1.0
- kubectl
- Helm >= 3.0
- Docker

## Local quick run
```bash
docker build -t hello-world:local .
docker run -p 8080:8080 hello-world:local
```

Then open `http://localhost:8080`.

## GitHub Actions secrets required

Set these in your repository secrets before running the full pipeline:

| Secret | Example value |
|--------|--------------|
| `AWS_ROLE_ARN` | `arn:aws:iam::123456789:role/github-actions` |
| `AWS_REGION` | `eu-west-2` |
| `ECR_REPOSITORY` | `hello-world-app` |

`EKS_CLUSTER_NAME` is optional — Terraform creates the cluster and passes 
the name to Helm automatically.

## Manual Helm deploy
```bash
TAG=$(aws ecr describe-images \
  --repository-name hello-world-app \
  --region eu-west-2 \
  --query "sort_by(imageDetails,& imagePushedAt)[-1].imageTags[0]" \
  --output text)

helm upgrade --install hello-world ./helm/hello-world \
  -n default --create-namespace \
  --set image.repository=<AWS_ACCOUNT_ID>.dkr.ecr.eu-west-2.amazonaws.com/hello-world-app \
  --set image.tag=$TAG \
  --set image.pullPolicy=Always \
  --set service.type=LoadBalancer
```

## Failure simulation

The workflow supports a `workflow_dispatch` input `simulate_failure`:

- `true` — fails intentionally at the CI test stage
- `false` — runs the full pipeline

This lets you demo CI/CD failure handling and recovery in a controlled way.

## Tech stack

| Tool | Purpose |
|------|---------|
| GitHub Actions | CI/CD automation |
| Docker | Containerisation |
| Amazon ECR | Docker image registry |
| Terraform | Infrastructure as code (VPC, EKS) |
| Helm | Kubernetes package management |
| Prometheus | Metrics collection |
| Grafana | Observability dashboards |
