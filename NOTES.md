# AWS DEPLOYMENT SERVICES

## 1. Introduction to AWS

**AWS (Amazon Web Services)** is a cloud computing platform provided by Amazon.

AWS provides many services that allow organizations to:

* Host applications
* Store data
* Run virtual machines
* Deploy containers
* Manage databases
* Build CI/CD pipelines
* Monitor applications
* Scale applications automatically
* Implement serverless applications

Instead of purchasing and maintaining physical servers, organizations can use AWS resources over the internet.

---

# 2. AWS in Application Deployment

AWS can be used at different stages of the deployment process.

A simplified deployment flow is:

```text
Developer
    ↓
GitHub
    ↓
CI/CD Pipeline
    ↓
Build Application
    ↓
Docker Image
    ↓
AWS
    ↓
Production Application
```

AWS provides different services depending on how the application needs to be deployed.

For example:

```text
Virtual Machine
      ↓
     EC2

Containers
      ↓
 ECS / Fargate

Kubernetes
      ↓
     EKS

Platform as a Service
      ↓
Elastic Beanstalk

Serverless
      ↓
    Lambda
```

---

# 3. Amazon EC2

**EC2 (Elastic Compute Cloud)** provides virtual machines in the AWS cloud.

Instead of purchasing a physical server, we can create a virtual server using EC2.

Example:

```text
Developer
    ↓
AWS EC2
    ↓
Application
    ↓
Users
```

An EC2 instance can run:

* Linux
* Windows
* Web servers
* Backend applications
* Databases
* Docker
* Other software

---

# 4. EC2 Deployment Process

A simple EC2 deployment can work like this:

```text
Source Code
    ↓
GitHub
    ↓
CI/CD
    ↓
Build Application
    ↓
EC2 Server
    ↓
Run Application
    ↓
Users
```

The developer can configure the EC2 server and install the required software.

For example:

```text
EC2
 ├── Operating System
 ├── Docker
 ├── Application
 └── Web Server
```

---

# 5. Advantages of EC2

EC2 provides a high level of control over the server.

Advantages include:

* Full operating-system control
* Flexible configurations
* Support for many operating systems
* Ability to install custom software
* Suitable for traditional applications
* Supports Docker
* Can be integrated with other AWS services

---

# 6. Disadvantages of EC2

EC2 requires more management than fully managed services.

The team may need to manage:

* Operating system
* Security updates
* Software installation
* Server configuration
* Scaling
* Monitoring
* Networking

Therefore, EC2 provides flexibility but also increases operational responsibility.

---

# 7. Amazon ECS

**ECS (Elastic Container Service)** is an AWS service for running and managing containers.

Instead of directly running an application on a virtual machine, the application can be packaged as a Docker container.

```text
Application
    ↓
Docker Image
    ↓
ECS
    ↓
Container
    ↓
Users
```

ECS is useful for container-based deployment.

---

# 8. Why Use ECS?

ECS makes it easier to deploy and manage multiple containers.

For example:

```text
ECS Cluster
     |
 ┌───┼────┐
 ↓   ↓    ↓
App  API  Worker
```

Each service can run its own containers.

---

# 9. ECS and CI/CD

ECS can be integrated into a CI/CD pipeline.

Example:

```text
Developer
    ↓
GitHub
    ↓
GitHub Actions
    ↓
Build Docker Image
    ↓
Push Image
    ↓
AWS ECS
    ↓
Deploy Container
```

This allows application deployments to become automated.

---

# 10. AWS Fargate

**AWS Fargate** is a serverless compute engine for containers.

The main idea is:

> Developers run containers without managing the underlying servers.

With traditional EC2:

```text
Application
    ↓
Container
    ↓
EC2
    ↓
Physical Infrastructure
```

With Fargate:

```text
Application
    ↓
Container
    ↓
Fargate
    ↓
AWS Infrastructure
```

AWS manages the underlying compute infrastructure.

---

# 11. Advantages of Fargate

Fargate can reduce operational work.

The team does not need to manage the underlying servers in the same way as EC2-based container deployments.

Benefits include:

* Server management is reduced
* Easy container deployment
* Works with ECS
* Supports scalable applications
* Useful for microservices
* Fits well with CI/CD

---

# 12. ECS vs Fargate

