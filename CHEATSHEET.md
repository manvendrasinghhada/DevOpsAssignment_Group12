⚡ Deployment Cheatsheet — CI/CD, Docker & Kubernetes Quick Reference
Git — Deployment-Relevant Commands
git checkout -b feature/my-change      # start new work
git add . && git commit -m "message"   # commit
git push origin feature/my-change      # push branch, opens PR
git tag -a v1.2.0 -m "release 1.2.0"   # tag a release
git push origin v1.2.0                 # push tag
git revert <commit-sha>                # safe rollback of a bad commit
git status                             # check working tree status
git log --oneline -5                   # view recent commits
git branch -a                          # list all branches
git diff                              # view uncommitted changes

Docker — Core Commands
docker build -t myapp:1.0 .            # build image from Dockerfile
docker images                          # list local images
docker run -p 8080:8080 myapp:1.0      # run container, map port
docker ps                              # list running containers
docker ps -a                           # list all containers
docker logs -f <container_id>          # stream logs
docker exec -it <container_id> bash    # shell into running container
docker stop <container_id>             # stop container
docker rm <container_id>               # remove stopped container
docker rmi myapp:1.0                   # remove image
docker tag myapp:1.0 myrepo/myapp:1.0  # tag for registry
docker push myrepo/myapp:1.0           # push to registry
docker pull myrepo/myapp:1.0           # pull from registry
docker inspect <container_id>          # inspect container details
docker stats                            # view resource usage
docker system prune -a                  # clean up unused resources

Docker Compose
docker compose up -d          # start all services in background
docker compose down           # stop and remove containers
docker compose logs -f        # follow logs of all services
docker compose build          # rebuild images
docker compose ps              # list compose services
docker compose restart         # restart services
docker compose pull            # pull latest images

Kubernetes — kubectl Basics
kubectl get pods                         # list pods
kubectl get deployments                  # list deployments
kubectl get services                     # list services
kubectl get nodes                        # list cluster nodes
kubectl describe pod <pod-name>          # detailed pod info / events
kubectl logs -f <pod-name>               # stream pod logs
kubectl exec -it <pod-name> -- bash      # shell into a pod
kubectl apply -f deployment.yaml         # create/update resources
kubectl delete -f deployment.yaml        # delete resources
kubectl rollout status deployment/myapp  # watch rollout progress
kubectl rollout undo deployment/myapp    # rollback to previous revision
kubectl scale deployment/myapp --replicas=5   # scale manually
kubectl get all                          # list common resources
kubectl get pods -o wide                 # show pod IP and node

Kubernetes — ConfigMap & Secret
kubectl create configmap app-config \
  --from-literal=ENV=production

kubectl create secret generic app-secret \
  --from-literal=API_KEY=my-secret-key

kubectl get configmaps                 # list ConfigMaps
kubectl get secrets                     # list Secrets
kubectl describe configmap app-config   # inspect ConfigMap

Kubernetes — Resource Monitoring
kubectl top nodes       # view node CPU/memory usage
kubectl top pods        # view pod CPU/memory usage
kubectl get events      # view cluster events
kubectl describe deployment myapp       # deployment details

Kubernetes — Health Probes
```

### Minimal Deployment + Service YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
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
          image: myrepo/myapp:1.0
          ports:
            
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080
  type: LoadBalancer
```

---

## GitHub Actions — Minimal CI/CD Pipeline

```yaml
# .github/workflows/deploy.yml
name: CI/CD
on:
  push:
    branches: [main]

jobs:
  build-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npm test
      - run: npm run build

  docker-deploy:
    needs: build-test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build and push image
        run: |
          docker build -t myrepo/myapp:${{ github.sha }} .
          docker push myrepo/myapp:${{ github.sha }}
      - name: Deploy
        run: kubectl set image deployment/myapp myapp=myrepo/myapp:${{ github.sha }}
```

---

## Deployment Strategies — Quick Comparison

| Strategy | Downtime | Rollback speed | Use when |
|---|---|---|---|
| Recreate | Yes | Slow | Simple/non-critical apps |
| Rolling | No | Medium | Default for most K8s apps |
| Blue-Green | No | Instant (switch back) | Need instant rollback safety |
| Canary | No | Fast, low blast radius | High-risk/high-traffic releases |

---

## Environment Variables — Quick Reference

