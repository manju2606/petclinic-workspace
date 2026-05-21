# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Structure

This workspace contains two repositories with distinct roles:

| Directory | Role | Editable? |
|-----------|------|-----------|
| `petclinic-platform/` | All AWS infrastructure (Terraform, Helm, K8s, ArgoCD, CI/CD) | Yes — primary work area |
| `spring-petclinic-microservices/` | Spring Boot application source (8 services) | **READ-ONLY** — never modify |

**All infrastructure work happens in `petclinic-platform/`.** See `petclinic-platform/CLAUDE.md` for the complete infrastructure conventions, safety hooks, MCP servers, and Jira backlog.

Both directories are git submodules. To pull the latest commits from their remotes:

```bash
git submodule update --remote --merge
```

GitHub: https://github.com/manju2606/petclinic-workspace

## Application Overview (`spring-petclinic-microservices/`)

Spring Boot 4.0.1 / Spring Cloud 2025.1.0 (Oakwood) / Java 17 multi-module Maven project. Eight services communicate via Eureka service discovery; config is centralized in config-server backed by a Git repo.

**Startup dependency chain:** config-server (8888) → discovery-server (8761) → all other services

| Service | Port | DB |
|---------|------|----|
| config-server | 8888 | — |
| discovery-server | 8761 | — |
| api-gateway | 8080 | — |
| customers-service | 8081 | MySQL (default: HSQLDB in-memory) |
| visits-service | 8082 | MySQL |
| vets-service | 8083 | MySQL |
| genai-service | 8084 | optional MySQL; requires `OPENAI_API_KEY` |
| admin-server | 9090 | — |

## Application Commands

All commands run from `spring-petclinic-microservices/`.

```bash
# Build all modules (skip tests)
./mvnw clean install -DskipTests

# Build and run tests
./mvnw clean verify

# Run a single service locally (start config-server first)
cd spring-petclinic-{service}
../mvnw spring-boot:run

# Run a specific test class
../mvnw test -pl spring-petclinic-{service} -Dtest=ClassName

# Build Docker images (linux/amd64, loads into local daemon)
./mvnw clean install -P buildDocker

# Build for ARM64 (required for Graviton/Apple Silicon)
./mvnw clean install -P buildDocker -Dcontainer.platform=linux/arm64

# Start full stack with Docker Compose
docker compose up

# Activate MySQL profile (for customers/visits/vets services)
../mvnw spring-boot:run -Dspring-boot.run.profiles=mysql

# Recompile CSS (only needed when modifying .scss in api-gateway)
cd spring-petclinic-api-gateway && ../mvnw generate-resources -P css
```

## Infrastructure Commands

All commands run from `petclinic-platform/`. See `petclinic-platform/CLAUDE.md` for full conventions.

```bash
# Terraform workflow (always plan before apply)
terraform init                  # required on first use or after module changes
terraform fmt -recursive
terraform validate
terraform plan -out plan.out
terraform apply plan.out

# Validate all Helm releases (8 services × 2 envs)
scripts/validate-helm.sh

# Validate a single Helm release
helm template petclinic helm/petclinic-service/ \
  -f helm-values/{service}.yaml \
  -f helm-values/{env}.yaml

# Security scan a Terraform module
checkov -d terraform/modules/{module}

# ArgoCD sync (after port-forward to argocd-server:8443)
argocd app sync {service}-{env}
```

## Key Architecture Decisions

- **CI/CD split:** GitHub Actions builds and pushes images, then commits the new SHA tag to `helm-values/{service}.yaml`. ArgoCD detects the commit and deploys. CI never runs `kubectl` or `helm upgrade`.
- **All-public subnets:** Resources are in public subnets (no NAT Gateway) for cost. Security groups are the enforced perimeter. See `petclinic-platform/docs/adr/0001-public-subnets.md`.
- **Single Helm chart:** One generic chart (`helm/petclinic-service/`) serves all 8 services. Per-service config in `helm-values/{service}.yaml`, per-env config in `helm-values/{dev,prod}.yaml`. ArgoCD merges both.
- **Docker target platform:** `linux/arm64` for EKS Graviton (t4g) nodes. Use `-Dcontainer.platform=linux/arm64` when building for the cluster.
- **Secrets:** AWS Secrets Manager + External Secrets Operator. Never in YAML or tfvars.

## Technical Spec

All infrastructure values (CIDRs, ports, instance sizes, probe timings, alert thresholds) are the single source of truth in `petclinic-platform/docs/technical-spec.md`. Read the relevant section before implementing any infrastructure story.