| ECS                             | Fargate                                      |
| ------------------------------- | -------------------------------------------- |
| Container orchestration service | Serverless compute for containers            |
| Manages container workloads     | Runs container workloads                     |
| Can use EC2 or Fargate capacity | Does not require users to manage EC2 servers |
| More infrastructure choices     | Less server management                       |

A simple way to remember:

**ECS = Manage containers**

**Fargate = Run containers without managing servers**

---

# 13. Amazon EKS

**EKS (Elastic Kubernetes Service)** is AWS's managed Kubernetes service.

Kubernetes is used to manage containerized applications at scale.

```text
Docker Containers
       ↓
   Kubernetes
       ↓
      EKS
       ↓
      AWS
```

EKS is useful when an organization wants to use Kubernetes while having AWS manage the Kubernetes control plane.

---

# 14. EKS Deployment

A simplified EKS deployment looks like:

```text
Developer
    ↓
GitHub
    ↓
CI/CD
    ↓
Docker Build
    ↓
Container Registry
    ↓
EKS
    ↓
Kubernetes Pods
    ↓
Users
```

---

# 15. EKS and Kubernetes

EKS uses Kubernetes concepts such as:

* Pods
* Deployments
* Services
* Ingress
* ConfigMaps
* Secrets
* Nodes

Example:

```text
EKS Cluster
     |
 ┌───┼────┐
 ↓   ↓    ↓
Pod Pod  Pod
```

Kubernetes can maintain the desired number of application instances.

---

# 16. Why Use EKS?

EKS can be useful for organizations that need:

* Kubernetes
* Container orchestration
* Multiple services
* Application scaling
* Self-healing
* Rolling deployments
* Kubernetes-based workflows

---

# 17. EKS vs ECS

| ECS                                | EKS                                    |
| ---------------------------------- | -------------------------------------- |
| AWS-native container orchestration | Managed Kubernetes                     |
| Simpler AWS-specific approach      | Kubernetes ecosystem                   |
| Easier for teams focused on AWS    | Useful for Kubernetes expertise        |
| Less Kubernetes complexity         | More Kubernetes flexibility            |
| Good for AWS-centric deployments   | Good for Kubernetes-based environments |

---

# 18. AWS Elastic Beanstalk

**Elastic Beanstalk** is a Platform as a Service (PaaS).

It makes application deployment easier by handling much of the underlying infrastructure configuration.

Instead of manually configuring everything:

```text
Code
 ↓
Elastic Beanstalk
 ↓
AWS Infrastructure
 ↓
Application
```

The developer can focus more on application code.

---

# 19. Why Use Elastic Beanstalk?

Elastic Beanstalk is useful when developers want relatively simple application deployment without manually managing every infrastructure component.

It can handle aspects such as:

* Application deployment
* Infrastructure provisioning
* Scaling
* Load balancing
* Application health monitoring

---

# 20. Elastic Beanstalk Deployment

Example:

```text
Developer
    ↓
Application Code
    ↓
Elastic Beanstalk
    ↓
AWS Resources
    ↓
Running Application
```

It can therefore simplify deployment for traditional web applications.

---

# 21. AWS Lambda

**AWS Lambda** is a serverless computing service.

With Lambda, developers upload code that runs in response to events.

The developer does not manage a traditional server for the function.

Example:

```text
Event
  ↓
Lambda Function
  ↓
Execute Code
  ↓
Result
```

---

# 22. Lambda Example

Suppose a user uploads an image.

The workflow could be:

```text
User Uploads Image
       ↓
Amazon S3
       ↓
Lambda Trigger
       ↓
Image Processing
       ↓
Result
```

Lambda runs the required code when the event occurs.

---

# 23. Lambda and Serverless

Lambda is part of the **serverless** model.

Serverless does not mean that servers do not exist.

It means that AWS manages the underlying server infrastructure for the developer.

The developer primarily focuses on:

**Writing and deploying functions.**

---

# 24. Advantages of Lambda

Lambda can be useful because:

* No traditional server management
* Automatically scales with requests
* Event-driven
* Suitable for small functions
* Can integrate with many AWS services
* Pay-per-use model can be useful for suitable workloads

---

# 25. Lambda Limitations

Lambda is not suitable for every application.

It is particularly designed for functions and event-driven workloads.

Applications that require:

* Long-running processes
* Full server control
* Persistent server environments

may be better suited to other deployment models.

---

# 26. EC2 vs ECS vs EKS vs Lambda

This is an important comparison.