```bash
# .env (never commit this — add to .gitignore)
DATABASE_URL=postgres://user:pass@localhost:5432/db
API_KEY=your-secret-key
NODE_ENV=production
```

```bash
echo ".env" >> .gitignore     # always ignore secrets files
```

---

## Common CI/CD Terms — At a Glance

| Term | One-liner |
|---|---|
| Pipeline | Full automated flow: commit → deployed |
| Artifact | Built output (image, binary, package) |
| Trigger | Event that starts a pipeline (push, PR, tag) |
| Job/Stage | A group of steps in a pipeline (e.g. "test", "deploy") |
| Runner/Agent | The machine that executes pipeline jobs |
| Gate | Manual or automatic approval before proceeding |
| Registry | Storage for versioned container images |

---

## Health Check Endpoint Pattern

```
GET /health   → 200 OK  (liveness: is the app running?)
GET /ready    → 200 OK  (readiness: can it accept traffic?)
```

Used by Kubernetes/load balancers to decide whether to route traffic to an instance or restart it.

---

## Quick Rollback Commands

```bash
kubectl rollout undo deployment/myapp          # Kubernetes rollback
git revert <bad-commit-sha> && git push        # code-level revert
docker service update --rollback myapp         # Docker Swarm rollback
```

Git — Deployment-Relevant Commands


bash
git checkout -b feature/my-change      # start new work
git add . && git commit -m "message"   # commit
git push origin feature/my-change      # push branch, opens PR
git tag -a v1.2.0 -m "release 1.2.0"   # tag a release
git push origin v1.2.0                 # push tag (often triggers prod deploy)
git revert <commit-sha>                # safe rollback of a bad commit
Docker — Core Commands


bash
docker build -t myapp:1.0 .            # build image from Dockerfile
docker images                          # list local images
docker run -p 8080:8080 myapp:1.0      # run container, map port
docker ps                              # list running containers
docker ps -a                           # list all containers (incl. stopped)
docker logs -f <container_id>          # stream logs
docker exec -it <container_id> bash    # shell into running container
docker stop <container_id>             # stop container
docker rm <container_id>               # remove stopped container
docker rmi myapp:1.0                   # remove image
docker tag myapp:1.0 myrepo/myapp:1.0  # tag for registry
docker push myrepo/myapp:1.0           # push to registry
docker pull myrepo/myapp:1.0           # pull from registry
docker system prune -a                 # clean up unused images/containers
Minimal Dockerfile (Node example)


dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --production
COPY . .
EXPOSE 8080
CMD ["node", "server.js"]
Minimal Dockerfile (Python example)


dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
Docker Compose


bash
docker compose up -d          # start all services in background
docker compose down           # stop and remove containers
docker compose logs -f        # follow logs of all services
docker compose build          # rebuild images


yaml
# docker-compose.yml
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - DATABASE_URL=postgres://db:5432/app
    depends_on:
      - db
  db:
    image: postgres:16
    environment:
      - POSTGRES_PASSWORD=example
Kubernetes — kubectl Basics


bash
kubectl get pods                         # list pods
kubectl get deployments                  # list deployments
kubectl get services                     # list services
kubectl describe pod <pod-name>          # detailed pod info / events
kubectl logs -f <pod-name>               # stream pod logs
kubectl exec -it <pod-name> -- bash      # shell into a pod
kubectl apply -f deployment.yaml         # create/update resources
kubectl delete -f deployment.yaml        # delete resources
kubectl rollout status deployment/myapp  # watch rollout progress
kubectl rollout undo deployment/myapp    # rollback to previous revision
kubectl scale deployment/myapp --replicas=5   # scale manually
Minimal Deployment + Service YAML


yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
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
          image: myrepo/myapp:1.0
          ports:
            - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080
  type: LoadBalancer
GitHub Actions — Minimal CI/CD Pipeline


yaml
# .github/workflows/deploy.yml
name: CI/CD
on:
  push:
    branches: [main]

jobs:
  build-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npm test
      - run: npm run build

  docker-deploy:
    needs: build-test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build and push image
        run: |
          docker build -t myrepo/myapp:${{ github.sha }} .
          docker push myrepo/myapp:${{ github.sha }}
      - name: Deploy
        run: kubectl set image deployment/myapp myapp=myrepo/myapp:${{ github.sha }}
