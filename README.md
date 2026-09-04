# DevSecOps GitHub Actions CI/CD - AWS ECS Application Deployment

This standalone project builds, scans, provisions, and deploys a containerized FastAPI application to AWS ECS/Fargate through GitHub Actions.

## Pipeline stages

```text
Pull request or manual run
  -> Install, Ruff, pytest
  -> pip-audit and Trivy filesystem scan
  -> Terraform format, validate, and Checkov scan
  -> AWS OIDC authentication
  -> Optional Terraform provisioning
  -> Docker build, Trivy image scan, and ECR push
  -> ECS task deployment and rollout verification
```

## Contents

- `app/`: sample FastAPI application with `/` and `/health`
- `tests/`: application tests
- `infra/`: Terraform ECS/Fargate, ECR, ALB, IAM, networking, and logging resources
- `.github/workflows/deploy-aws-ecs.yml`: GitHub Actions pipeline
- `Dockerfile`: non-root application image

## GitHub configuration

Add these repository variables:

| Variable | Description |
| --- | --- |
| `AWS_REGION` | AWS deployment region |
| `AWS_VPC_ID` | Existing VPC ID |
| `AWS_SUBNET_IDS` | Comma-separated subnet IDs |
| `ECR_REPOSITORY` | ECR repository URI |
| `ECS_CLUSTER` | ECS cluster name |
| `ECS_SERVICE` | ECS service name |

Add this protected-environment secret:

| Secret | Description |
| --- | --- |
| `AWS_ROLE_ARN` | IAM role assumed through GitHub OIDC |

The IAM role must trust this repository and its protected `production` environment. Grant least-privilege permissions for Terraform target resources, ECR, ECS, and CloudWatch.

## Running the workflow

Pull requests run validation and security checks. A manual run from **Actions → AWS ECS DevSecOps Deployment** provides:

- `apply_infrastructure`: provision or update Terraform resources
- `deploy_application`: build, push, and deploy the commit-tagged image

Protect the `production` environment so an approval is required before deployment.

## AWS resources

Terraform creates the ECR repository, ECS cluster, task definition, service, load balancer, security groups, IAM execution role, and CloudWatch log group. It expects the VPC and subnets to already exist.

The ECS service uses the image tagged with the Git commit SHA, waits for ECS service stability, and relies on the `/health` endpoint and load balancer target health for verification.

AWS infrastructure can incur charges. Use budgets and alerts, review Terraform plans, and destroy demonstration resources when finished.
