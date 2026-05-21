# Petclinic Workspace

Monorepo workspace for the Spring Petclinic Microservices project on AWS. Contains the application source and its AWS infrastructure as git submodules.

## Submodules

| Submodule | Description |
|-----------|-------------|
| [`petclinic-platform`](petclinic-platform/) | AWS infrastructure — Terraform, Helm, Kubernetes manifests, ArgoCD, CI/CD pipelines |
| [`spring-petclinic-microservices`](spring-petclinic-microservices/) | Spring Boot 4 / Spring Cloud application — 8 microservices (read-only reference) |

## Getting Started

```bash
git clone --recurse-submodules https://github.com/manju2606/petclinic-workspace.git
cd petclinic-workspace
```

If you already cloned without `--recurse-submodules`:

```bash
git submodule update --init --recursive
```

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Cloud | AWS (eu-central-1) |
| IaC | Terraform >= 1.6, AWS provider ~> 5.0 |
| Cluster | Amazon EKS (Graviton t4g nodes, ARM64) |
| Database | Amazon RDS MySQL |
| Registry | Amazon ECR |
| Secrets | AWS Secrets Manager + External Secrets Operator |
| Packaging | Helm (single generic chart for all 8 services) |
| CI | GitHub Actions (build + push only, OIDC auth) |
| CD | ArgoCD (GitOps — auto-sync dev, manual sync prod) |
| App | Spring Boot 4.0.1 / Spring Cloud 2025.1.0 / Java 17 |

## Repository Layout

```
petclinic-workspace/
├── CLAUDE.md                          # Claude Code workspace instructions
├── petclinic-platform/                # Infrastructure repo (all work happens here)
│   ├── terraform/                     # VPC, EKS, RDS, ECR, Secrets modules
│   ├── helm/petclinic-service/        # Generic Helm chart (shared by all 8 services)
│   ├── helm-values/                   # Per-service + per-env values
│   ├── k8s/                           # Kubernetes base manifests + overlays
│   ├── .github/workflows/             # CI pipelines
│   ├── scripts/                       # Operational scripts
│   └── docs/                          # Architecture docs, runbooks, ADRs
└── spring-petclinic-microservices/    # Application source (read-only)
    ├── spring-petclinic-api-gateway/
    ├── spring-petclinic-config-server/
    ├── spring-petclinic-discovery-server/
    ├── spring-petclinic-customers-service/
    ├── spring-petclinic-visits-service/
    ├── spring-petclinic-vets-service/
    ├── spring-petclinic-genai-service/
    └── spring-petclinic-admin-server/
```
