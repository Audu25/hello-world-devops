# Hello World DevOps on EKS

This project simulates a microservice CI/CD pipeline:

1. Developer pushes code to GitHub
2. GitHub Actions runs tests
3. Docker image is built
4. Image pushed to Amazon ECR
5. Terraform provisions EKS-related infra
6. Helm deploys app to EKS

## Structure

- `app/`: Flask hello-world microservice
- `Dockerfile`: container image build
- `.github/workflows/cicd.yml`: CI/CD workflow
- `terraform/`: AWS infra (VPC + EKS module)
- `helm/hello-world/`: Helm chart for deployment

## Local quick run

```bash
docker build -t hello-world:local .
docker run -p 8080:8080 hello-world:local
```

Then open `http://localhost:8080`.

## GitHub Actions secrets required

Set these repository secrets before running full deployment:

- `AWS_ROLE_ARN`
- `AWS_REGION` (example: `eu-west-2`)
- `ECR_REPOSITORY` (example: `hello-world-app`)
- `EKS_CLUSTER_NAME`

## Failure simulation (Question 1)

Workflow supports `workflow_dispatch` input `simulate_failure`:

- `true`: fails intentionally in CI test stage.
- `false`: runs full pipeline.

This lets you demo CI/CD failure handling.