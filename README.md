# DevOps Deployment Guide

This repository is a practical reference for understanding how modern application deployment works. It covers the path from a Git change to a monitored production release using CI/CD, Docker, Kubernetes, infrastructure as code, and cloud services.

> **Repository status:** This is a documentation and command reference project. It does not currently include an application source tree, Dockerfile, Kubernetes manifests, or a GitHub Actions workflow to run directly. Commands containing names such as `myapp`, `myrepo`, or `deployment.yaml` are templates and must be adapted to a real application.

## Contents

- [Deployment at a glance](#deployment-at-a-glance)
- [Recommended release flow](#recommended-release-flow)
- [Prerequisites](#prerequisites)
- [Local container workflow](#local-container-workflow)
- [CI/CD workflow](#cicd-workflow)
- [Kubernetes deployment](#kubernetes-deployment)
- [Configuration and secrets](#configuration-and-secrets)
- [Verification and monitoring](#verification-and-monitoring)
- [Rollback](#rollback)
- [Deployment checklist](#deployment-checklist)
- [Reference files](#reference-files)

## Deployment at a glance

```text
Feature branch
     |
     v
Pull request --> lint, test, build
     |
     v
main branch --> build the versioned artifact/image
     |
     v
Staging --> smoke tests and approval
     |
     v
Production --> rolling/canary release and monitoring
     |
     +--> rollback to the previous known-good version if required
```

The same versioned artifact should move from staging to production. Do not rebuild it between environments; rebuilding can introduce differences that staging did not test.

## Recommended release flow

1. Create a short-lived feature branch from `main`.
2. Make the change and push the branch.
3. Open a pull request. CI should run linting, tests, and a build.
4. Merge only after review and successful CI checks.
5. On `main`, build and tag the deployable artifact with the commit SHA or release version.
6. Push the image to a container registry.
7. Deploy that exact image to staging and run smoke tests.
8. Approve a production release, or allow the production job to run automatically if that is the team's policy.
9. Monitor health, logs, error rate, and latency after deployment.
10. Roll back to the previous image when the release is unhealthy.

## Prerequisites

Install only the tools required by the deployment target:

- Git
- Docker Desktop, for building and testing containers locally
- A container registry account, such as Docker Hub, Amazon ECR, Azure Container Registry, or Google Artifact Registry
- `kubectl` and access credentials, when deploying to Kubernetes
- A CI/CD platform, such as GitHub Actions
- Access to the target cloud account and its secret/configuration store

Before deploying, decide and document the real values for:

- Application name and image name
- Container port and public URL
- Registry and image tag format
- Staging and production environments
- Required environment variables
- Health-check endpoints, such as `/health` and `/ready`
- Approval and rollback owners

## Local container workflow

The application must have a `Dockerfile` before these commands can be used. Replace the example image name and port with the application's values.

```bash
# Build an image
docker build -t myrepo/myapp:local .

# Run it locally; the left port is the host port
docker run --rm -p 8080:8080 myrepo/myapp:local

# Inspect the running service
docker ps
docker logs -f <container_id>
```

For a multi-service local environment, use Docker Compose:

```bash
docker compose up -d
docker compose logs -f
docker compose down
```

The image should listen on `0.0.0.0` inside the container, not only on `localhost`, so that Docker and Kubernetes can route traffic to it.

## CI/CD workflow

A GitHub Actions workflow normally contains these stages:

1. **Validate:** checkout, install dependencies, lint, and run tests.
2. **Build:** compile/package the application and build its Docker image.
3. **Publish:** push the image to a registry using an immutable tag such as `${{ github.sha }}`.
4. **Deploy staging:** update staging to that exact tag.
5. **Verify:** run smoke tests and check rollout health.
6. **Deploy production:** require approval when the environment is protected, then release the same tag.

Example image commands for a pipeline:

```bash
docker build -t myrepo/myapp:${GIT_SHA} .
docker push myrepo/myapp:${GIT_SHA}
```

Use CI platform secrets for registry credentials, cloud credentials, and application secrets. Never print those values in logs. Production should not use the mutable `latest` tag because it makes releases difficult to trace and reproduce.

## Kubernetes deployment

Kubernetes deployment files should define at least a `Deployment` and a `Service`. The deployment should include resource limits, readiness/liveness probes, and a rolling-update policy before being used in production.

```bash
kubectl apply -f deployment.yaml
kubectl rollout status deployment/myapp
kubectl get pods
kubectl get services
kubectl logs -f deployment/myapp
```

A typical release changes only the image tag, for example:

```bash
kubectl set image deployment/myapp \
  myapp=myrepo/myapp:${GIT_SHA}
kubectl rollout status deployment/myapp
```

Do not place passwords or API keys directly in a committed manifest. Use a Kubernetes `Secret`, an external secret manager, or the secret mechanism provided by the cloud platform.

## Configuration and secrets

Configuration that changes between environments belongs outside the image. Common examples include `DATABASE_URL`, `API_URL`, `NODE_ENV`, and feature flags.

For local development, use an ignored `.env` file:

```dotenv
NODE_ENV=development
DATABASE_URL=postgres://user:password@localhost:5432/app
```

Add `.env` to `.gitignore` and use the CI/CD platform or cloud secret manager for staging and production. Rotate any secret that is accidentally committed, even if the commit is later deleted.

## Verification and monitoring

Deployment is complete only after the new version is healthy:

- Confirm the rollout reaches the desired number of ready replicas.
- Call the public endpoint and the `/health` endpoint.
- Confirm `/ready` reports that dependencies are available, when implemented.
- Review application logs for startup failures and repeated errors.
- Check request latency, error rate, CPU, and memory.
- Watch the service for an agreed observation period after release.

Health checks let an orchestrator remove unhealthy instances from traffic and restart failed containers. Monitoring and alerting should be configured before production deployment, not during the first incident.

## Rollback

Use the deployment platform's rollback mechanism rather than rebuilding an old version from source.

```bash
# Kubernetes: return to the previous rollout
kubectl rollout undo deployment/myapp
kubectl rollout status deployment/myapp

# Git: create a reviewed code-level revert when appropriate
git revert <bad-commit-sha>
git push origin main
```

After a rollback, check service health, preserve the failed release logs, and record the incident and follow-up action. Database migrations need special care: prefer backward-compatible, additive changes so both the old and new application versions can run during a rollout.

## Deployment checklist

- [ ] Tests and build pass in CI.
- [ ] The image has an immutable commit or release tag.
- [ ] The image was pushed to the intended registry.
- [ ] Required environment variables and secrets exist in the target environment.
- [ ] Health and readiness checks are available.
- [ ] Staging smoke tests pass.
- [ ] Production approval is recorded, when required.
- [ ] The rollout reaches a healthy state.
- [ ] Logs, metrics, and alerts are being monitored.
- [ ] The previous image tag and rollback command are known.

## Reference files

- [NOTES.md](NOTES.md): detailed explanations of SDLC, environments, CI/CD, Docker, Kubernetes, IaC, monitoring, rollback, and cloud platforms.
- [CHEATSHEET.md](CHEATSHEET.md): quick command reference and minimal Docker, Compose, Kubernetes, and GitHub Actions examples.

---

## Additional DevOps deployment notes

### Deployment strategies

Modern teams usually choose one of several deployment patterns depending on risk tolerance and service criticality.

#### Rolling deployment

A rolling deployment replaces pods or instances gradually. Some old instances remain in service while new ones are added. This approach is simple and low-risk for large systems because it avoids sudden disruption. However, it may take longer to complete a full rollout and it requires careful health checks to ensure a partially updated system remains stable.

#### Blue-green deployment

In a blue-green strategy, two production-like environments exist at the same time: blue and green. The current live environment serves user traffic, while the new version is deployed to the inactive environment. Once validation succeeds, traffic is shifted to the new environment. This approach reduces downtime and makes rollback very fast, but it costs more because the infrastructure is duplicated.

#### Canary deployment

A canary release introduces the new version to a subset of users or traffic first. This reduces blast radius because only a fraction of the user base experiences the change. If metrics stay healthy, traffic is increased gradually. Canary deployments are common in production systems where zero-downtime and resilience are critical.

#### Feature flag deployment

Some teams decouple deployment from release by merging code into the main branch while keeping features switched off behind flags. This technique allows release managers to enable features gradually in production. It helps reduce risk and improve experimentation, but it also adds complexity and requires strong governance around flag lifecycle and cleanup.

### Infrastructure as Code (IaC)

Infrastructure as Code means infrastructure is managed using version-controlled configuration files instead of manual clicks in a cloud console. Examples include Terraform, CloudFormation, Bicep, and Pulumi. Benefits include repeatability, faster provisioning, auditability, and easier rollback.

A good IaC workflow includes:

- storing definitions in Git
- validating templates in CI
- separating environments such as dev, staging, and production
- documenting required inputs and outputs
- using drift detection to compare actual infrastructure with the desired state

### Container lifecycle management

Containers are not the end of the story. They need lifecycle management to function reliably in production.

Important considerations include:

- container image scanning for security vulnerabilities
- pinning image versions instead of using floating tags
- limiting resource requests and limits
- configuring readiness and liveness probes
- cleaning up old images and container artifacts
- using private registries when the system is sensitive or regulated

Container orchestration platforms such as Kubernetes help automate restart policies, scaling, networking, traffic routing, and state reconciliation.

### Kubernetes and service orchestration

Kubernetes manages workloads across nodes and clusters. Typical resources include:

- Deployment for managing replica sets
- Service for stable networking and routing
- ConfigMap for non-secret configuration
- Secret for sensitive values
- Ingress for HTTP routing from outside the cluster
- PersistentVolume and PersistentVolumeClaim for durable storage
- HorizontalPodAutoscaler for automatic scaling based on CPU or memory

Production Kubernetes clusters should include cluster-wide logging, node health checks, auto-repair policies, network policies, and resource quotas. Without these, the cluster may appear healthy while still failing under real traffic.

### CI/CD best practices

A robust CI/CD pipeline should enforce quality gates before release.

Recommended practices:

- run linting and static analysis on every pull request
- execute unit and integration tests early and consistently
- build immutable artifacts from source code
- publish versioned images or packages to a registry
- require review and approvals for protected branches
- store secrets in the CI/CD platform, not in the repo
- avoid manual changes directly in production environments
- keep the deployment pipeline observable and auditable

GitHub Actions, GitLab CI, Jenkins, Azure DevOps, and CircleCI all support similar stages: validate, build, test, release, deploy, verify, and monitor.

### Monitoring and observability

Monitoring is what tells teams whether a deployment is healthy. Observability goes further by making systems understandable when they fail.

Key metrics include:

- CPU and memory usage
- request latency
- error rate and failed requests
- queue depth and background job throughput
- database query latency
- number of running replicas and restarts
- availability and downtime windows

Logs, traces, and metrics should be correlated to quickly answer questions such as: Who is impacted? What changed? Which service failed? What was the outage window?

A production-ready environment usually includes dashboards, alert rules, escalation paths, and post-incident review habits.

### Security in deployment

Security is part of delivery, not an afterthought. Teams should follow baseline controls such as:

- least-privilege access to cloud resources
- authentication for deployment jobs
- signed commits and verified pipelines
- image vulnerability scanning
- dependency vulnerability scanning
- secret rotation and expiration
- network segmentation and firewall policies
- regular patching of hosts and runtimes

Production systems should avoid hard-coded credentials, local admin access, or unrestricted internet exposure. Security checks should be part of the CI process and the runtime environment.

### Database and migration considerations

Application deployment is often more complicated than code rollout because the database may also change.

Best practices include:

- keep schema changes backward compatible when possible
- apply migrations in small, reversible steps
- test migration scripts in staging before production
- back up data before risky database operations
- ensure app and database versions are compatible during rollout windows
- monitor write amplification, query performance, and lock times

If a deployment includes a breaking database change, it must be planned carefully so all application versions can coexist safely.

### Collaboration and operations

DevOps is not only automation. It is also collaboration among developers, QA, operations, security, and release managers.

A healthy team usually defines:

- ownership for each service
- communication channels for incidents
- service-level objectives and error budgets
- on-call coverage and escalation rules
- playbooks for rollback and handoff
- architecture review and change approval standards

When teams share responsibility, deployments become more predictable and recovery becomes faster.

### Common deployment risks

Some of the most common deployment issues include:

- configuration drift between environments
- missing secret or variable values in production
- insufficient health checks and monitoring
- overloading shared services during peak time
- incomplete rollback plans
- untested database migrations
- image tag confusion caused by mutable tags like latest
- manual steps that bypass the CI/CD pipeline

Shifting from ad hoc releases to repeatable automation reduces these risks significantly.

### Example production workflow

A typical production deployment can look like this:

1. A developer creates a pull request and pushes code.
2. CI runs linting, tests, build validation, and security checks.
3. The application is packaged and versioned with an immutable tag.
4. The image is pushed to a container registry.
5. The team deploys the same artifact to staging.
6. Smoke tests verify critical flows.
7. A production approval is requested or automatically granted.
8. The service is released using a controlled strategy such as rolling or canary.
9. Logs, metrics, dashboards, and health checks are monitored.
10. The team rolls back quickly if error rates or latency exceed thresholds.

### Final takeaway

Deployment is one of the most important parts of the software lifecycle because it connects code creation to real user impact. A strong DevOps practice combines version control, automation, testing, containerization, cloud infrastructure, monitoring, and disciplined release management. When done correctly, it reduces risk, improves reliability, and helps teams deliver value faster with confidence.

---

### Summary

- Deployment is the process of moving an application from development into a usable environment.
- CI/CD helps automate validation, packaging, and release.
- Containers and Kubernetes support scalable and portable deployments.
- Monitoring, security, rollback planning, and environment consistency are essential.
- DevOps is about people, process, and tooling working together to deliver software safely and efficiently.