| Service               | Main Purpose                 |
| --------------------- | ---------------------------- |
| **EC2**               | Virtual machines             |
| **ECS**               | Container orchestration      |
| **Fargate**           | Serverless container compute |
| **EKS**               | Managed Kubernetes           |
| **Elastic Beanstalk** | Platform as a Service        |
| **Lambda**            | Serverless functions         |

---

# 27. Easy Way to Remember AWS Services

```text
Need a Virtual Machine?
        ↓
       EC2

Need Containers?
        ↓
       ECS

Need Containers without managing servers?
        ↓
     Fargate

Need Kubernetes?
        ↓
       EKS

Need Easy Application Deployment?
        ↓
Elastic Beanstalk

Need Serverless Functions?
        ↓
      Lambda
```

---

# 28. AWS Container Deployment Architecture

A modern Docker-based AWS deployment may look like:

```text
                Developer
                    ↓
                  GitHub
                    ↓
                CI Pipeline
                    ↓
              Docker Build
                    ↓
              Docker Image
                    ↓
              Image Registry
                    ↓
              ECS / Fargate
                    ↓
              Running Container
                    ↓
                  Users
```

---

# 29. AWS with CI/CD

AWS services can be integrated with CI/CD tools.

For example:

```text
GitHub
   ↓
GitHub Actions
   ↓
Build
   ↓
Test
   ↓
Docker Image
   ↓
AWS Container Registry
   ↓
ECS / EKS
   ↓
Production
```

This creates an automated deployment pipeline.

---

# 30. AWS Container Registry

Container images need somewhere to be stored.

AWS provides **Amazon ECR (Elastic Container Registry)** for container images.

Example:

```text
Docker Build
     ↓
Docker Image
     ↓
Amazon ECR
     ↓
ECS / EKS
```

The deployment system can retrieve the image from the registry.

---

# 31. Why ECR is Important

ECR provides a place to store container images used by AWS container services.

A CI/CD pipeline can:

1. Build the Docker image.
2. Tag the image.
3. Push the image to ECR.
4. Deploy that image to ECS or EKS.

---

# 32. Versioning Docker Images

A good deployment system should identify exactly which application version is being deployed.

For example:

```text
myapp:abc123
```

where `abc123` can represent a commit identifier.

This makes deployments easier to trace.

---

# 33. AWS Deployment with Docker

The complete process can be:

```text
Source Code
     ↓
GitHub
     ↓
CI
     ↓
Docker Build
     ↓
Docker Image
     ↓
Amazon ECR
     ↓
ECS/Fargate
     ↓
Production
```

---

# 34. AWS Load Balancing

For applications receiving traffic from many users, a load balancer can distribute requests across application instances.

Conceptually:

```text
              Users
                ↓
          Load Balancer
          /     |      \
         ↓      ↓       ↓
       App     App     App
```

This can improve availability and distribute traffic.

---

# 35. AWS Auto Scaling

Applications may experience changing traffic.

Example:

```text
Normal Traffic
     ↓
2 Instances
```

During high traffic:

```text
High Traffic
     ↓
5 Instances
```

Auto Scaling can help applications adjust capacity based on demand.

---

# 36. AWS Monitoring

Deployment is not complete just because the application starts.

The application should be monitored.

AWS provides monitoring capabilities through services such as **Amazon CloudWatch**.

Important things to monitor include:

* CPU usage
* Memory-related metrics where available
* Request activity
* Errors
* Application logs
* Infrastructure health

---

# 37. AWS Logging

Logs help engineers understand what is happening inside applications and infrastructure.

Example:

```text
Application
    ↓
Logs
    ↓
Cloud Logging/Monitoring
    ↓
Engineer
```

Logs are useful when investigating deployment problems.

---

# 38. AWS Security

Cloud deployment must also consider security.

Important areas include:

* Identity and access management
* Network security
* Secrets
* Encryption
* Access control
* Security groups
* Least-privilege permissions

Credentials should not be hardcoded inside application code.

---

# 39. AWS IAM

**IAM (Identity and Access Management)** controls who can access AWS resources and what actions they can perform.

Example:

```text
Developer
   ↓
IAM Permissions
   ↓
AWS Resources
```

Different users and services should receive only the permissions they actually need.

---

# 40. AWS Deployment Environments

AWS can support multiple environments.

For example:

```text
Development AWS Environment
          ↓
Staging AWS Environment
          ↓
Production AWS Environment
```

