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