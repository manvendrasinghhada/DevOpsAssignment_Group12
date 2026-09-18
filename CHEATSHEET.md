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
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 10

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