This follows the general DevOps approach of testing before production.

---

# 41. Staging on AWS

A staging environment can be created to closely resemble production.

Example:

```text
Staging
 ↓
ECS/Fargate
 ↓
Application
```

Automated tests can run against staging before production deployment.

---

# 42. Production on AWS

After testing and approval:

```text
Staging
   ↓
Approval
   ↓
Production
```

The same application artifact should ideally be promoted from staging to production rather than rebuilding a different artifact.

---

# 43. AWS Rolling Deployment

A new version can be gradually introduced.

```text
Old Old Old
   ↓
New Old Old
   ↓
New New Old
   ↓
New New New
```

This reduces the need for complete downtime.

---

# 44. AWS Blue-Green Deployment

Two environments can be maintained.

```text
BLUE
Current Version

GREEN
New Version
```

Traffic can be switched between them.

```text
Users
 ↓
BLUE
```

Then:

```text
Users
 ↓
GREEN
```

If the new version has problems, traffic can be switched back.

---

# 45. AWS Canary Deployment

A small amount of traffic can be sent to a new version first.

Example:

```text
95% → Old Version
5%  → New Version
```

If the new version is healthy:

```text
80% → Old
20% → New
```

Eventually:

```text
100% → New Version
```

This reduces the initial blast radius.

---

# 46. AWS Rollback

If a deployment causes problems:

```text
New Version
     ↓
Problem
     ↓
Detection
     ↓
Rollback
     ↓
Previous Version
```

Rollback is an important part of reliable deployment.

---

# 47. AWS Disaster Recovery

Organizations also need plans for major failures.

Disaster recovery can involve:

* Backups
* Multiple availability zones
* Replication
* Recovery procedures
* Monitoring
* Automated failover

The exact architecture depends on application requirements.

---

# 48. AWS Scalability

One major advantage of cloud deployment is the ability to scale infrastructure.

Example:

```text
100 Users
   ↓
Small Capacity
```

If traffic increases:

```text
100,000 Users
      ↓
Larger Capacity
```

AWS provides many services and mechanisms that can support scaling.

---

# 49. AWS Availability

Applications should ideally continue operating even if one infrastructure component fails.

A simplified architecture may look like:

```text
             Load Balancer
             /           \
            ↓             ↓
       Instance A     Instance B
```

If one instance fails, another can continue serving requests.

---

# 50. AWS Microservices

AWS container services are commonly useful for microservice architectures.

Example:

```text
                 Application
                     |
       ┌─────────────┼─────────────┐
       ↓             ↓             ↓
     User          Payment        Order
    Service        Service        Service
```

Each service can potentially be deployed independently.

---

# 51. AWS Serverless Architecture

A serverless application might look like:

```text
User
 ↓
API
 ↓
Lambda
 ↓
Database
```

The developer does not need to manage traditional servers for the Lambda functions.

---

# 52. AWS PaaS Architecture

With Elastic Beanstalk:

```text
Developer
    ↓
Application Code
    ↓
Elastic Beanstalk
    ↓
Managed AWS Environment
    ↓
Users
```

This simplifies the deployment process.

---

# 53. Choosing the Right AWS Service

The choice depends on the project.

### Choose EC2 when:

You need more control over a virtual machine.

### Choose ECS when:

You want AWS-native container orchestration.

### Choose Fargate when:

You want to run containers without managing the underlying servers.

### Choose EKS when:

You specifically need Kubernetes.

### Choose Elastic Beanstalk when:

You want a simpler managed application deployment platform.

### Choose Lambda when:

Your application fits an event-driven/serverless function model.

---

# 54. AWS Services in One Diagram

```text
                         AWS
                          |
       ┌──────────────────┼──────────────────┐
       ↓                  ↓                  ↓
     Compute           Containers         Serverless
       |                  |                  |
      EC2            ECS / Fargate        Lambda
       |
      VMs

                         |
                         ↓
                       EKS
                    Kubernetes

                         |
                         ↓
                 Elastic Beanstalk
                       PaaS
```

---

# 55. Complete AWS CI/CD Example

A realistic deployment pipeline can be:

```text
Developer
    ↓
Feature Branch
    ↓
Pull Request
    ↓
GitHub Actions
    ↓
Lint
    ↓
Unit Tests
    ↓
Build
    ↓
Docker Image
    ↓
Amazon ECR
    ↓
Staging
    ↓
Automated Tests
    ↓
Approval
    ↓
Production
    ↓
ECS / Fargate / EKS
    ↓
Cloud Monitoring
```

