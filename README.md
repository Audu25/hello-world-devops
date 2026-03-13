# Hello World DevOps on EKS

End-to-end CI/CD pipeline deploying a Flask microservice to AWS EKS using 
Terraform, GitHub Actions, and Helm — with failure simulation built in.

## Architecture
```<svg width="100%" viewBox="0 0 680 620">
  <defs>
    <marker id="arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M2 1L8 5L2 9" fill="none" stroke="context-stroke" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
    </marker>
  </defs>

  <!-- Stage labels -->
  <text class="ts" x="40" y="30" text-anchor="middle" dominant-baseline="central">1</text>
  <text class="ts" x="180" y="30" text-anchor="middle" dominant-baseline="central">2</text>
  <text class="ts" x="320" y="30" text-anchor="middle" dominant-baseline="central">3</text>
  <text class="ts" x="460" y="30" text-anchor="middle" dominant-baseline="central">4</text>
  <text class="ts" x="600" y="30" text-anchor="middle" dominant-baseline="central">5</text>

  <!-- Row 1: Developer → GitHub -->
  <g class="node c-gray" onclick="sendPrompt('What does the developer push in this pipeline?')">
    <rect x="10" y="50" width="120" height="56" rx="8" stroke-width="0.5"/>
    <text class="th" x="70" y="71" text-anchor="middle" dominant-baseline="central">Developer</text>
    <text class="ts" x="70" y="89" text-anchor="middle" dominant-baseline="central">Pushes code</text>
  </g>

  <line x1="130" y1="78" x2="148" y2="78" class="arr" marker-end="url(#arrow)"/>

  <g class="node c-purple" onclick="sendPrompt('What happens when code is pushed to GitHub?')">
    <rect x="150" y="50" width="120" height="56" rx="8" stroke-width="0.5"/>
    <text class="th" x="210" y="71" text-anchor="middle" dominant-baseline="central">GitHub</text>
    <text class="ts" x="210" y="89" text-anchor="middle" dominant-baseline="central">Source repository</text>
  </g>

  <line x1="270" y1="78" x2="288" y2="78" class="arr" marker-end="url(#arrow)"/>

  <g class="node c-purple" onclick="sendPrompt('What does GitHub Actions do in this pipeline?')">
    <rect x="290" y="50" width="140" height="56" rx="8" stroke-width="0.5"/>
    <text class="th" x="360" y="71" text-anchor="middle" dominant-baseline="central">GitHub Actions</text>
    <text class="ts" x="360" y="89" text-anchor="middle" dominant-baseline="central">CI — build & test</text>
  </g>

  <line x1="430" y1="78" x2="448" y2="78" class="arr" marker-end="url(#arrow)"/>

  <g class="node c-teal" onclick="sendPrompt('What is Amazon ECR used for here?')">
    <rect x="450" y="50" width="120" height="56" rx="8" stroke-width="0.5"/>
    <text class="th" x="510" y="71" text-anchor="middle" dominant-baseline="central">Amazon ECR</text>
    <text class="ts" x="510" y="89" text-anchor="middle" dominant-baseline="central">Docker image store</text>
  </g>

  <!-- Down arrow from GitHub Actions to Terraform -->
  <line x1="360" y1="106" x2="360" y2="154" class="arr" marker-end="url(#arrow)"/>
  <text class="ts" x="375" y="133" dominant-baseline="central">on push</text>

  <!-- Terraform box (middle row center) -->
  <g class="node c-purple" onclick="sendPrompt('What AWS infrastructure does Terraform provision here?')">
    <rect x="270" y="156" width="180" height="56" rx="8" stroke-width="0.5"/>
    <text class="th" x="360" y="177" text-anchor="middle" dominant-baseline="central">Terraform</text>
    <text class="ts" x="360" y="195" text-anchor="middle" dominant-baseline="central">Provisions VPC + EKS</text>
  </g>

  <!-- Down arrow from Terraform to Helm -->
  <line x1="360" y1="212" x2="360" y2="260" class="arr" marker-end="url(#arrow)"/>

  <!-- Helm box -->
  <g class="node c-teal" onclick="sendPrompt('How does Helm deploy the app to EKS?')">
    <rect x="270" y="262" width="180" height="56" rx="8" stroke-width="0.5"/>
    <text class="th" x="360" y="283" text-anchor="middle" dominant-baseline="central">Helm</text>
    <text class="ts" x="360" y="301" text-anchor="middle" dominant-baseline="central">Deploys to EKS cluster</text>
  </g>

  <!-- ECR down arrow to Helm (image pulled) -->
  <path d="M510 106 L510 318 L452 318" fill="none" stroke="var(--s)" stroke-width="1" marker-end="url(#arrow)"/>
  <text class="ts" x="530" y="220" dominant-baseline="central">image</text>
  <text class="ts" x="530" y="236" dominant-baseline="central">pulled</text>

  <!-- Down arrow from Helm to EKS cluster -->
  <line x1="360" y1="318" x2="360" y2="366" class="arr" marker-end="url(#arrow)"/>

  <!-- AWS Cloud container -->
  <rect x="40" y="368" width="600" height="200" rx="16" fill="none" stroke="var(--b)" stroke-width="0.5" stroke-dasharray="6 4"/>
  <text class="ts" x="56" y="388" dominant-baseline="central">AWS Cloud</text>

  <!-- EKS Cluster container -->
  <rect x="60" y="400" width="560" height="148" rx="12" fill="none" stroke="var(--b)" stroke-width="0.5" stroke-dasharray="4 3"/>
  <text class="ts" x="76" y="418" dominant-baseline="central">EKS cluster</text>

  <!-- Flask app pod -->
  <g class="node c-teal" onclick="sendPrompt('What runs inside the Flask app pod?')">
    <rect x="80" y="428" width="150" height="56" rx="8" stroke-width="0.5"/>
    <text class="th" x="155" y="449" text-anchor="middle" dominant-baseline="central">Flask app</text>
    <text class="ts" x="155" y="467" text-anchor="middle" dominant-baseline="central">Pod / LoadBalancer</text>
  </g>

  <!-- Prometheus -->
  <g class="node c-coral" onclick="sendPrompt('What does Prometheus monitor in this setup?')">
    <rect x="270" y="428" width="140" height="56" rx="8" stroke-width="0.5"/>
    <text class="th" x="340" y="449" text-anchor="middle" dominant-baseline="central">Prometheus</text>
    <text class="ts" x="340" y="467" text-anchor="middle" dominant-baseline="central">Scrapes metrics</text>
  </g>

  <!-- Grafana -->
  <g class="node c-coral" onclick="sendPrompt('What dashboards does Grafana show?')">
    <rect x="450" y="428" width="140" height="56" rx="8" stroke-width="0.5"/>
    <text class="th" x="520" y="449" text-anchor="middle" dominant-baseline="central">Grafana</text>
    <text class="ts" x="520" y="467" text-anchor="middle" dominant-baseline="central">Dashboard + alerts</text>
  </g>

  <!-- Prometheus to Grafana arrow -->
  <line x1="410" y1="456" x2="448" y2="456" class="arr" marker-end="url(#arrow)"/>

  <!-- Flask to Prometheus scrape arrow -->
  <line x1="230" y1="456" x2="268" y2="456" class="arr" marker-end="url(#arrow)"/>

  <!-- Helm deploy arrow to Flask pod -->
  <path d="M360 366 L360 395 L155 395 L155 426" fill="none" stroke="var(--s)" stroke-width="1" marker-end="url(#arrow)"/>

  <!-- Failure simulation note -->
  <rect x="80" y="550" width="520" height="36" rx="8" fill="none" stroke="var(--b)" stroke-width="0.5" stroke-dasharray="3 3"/>
  <text class="ts" x="340" y="572" text-anchor="middle" dominant-baseline="central">Failure simulation: workflow_dispatch input simulate_failure=true fails CI intentionally</text>

</svg>
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
