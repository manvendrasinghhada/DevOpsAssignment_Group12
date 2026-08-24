# 📝 Deployment Notes — Cloud/DevOps (CI/CD + Containers)

> Intermediate-level, end-to-end notes on how deployment works in a real engineering org.

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Where Deployment Fits in the SDLC](#2-where-deployment-fits-in-the-sdlc)
3. [Environments](#3-environments)
4. [Version Control & Branching](#4-version-control--branching)
5. [Continuous Integration (CI)](#5-continuous-integration-ci)
6. [Containerization with Docker](#6-containerization-with-docker)
7. [Container Orchestration (Kubernetes)](#7-container-orchestration-kubernetes)
8. [Continuous Delivery vs Continuous Deployment](#8-continuous-delivery-vs-continuous-deployment)
9. [CI/CD Tools Overview](#9-cicd-tools-overview)
10. [Deployment Strategies](#10-deployment-strategies)
11. [Infrastructure as Code (IaC)](#11-infrastructure-as-code-iac)
12. [Secrets & Config Management](#12-secrets--config-management)
13. [Monitoring, Logging & Alerting](#13-monitoring-logging--alerting)
14. [Rollback & Disaster Recovery](#14-rollback--disaster-recovery)
15. [Cloud Platforms Overview](#15-cloud-platforms-overview)
16. [End-to-End Walkthrough (Example)](#16-end-to-end-walkthrough-example)
17. [Best Practices & Common Pitfalls](#17-best-practices--common-pitfalls)
18. [Glossary](#18-glossary)

---

## 1. Introduction

Deployment is the set of processes that move code from a developer's machine to a live, running system that real users interact with — safely, repeatably, and with the ability to undo mistakes.

In modern engineering, deployment is **automated end-to-end**. Manually copying files to a server (FTP-style deployment) is largely a thing of the past for serious projects. Instead, teams build **pipelines**: a sequence of automated stages that a code change flows through before it reaches users.

The two ideas at the heart of modern deployment:
- **CI/CD** — Continuous Integration / Continuous Delivery (or Deployment): automation of build, test, and release.
- **Containers** — packaging an application with everything it needs to run, so it behaves identically everywhere (your laptop, staging, production).

---

## 2. Where Deployment Fits in the SDLC

The Software Development Life Cycle (SDLC) is usually described as: **Plan → Code → Build → Test → Release → Deploy → Operate → Monitor**.

Deployment sits at the **Release → Deploy** boundary, but in a DevOps culture it's connected to everything around it:

- It depends on **CI** (build/test) having passed.
- It feeds into **Operate/Monitor** (once live, you watch it).
- It's tightly looped with **Rollback** (if monitoring detects a problem, you deploy backward).

This is why the SDLC is often drawn as an infinite loop (the "DevOps infinity loop") rather than a straight line — deployment isn't the end, it's a repeating stage.

---

## 3. Environments

Most teams run the same code through multiple environments, each with a different purpose:

| Environment | Purpose | Who uses it |
|---|---|---|
| **Local** | Developer's own machine | Individual developer |
| **Development (Dev)** | Shared integration environment for active work | Dev team |
| **Staging (Pre-prod)** | Mirrors production as closely as possible; final testing | QA, product |
| **Production (Prod)** | Live environment serving real users | End users |

**Why not deploy straight to production?**
Each environment acts as a filter that catches bugs before they reach real users. Staging in particular is meant to be a near-exact replica of production (same OS, same config shape, same scale-down infra) so that "it worked in staging" is a meaningful signal.

Environment-specific behavior is usually controlled via **environment variables** (e.g. `DATABASE_URL`, `API_KEY`), not hardcoded values or separate code branches.

---

## 4. Version Control & Branching

Deployment pipelines are triggered by **Git events** (a push, a merge, a tag). So the branching strategy directly shapes how deployment works.

Common strategies:

- **Git Flow** — long-lived `main` and `develop` branches, plus `feature/*`, `release/*`, `hotfix/*`. Heavier process, common in versioned/release-based products.
- **Trunk-Based Development** — everyone merges small changes into `main` frequently; feature flags hide unfinished work. Pairs very well with CI/CD because `main` is always deployable.
- **GitHub Flow** — a simplified version: `main` is always deployable, work happens in short-lived feature branches merged via Pull Requests.

**Typical trigger mapping:**
- Push to `feature/*` → run tests only (CI)
- Merge to `main` → deploy to staging automatically
- Tag `v1.2.0` or manual approval → deploy to production

---

## 5. Continuous Integration (CI)

CI means: **every code change is automatically built and tested** as soon as it's pushed, rather than integrating everyone's work in one big painful merge at the end.

A typical CI stage does:
1. Checkout code
2. Install dependencies
3. Run linters/static analysis
4. Run unit tests (and sometimes integration tests)
5. Build the application (compile, bundle, etc.)
6. Report status back to the PR (pass/fail)

**Why it matters:** it catches bugs within minutes of being introduced, not weeks later. It also means `main` stays in a "known-good" state, which is a prerequisite for automated deployment.

---

## 6. Containerization with Docker

**The problem containers solve:** "it works on my machine" — differences in OS, installed libraries, and versions between dev, staging, and prod cause bugs that have nothing to do with the actual code.

**Docker** packages an application with its exact dependencies, runtime, and configuration into a single unit called an **image**. A running instance of an image is a **container**.

Key concepts:
- **Dockerfile** — a script of instructions to build an image (base OS, install deps, copy code, set start command).
- **Image** — a built, immutable snapshot (like a class).
- **Container** — a running instance of an image (like an object).
- **Registry** — a place to store and version images (Docker Hub, AWS ECR, GCP Artifact Registry, Azure ACR).
- **Docker Compose** — defines and runs multi-container setups (e.g., app + database + cache) locally with one config file.

**Why containers fit CI/CD so well:** the exact same image that passed tests in CI is the one deployed to staging and then production — nothing is rebuilt or reinstalled in between, eliminating environment drift.

---

## 7. Container Orchestration (Kubernetes)

Once you have more than a couple of containers (multiple services, multiple replicas for scale, self-healing needs), you need something to manage them. That's **container orchestration**.

**Kubernetes (K8s)** is the dominant orchestrator. Core concepts:

| Concept | What it is |
|---|---|
| **Pod** | Smallest deployable unit; wraps one (or a few tightly-coupled) containers |
| **Deployment** | Declares how many replicas of a pod should run and how to update them |
| **Service** | Stable network endpoint that routes traffic to healthy pods |
| **Ingress** | Manages external HTTP(S) access/routing into the cluster |
| **ConfigMap / Secret** | Externalized configuration and sensitive values |
| **Node** | A physical/virtual machine in the cluster running pods |

Kubernetes continuously watches the *desired state* (declared in YAML) vs the *actual state*, and self-heals: if a container crashes, it's restarted automatically; if a node dies, pods are rescheduled elsewhere.

**Lighter-weight alternatives** intermediate teams also use: AWS ECS/Fargate, Google Cloud Run, Azure Container Apps — container orchestration without managing a full K8s cluster.

---

## 8. Continuous Delivery vs Continuous Deployment

These two terms are often confused:

- **Continuous Delivery (CD):** every change that passes CI is automatically prepared for release (built, tested, packaged) and is **ready to deploy at any time** — but a human clicks "deploy" (usually to production).
- **Continuous Deployment (also CD):** goes one step further — every change that passes all automated checks is deployed **automatically, with no human gate**, all the way to production.

Most companies practice **Continuous Delivery to production with a manual gate**, and full **Continuous Deployment** for staging/lower environments.

---

## 9. CI/CD Tools Overview

| Tool | Notes |
|---|---|
| **GitHub Actions** | YAML-based, lives in `.github/workflows/`, tightly integrated with GitHub repos — very common for small/medium projects |
| **GitLab CI/CD** | YAML-based (`.gitlab-ci.yml`), built into GitLab, strong built-in container registry |
| **Jenkins** | Self-hosted, highly customizable via plugins, `Jenkinsfile` (Groovy) — older but still widely used in enterprises |
| **CircleCI** | Cloud-based, YAML config, known for fast parallel builds |
| **ArgoCD** | GitOps-style CD specifically for Kubernetes — deploys by syncing cluster state to a Git repo |

A pipeline config generally defines: **triggers** (on push/PR/tag) → **jobs/stages** (build, test, deploy) → **steps** within each job → **environment/secrets** used.

---

## 10. Deployment Strategies

How you swap the *old* running version for the *new* one matters a lot for uptime and risk.

| Strategy | How it works | Downtime? | Risk |
|---|---|---|---|
| **Recreate** | Stop old version entirely, then start new version | Yes | High (all-or-nothing) |
| **Rolling Update** | Gradually replace old instances with new ones, a few at a time | No | Medium |
| **Blue-Green** | Run two identical environments ("blue" = live, "green" = new); switch traffic all at once | No | Low (instant rollback = switch back) |
| **Canary** | Send a small % of traffic to the new version first, gradually increase | No | Low (catch issues early, small blast radius) |
| **A/B Testing** | Similar to canary, but split by user segment for feature comparison rather than safety | No | Low |

**Rule of thumb:** Canary and Blue-Green are preferred for production; Rolling updates are the Kubernetes default; Recreate is mostly used for simple/non-critical services.

---

## 11. Infrastructure as Code (IaC)

Instead of manually clicking around a cloud console to create servers, networks, and databases, IaC defines infrastructure in **version-controlled config files**.

- **Terraform** — cloud-agnostic, declarative (`.tf` files), the most widely used IaC tool.
- **AWS CloudFormation** — AWS-native equivalent.
- **Ansible** — more focused on configuration management (installing/configuring software on existing servers) than provisioning, though it can do both.

**Why it matters for deployment:** IaC means environments (staging, prod) can be recreated identically and consistently, and infrastructure changes go through the same PR/review process as code — no untracked manual changes ("configuration drift").

---

## 12. Secrets & Config Management

Never commit secrets (API keys, DB passwords, tokens) to Git — even in private repos.

Common approaches:
- **Environment variables** injected at runtime (via CI/CD pipeline secrets, or the orchestrator)
- **`.env` files** for local dev only — always in `.gitignore`
- **Secret managers**: AWS Secrets Manager, HashiCorp Vault, GitHub Actions Secrets, Kubernetes Secrets
- **The 12-Factor App principle:** config that varies between environments should live in the environment, not in the code.

---

## 13. Monitoring, Logging & Alerting

Deployment isn't done when the deploy command finishes — you need visibility into whether it's actually healthy.

- **Logging:** centralized log collection (e.g., ELK stack — Elasticsearch/Logstash/Kibana, or cloud-native equivalents like CloudWatch Logs)
- **Metrics:** CPU, memory, request latency, error rate (Prometheus + Grafana is a very common combo)
- **Alerting:** automated notifications (Slack, PagerDuty, email) when metrics cross a threshold
- **Health checks:** endpoints like `/health` or `/ready` that orchestrators poll to know if a container is alive and ready for traffic — this is what enables self-healing and safe rolling updates

---

## 14. Rollback & Disaster Recovery

Things will break. A good deployment pipeline makes reversing a bad release **fast and boring**, not a crisis.

- **Rollback** = redeploying the last known-good version (often just re-running the previous pipeline artifact/image tag)
- **Feature flags** allow disabling a broken feature instantly without a full redeploy
- **Database migrations** are the hardest part to roll back — prefer backward-compatible migrations (additive changes) so old and new code can both run against the same schema during a rollout

**Key metric:** MTTR (Mean Time To Recovery) — how fast you can detect and fix/rollback an incident. Good CI/CD pipelines optimize for low MTTR, not just fast deploys.

---

## 15. Cloud Platforms Overview

| Provider | Common deployment services |
|---|---|
| **AWS** | EC2 (VMs), ECS/Fargate (containers), EKS (managed K8s), Elastic Beanstalk (PaaS), Lambda (serverless) |
| **Azure** | Azure VMs, AKS (managed K8s), App Service (PaaS), Azure Functions (serverless) |
| **GCP** | Compute Engine (VMs), GKE (managed K8s), Cloud Run (serverless containers), Cloud Functions |

**General trend:** teams increasingly prefer *managed* container/serverless services (Cloud Run, Fargate, App Service) over managing raw VMs or self-hosted Kubernetes, to reduce operational overhead — full self-managed Kubernetes is reserved for teams with real scale/complexity needs.

---

## 16. End-to-End Walkthrough (Example)

A realistic pipeline for a typical web app:

1. **Developer** pushes code to a `feature/*` branch, opens a PR
2. **CI (GitHub Actions)** runs on the PR: lint → unit tests → build
3. PR reviewed and merged into `main`
4. **CI** runs again on `main`: build → test → **build Docker image** → push image to a registry (tagged with commit SHA)
5. **CD pipeline** automatically deploys that image to the **staging** environment (e.g., a K8s namespace or Cloud Run service)
6. Automated smoke tests / QA run against staging
7. On approval (manual gate, or automatic if using full Continuous Deployment), the **same image** is deployed to **production** using a rolling or canary strategy
8. **Monitoring** dashboards and alerts watch error rates and latency post-deploy
9. If something's wrong → **rollback** to the previous image tag; if not → the release is complete

The key idea to remember: **the exact same artifact (Docker image) moves through every environment** — nothing is rebuilt between staging and production, which is what makes "it worked in staging" a trustworthy signal.

---

## 17. Best Practices & Common Pitfalls

**Do:**
- Keep `main`/trunk always deployable
- Automate everything that's repeatable (build, test, deploy)
- Make small, frequent deployments rather than big infrequent ones (smaller blast radius)
- Use health checks so orchestrators can detect and replace unhealthy instances
- Version and tag every build artifact (image tags = commit SHA, not just `latest`)
- Keep staging as close to production as possible

**Avoid:**
- Manually SSHing into production servers to "quickly fix something"
- Using `latest` tag for production deployments (not reproducible/traceable)
- Storing secrets in code or `.env` files committed to Git
- Skipping staging under time pressure
- Deploying on Fridays without a solid rollback plan (a common team joke, but a real risk)
- Treating monitoring as optional — a deploy without observability is a deploy you're flying blind on

---

## 18. Glossary

| Term | Meaning |
|---|---|
| **CI** | Continuous Integration — automated build & test on every change |
| **CD** | Continuous Delivery/Deployment — automated release process |
| **Artifact** | A built output (binary, Docker image, package) ready to deploy |
| **Pipeline** | The full automated sequence from commit to deployed |
| **Blast radius** | How much of the system/users are affected if a change goes wrong |
| **IaC** | Infrastructure as Code |
| **Orchestration** | Automated management of containers at scale (Kubernetes, ECS) |
| **Rollback** | Reverting to a previous known-good deployment |
| **MTTR** | Mean Time To Recovery — how fast an incident is resolved |
| **Feature flag** | A toggle to enable/disable functionality without redeploying |