Deployment Strategies — Quick Comparison
Strategy	Downtime	Rollback speed	Use when
Recreate	Yes	Slow	Simple/non-critical apps
Rolling	No	Medium	Default for most K8s apps
Blue-Green	No	Instant (switch back)	Need instant rollback safety
Canary	No	Fast, low blast radius	High-risk/high-traffic releases
Environment Variables — Quick Reference


bash
# .env (never commit this — add to .gitignore)
DATABASE_URL=postgres://user:pass@localhost:5432/db
API_KEY=your-secret-key
NODE_ENV=production


bash
echo ".env" >> .gitignore     # always ignore secrets files
Common CI/CD Terms — At a Glance
Term	One-liner
Pipeline	Full automated flow: commit → deployed
Artifact	Built output (image, binary, package)
Trigger	Event that starts a pipeline (push, PR, tag)
Job/Stage	A group of steps in a pipeline (e.g. "test", "deploy")
Runner/Agent	The machine that executes pipeline jobs
Gate	Manual or automatic approval before proceeding
Registry	Storage for versioned container images
Health Check Endpoint Pattern


GET /health   → 200 OK  (liveness: is the app running?)
GET /ready    → 200 OK  (readiness: can it accept traffic?)
Used by Kubernetes/load balancers to decide whether to route traffic to an instance or restart it.

Quick Rollback Commands


bash
kubectl rollout undo deployment/myapp          # Kubernetes rollback
git revert <bad-commit-sha> && git push        # code-level revert
docker service update --rollback myapp         # Docker Swarm rollback
Helm — Package Manager for Kubernetes


bash
helm repo add bitnami https://charts.bitnami.com/bitnami   # add a chart repo
helm repo update                        # refresh repo indexes
helm search repo postgres               # search for a chart
helm install myapp ./mychart            # install a chart as a release
helm upgrade myapp ./mychart            # upgrade an existing release
helm upgrade --install myapp ./mychart  # install if missing, upgrade if present
helm rollback myapp 1                   # rollback to revision 1
helm list                               # list releases in current namespace
helm uninstall myapp                    # remove a release
helm template ./mychart                 # render templates locally (no install)
helm show values ./mychart              # print default values.yaml


bash
# override values at install time
helm upgrade --install myapp ./mychart \
  --set image.tag=1.2.0 \
  --set replicaCount=3 \
  -f values.prod.yaml
Kubernetes Secrets & ConfigMaps


bash
kubectl create secret generic db-secret \
  --from-literal=DATABASE_URL=postgres://user:pass@db:5432/app

kubectl create configmap app-config \
  --from-literal=NODE_ENV=production

kubectl get secrets                     # list secrets
kubectl get secret db-secret -o yaml    # view (base64-encoded) secret


yaml
# referencing a secret and configmap in a pod spec
spec:
  containers:
    - name: myapp
      envFrom:
        - secretRef:
            name: db-secret
        - configMapRef:
            name: app-config
Note: base64 is encoding, not encryption — secrets aren't safe at rest without additional tooling (e.g. Sealed Secrets, External Secrets Operator, Vault).

Ingress — Routing External Traffic


yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: myapp-service
                port:
                  number: 80
Resource Requests & Limits


yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
Requests — guaranteed minimum, used for scheduling decisions
Limits — hard ceiling; CPU is throttled, memory over-limit gets the pod OOMKilled
Probes (Liveness / Readiness / Startup)


yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 10

  periodSeconds: 15
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5

CI/CD — Useful Pipeline Steps
Developer
   ↓
Git Commit
   ↓
Pull Request
   ↓
Automated Tests
   ↓
Build Application
   ↓
Build Docker Image
   ↓
Push Image to Registry
   ↓
Deploy to Kubernetes
   ↓
Health Check
   ↓
Production

Docker Image Tagging
docker build -t myrepo/myapp:${GIT_COMMIT} .
docker tag myrepo/myapp:${GIT_COMMIT} myrepo/myapp:latest
docker push myrepo/myapp:${GIT_COMMIT}
docker push myrepo/myapp:latest


Tip: Prefer immutable tags such as commit SHA or release versions for deployments rather than relying only on latest.

