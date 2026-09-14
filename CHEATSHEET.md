# ⚡ Deployment Cheatsheet — CI/CD, Docker & Kubernetes Quick Reference

---

## Git — Deployment-Relevant Commands

```bash
git checkout -b feature/my-change      # start new work
git add . && git commit -m "message"   # commit
git push origin feature/my-change      # push branch, opens PR
git tag -a v1.2.0 -m "release 1.2.0"   # tag a release
git push origin v1.2.0                 # push tag (often triggers prod deploy)
git revert <commit-sha>                # safe rollback of a bad commit
```

---

## Docker — Core Commands

```bash
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
```

### Minimal Dockerfile (Node example)

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --production
COPY . .
EXPOSE 8080
CMD ["node", "server.js"]
```

### Minimal Dockerfile (Python example)

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

## Docker Compose

```bash
docker compose up -d          # start all services in background
docker compose down           # stop and remove containers
docker compose logs -f        # follow logs of all services
docker compose build          # rebuild images
```

```yaml
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
```

---

## Kubernetes — `kubectl` Basics

```bash
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