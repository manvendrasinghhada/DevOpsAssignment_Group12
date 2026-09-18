Project Architecture

The deployment architecture can be represented as:

                    Developer
                        |
                        v
                 GitHub Repository
                        |
                        v
                GitHub Actions CI/CD
                        |
             +----------+----------+
             |                     |
             v                     v
        Automated Tests       Docker Build
                                   |
                                   v
                          Container Registry
                                   |
                         +---------+---------+
                         |                   |
                         v                   v
                      Staging            Production
                         |                   |
                         v                   v
                    Kubernetes          Kubernetes
                         |                   |
                         +---------+---------+
                                   |
                                   v
                         Monitoring & Logging

The architecture separates source control, continuous integration, container packaging, deployment, and monitoring. This makes each stage easier to manage, troubleshoot, and improve independently.

Suggested Repository Structure

A production-oriented DevOps repository can follow a structure similar to:

devops-project/
│
├── .github/
│   └── workflows/
│       └── deploy.yml
│
├── app/
│   ├── src/
│   ├── tests/
│   └── package.json
│
├── docker/
│   └── Dockerfile
│
├── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   └── ingress.yaml
│
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
│
├── docs/
│   ├── architecture.md
│   └── deployment.md
│
├── .gitignore
├── README.md
└── LICENSE

The exact structure depends on the application and deployment platform. Documentation-only repositories do not need every directory shown above.

Environment Management

A DevOps project should clearly separate environments.

Development

The development environment is used by developers to build and test new features.

Typical characteristics:

- frequent code changes
- local Docker containers
- development databases
- debugging enabled
- relaxed resource requirements

Staging

Staging should closely resemble production.

Typical activities include:

- integration testing
- smoke testing
- deployment validation
- database migration testing
- performance checks
- release verification

Production

Production is the environment accessed by real users.

Production should have:

- restricted access
- secure secrets
- monitoring and alerting
- backups
- controlled deployments
- rollback procedures
- appropriate resource limits

Keeping environments consistent reduces the common problem of "works on my machine."

Git Branching Strategy

A simple branching strategy can be used for this project:

main
 |
 +---- feature/login
 |
 +---- feature/payment
 |
 +---- bugfix/api-error

Recommended workflow:

1. Create a feature or bug-fix branch.
2. Make changes locally.
3. Push the branch to GitHub.
4. Create a Pull Request.
5. Run automated CI checks.
6. Review the changes.
7. Merge into "main".
8. Trigger the deployment pipeline.

Protected branches can be configured so that direct pushes to "main" are restricted.

Example Git Commands

# Clone repository
git clone <repository-url>

# Enter project
cd devops-project

# Create feature branch
git checkout -b feature/new-feature

# Check changes
git status

# Add changes
git add .

# Commit
git commit -m "Add new deployment feature"

# Push branch
git push origin feature/new-feature

After the Pull Request is reviewed and merged, the CI/CD pipeline can automatically build and deploy the application.

Example GitHub Actions Pipeline

A simplified workflow can look like:

name: DevOps CI/CD

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:

  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Install dependencies
        run: |
          echo "Install project dependencies here"

      - name: Run tests
        run: |
          echo "Run automated tests here"

  build:
    needs: test
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Build Docker image
        run: |
          docker build -t myapp:${{ github.sha }} .

  deploy:
    needs: build
    runs-on: ubuntu-latest

    steps:
      - name: Deploy application
        run: |
          echo "Deploy application to the target environment"

This is a template workflow. The commands should be replaced with the actual application's dependency installation, testing, registry authentication, and deployment commands.

Dockerfile Best Practices

A production Dockerfile should be optimized for security, performance, and reproducibility.

Recommended practices:

- use a small official base image
- use multi-stage builds when appropriate
- avoid running applications as root
- copy only required files
- use ".dockerignore"
- pin important dependency versions
- avoid storing secrets inside the image
- expose only the required application port

Example:

FROM node:22-alpine

WORKDIR /app

COPY package*.json ./

RUN npm ci --omit=dev

COPY . .

USER node

EXPOSE 8080

CMD ["npm", "start"]

The Dockerfile must be adapted to the actual programming language and application framework.

Docker Image Versioning

Every production image should have an identifiable version.

Recommended examples:

myapp:1.0.0
myapp:1.1.0
myapp:2026-09-18
myapp:<commit-sha>

Using the Git commit SHA is particularly useful because it creates a direct relationship between a deployed container and the source code that produced it.

Avoid relying only on:

myapp:latest

because "latest" does not uniquely identify a release.

Kubernetes Resource Example

A basic Kubernetes deployment can be structured like:

apiVersion: apps/v1
kind: Deployment

metadata:
  name: myapp

spec:
  replicas: 3

  strategy:
    type: RollingUpdate

  selector:
    matchLabels:
      app: myapp

  template:
    metadata:
      labels:
        app: myapp

    spec:
      containers:
        - name: myapp
          image: myrepo/myapp:VERSION

          ports:
            - containerPort: 8080

          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"

            limits:
              cpu: "500m"
              memory: "512Mi"

          readinessProbe:
            httpGet:
              path: /ready
              port: 8080

          livenessProbe:
            httpGet:
              path: /health
              port: 8080