Kubernetes Rollout & Rollback
kubectl rollout status deployment/myapp       # check rollout
kubectl rollout history deployment/myapp      # view revisions
kubectl rollout undo deployment/myapp         # rollback
kubectl rollout restart deployment/myapp      # restart pods

Common CI/CD Terms — At a Glance
Term	One-liner
Pipeline	Full automated flow: commit → deployed
Artifact	Built output (image, binary, package)
Trigger	Event that starts a pipeline
Job/Stage	Group of pipeline steps
Runner/Agent	Machine executing pipeline jobs
Gate	Approval before proceeding
Registry	Storage for versioned container images
Deployment	Process of releasing an application
Rollback	Returning to a previous working version
Observability	Monitoring logs, metrics, and traces
Quick Troubleshooting
docker ps -a                         # check container status
docker logs <container_id>           # inspect container logs
docker inspect <container_id>        # inspect container configuration

kubectl get pods                     # check pod status
kubectl describe pod <pod-name>      # inspect pod events
kubectl logs <pod-name>              # check application logs
kubectl get events --sort-by=.lastTimestamp

Quick Rollback Commands
kubectl rollout undo deployment/myapp
git revert <bad-commit-sha> && git push
docker service update --rollback myapp

🚀 Deployment Checklist
☐ Code committed and pushed
☐ Tests passing
☐ Docker image built successfully
☐ Image tagged with version/commit SHA
☐ Image pushed to registry
☐ Kubernetes manifests updated
☐ Deployment applied
☐ Rollout completed successfully
☐ Health checks passing
☐ Application logs verified
  periodSeconds: 10
Namespaces & Context Switching


bash
kubectl get namespaces                          # list namespaces
kubectl config set-context --current --namespace=staging   # switch default namespace
kubectl get pods -n production                  # target a specific namespace
kubectl config get-contexts                     # list available clusters/contexts
kubectl config use-context prod-cluster         # switch cluster context
Debugging Cheatsheet


bash
kubectl get events --sort-by=.metadata.creationTimestamp   # recent cluster events
kubectl top pods                        # CPU/memory usage per pod (needs metrics-server)
kubectl top nodes                       # CPU/memory usage per node
kubectl describe node <node-name>       # node capacity, taints, conditions
kubectl get pod <pod-name> -o yaml      # full pod spec/status dump
kubectl logs <pod-name> --previous      # logs from a crashed/restarted container
kubectl port-forward svc/myapp 8080:80  # tunnel a service to localhost
Monitoring & Observability Stack (Common Picks)
Layer	Common tools
Metrics	Prometheus, Datadog, CloudWatch
Dashboards	Grafana
Logs	Loki, ELK/EFK stack, Fluentd
Tracing	Jaeger, OpenTelemetry, Zipkin
Alerting	Alertmanager, PagerDuty, Opsgenie
Terraform — Infra-as-Code Basics


bash
terraform init        # download providers, set up backend
terraform plan         # preview changes
terraform apply         # apply changes
terraform destroy       # tear down managed infra
terraform fmt            # auto-format .tf files
terraform state list      # list resources in state

Git

git status → check changes
git switch -c branch → new branch
git add . → stage changes
git commit -m "msg" → save changes
git push → upload changes
git pull → get latest changes
git merge → combine branches
git revert → undo commit safely

Docker

Docker → containerization
Image → application package/template
Container → running image
docker build → create image
docker run → start container
docker ps → running containers
docker logs → container logs
docker push/pull → registry se upload/download

Docker Compose

Multiple containers/services manage karta hai.
docker compose up -d → start
docker compose down → stop/remove
docker compose logs → logs
docker compose build → rebuild

Kubernetes

Kubernetes → container orchestration
Pod → smallest deployable unit
Deployment → manages Pods
Service → stable network access
Ingress → HTTP/HTTPS routing
ConfigMap → non-sensitive config
Secret → sensitive config
kubectl get pods → Pods check
kubectl apply -f file.yaml → deploy/update
kubectl logs pod → logs
kubectl rollout undo → rollback
kubectl scale → replicas increase/decrease

CI/CD

CI → Build + Test automatically
CD → Deliver/Deploy automatically

Pipeline:
Git → Build → Test → Docker → Registry → Kubernetes → Deploy → Monitor

Deployment Strategies

Rolling → gradually old Pods replace
Blue-Green → two environments, traffic switch
Canary → small percentage users first