---

# 56. AWS Deployment Benefits

AWS provides:

* Cloud infrastructure
* Flexible deployment options
* Container services
* Kubernetes
* Serverless computing
* Monitoring
* Security services
* Scalability
* High availability options
* Integration with CI/CD

---

# 57. AWS Deployment Challenges

AWS also introduces complexity.

Common challenges include:

* Large number of services
* Configuration complexity
* Security configuration
* Cost management
* Networking complexity
* Monitoring requirements
* IAM permissions
* Infrastructure management

Therefore, teams should select services according to their actual requirements.

---

# 58. AWS Cost Management

Cloud resources can generate costs based on usage and configuration.

Therefore, teams should monitor:

* Compute usage
* Storage
* Network traffic
* Database usage
* Container workloads
* Serverless invocations

Unused resources should be identified and removed where appropriate.

---

# 59. AWS and Infrastructure as Code

AWS infrastructure can be managed using Infrastructure as Code tools.

For example:

```text
Terraform
     ↓
AWS Resources
```

or:

```text
CloudFormation
     ↓
AWS Resources
```

This makes infrastructure more repeatable and easier to version-control.

---

# 60. AWS and Docker

Docker fits naturally into AWS container deployment.

```text
Application
    ↓
Dockerfile
    ↓
Docker Image
    ↓
Amazon ECR
    ↓
ECS / Fargate / EKS
    ↓
Production
```

---

# 61. AWS and Kubernetes

For Kubernetes-based applications:

```text
Developer
    ↓
Docker
    ↓
Container Image
    ↓
Registry
    ↓
Amazon EKS
    ↓
Kubernetes
    ↓
Pods
    ↓
Users
```

---

# 62. AWS and GitHub Actions

GitHub Actions can automate AWS deployment.

A typical workflow is:

```text
Git Push
   ↓
GitHub Actions
   ↓
Test
   ↓
Build
   ↓
Docker Image
   ↓
Push to ECR
   ↓
Deploy to AWS
```

This reduces manual deployment work.

---

# 63. AWS Production Best Practice

A production deployment should ideally have:

```text
Version Control
       +
CI/CD
       +
Immutable Artifacts
       +
Secure Secrets
       +
Monitoring
       +
Health Checks
       +
Rollback
```

Together these practices improve deployment reliability.

---

# 64. AWS Deployment — Final Comparison

| AWS Service           | Type                         | Main Use                                |
| --------------------- | ---------------------------- | --------------------------------------- |
| **EC2**               | IaaS / VM                    | Virtual servers                         |
| **ECS**               | Container orchestration      | Manage containers                       |
| **Fargate**           | Serverless container compute | Run containers without managing servers |
| **EKS**               | Managed Kubernetes           | Kubernetes workloads                    |
| **Elastic Beanstalk** | PaaS                         | Simplified application deployment       |
| **Lambda**            | Serverless                   | Event-driven functions                  |
| **ECR**               | Container Registry           | Store Docker images                     |
| **CloudWatch**        | Monitoring                   | Logs and metrics                        |
| **IAM**               | Security                     | Access control                          |

---

# 65. Final AWS Deployment Flow

The complete concept can be remembered as:

```text
                    DEVELOPER
                        ↓
                      GIT
                        ↓
                    CI/CD
                        ↓
                 BUILD + TEST
                        ↓
                  DOCKER IMAGE
                        ↓
                     ECR
                        ↓
          ┌─────────────┼─────────────┐
          ↓             ↓             ↓
        ECS           EKS          FARGATE
          |             |             |
          └─────────────┼─────────────┘
                        ↓
                    PRODUCTION
                        ↓
                    MONITORING
                        ↓
                 HEALTH CHECKS
                        ↓
               ┌────────┴────────┐
               ↓                 ↓
            Healthy           Failure
               ↓                 ↓
          Continue          Rollback
```

## Short Exam Answer

**AWS provides multiple services for application deployment. EC2 provides virtual machines, ECS manages containers, Fargate runs containers without requiring users to manage the underlying servers, EKS provides managed Kubernetes, Elastic Beanstalk simplifies application deployment through a PaaS model, and Lambda provides serverless event-driven computing. These services can be integrated with CI/CD pipelines to automate application build, testing, packaging, deployment, monitoring, and rollback.**