This example demonstrates several production concepts including replicas, rolling updates, resource management, and health probes.

Service Configuration

A Kubernetes "Service" provides stable network access to application pods.

Example:

apiVersion: v1
kind: Service

metadata:
  name: myapp-service

spec:
  selector:
    app: myapp

  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080

  type: ClusterIP

The service forwards traffic from port "80" to the application's container port "8080".

Deployment Verification Commands

After deployment, verify the complete Kubernetes workload:

kubectl get deployments
kubectl get pods
kubectl get services
kubectl get events

Check application logs:

kubectl logs deployment/myapp

Check rollout progress:

kubectl rollout status deployment/myapp

Check the deployment configuration:

kubectl describe deployment myapp

If a pod is failing:

kubectl describe pod <pod-name>
kubectl logs <pod-name>

These commands are useful during deployment troubleshooting.

Troubleshooting Guide

Pod is stuck in Pending

Possible causes:

- insufficient cluster resources
- scheduling constraints
- missing persistent volume
- node availability problems

Useful command:

kubectl describe pod <pod-name>

Container keeps restarting

Possible causes:

- application startup failure
- incorrect environment variables
- failed liveness probe
- missing dependency
- incorrect command or entry point

Check:

kubectl logs <pod-name>

ImagePullBackOff

Possible causes:

- incorrect image name
- incorrect image tag
- private registry authentication failure
- image does not exist

Check:

kubectl describe pod <pod-name>

Application is running but unreachable

Check:

kubectl get pods
kubectl get service
kubectl describe service myapp-service

Verify that the service selector matches the labels used by the pods.

Deployment Metrics

Important deployment metrics include:

Metric| Purpose
Deployment Frequency| Measures how often releases occur
Lead Time for Changes| Measures time from code change to deployment
Change Failure Rate| Measures deployments that cause failures
Mean Time to Recovery| Measures recovery speed after failure
Availability| Measures service uptime
Error Rate| Measures failed requests
Latency| Measures application response time

These metrics can help teams understand the effectiveness and reliability of their software delivery process.

Disaster Recovery

A production deployment strategy should also consider disaster recovery.

Important practices include:

- regular database backups
- tested backup restoration
- documented recovery procedures
- infrastructure definitions stored in Git
- separate recovery environments when required
- defined recovery objectives
- periodic disaster-recovery exercises

Two important concepts are:

RPO — Recovery Point Objective

The maximum acceptable amount of data loss measured in time.

RTO — Recovery Time Objective

The maximum acceptable time required to restore the service.

DevOps Security Pipeline

Security checks can be integrated into CI/CD:

Code
 |
 v
Lint
 |
 v
Unit Tests
 |
 v
Dependency Scan
 |
 v
SAST
 |
 v
Docker Build
 |
 v
Container Scan
 |
 v
Deploy Staging
 |
 v
Smoke Test
 |
 v
Production

Security should be automated wherever practical so that vulnerabilities are detected before reaching production.

Release Versioning

A consistent release naming convention makes deployments easier to track.

Example:

v1.0.0
v1.1.0
v1.1.1
v2.0.0

Semantic versioning generally follows:

MAJOR.MINOR.PATCH

- MAJOR: incompatible changes
- MINOR: backward-compatible features
- PATCH: backward-compatible bug fixes

The project's actual versioning policy should be documented and followed consistently.

Deployment Documentation

Every production deployment should leave enough information for another team member to understand what happened.

A deployment record can include:

Release Version:
Commit SHA:
Deployment Date:
Environment:
Image:
Database Migration:
Deployment Strategy:
Health Check:
Rollback Version:
Deployment Owner:
Status:

Good deployment documentation improves traceability and makes incident investigation easier.

Future Improvements

The project can be extended with additional DevOps capabilities:

- automated Docker image scanning
- Terraform-based infrastructure provisioning
- Kubernetes Horizontal Pod Autoscaler
- centralized logging
- Prometheus metrics
- Grafana dashboards
- automated smoke tests
- blue-green deployment
- canary deployment
- Slack or email deployment notifications
- dependency security scanning
- infrastructure drift detection
- automated rollback
- cloud deployment using AWS, Azure, or Google Cloud

These features can be added incrementally as the project becomes more advanced.

Project Learning Outcomes

After completing this project, the learner should understand:

- Git and GitHub-based development workflows
- CI/CD pipeline design
- Docker image creation and management
- Container registry usage
- Kubernetes deployment concepts
- configuration and secret management
- infrastructure as code
- monitoring and observability
- deployment strategies
- rollback and recovery
- DevOps security practices
- production troubleshooting

Conclusion

This project demonstrates how DevOps connects development, testing, deployment, infrastructure, security, and monitoring into a continuous software delivery process.

The main objective is not simply to deploy an application once, but to create a repeatable, traceable, automated, and recoverable deployment process.

A mature deployment pipeline should make it easy to answer five questions:

1. What changed?
2. Which version is running?
3. Where is it deployed?
4. Is the application healthy?
5. How can we safely recover if something goes wrong?

By combining Git, CI/CD, Docker, Kubernetes, IaC, security, monitoring, and proper release management, this project provides a practical foundation for understanding modern DevOps deployment.
