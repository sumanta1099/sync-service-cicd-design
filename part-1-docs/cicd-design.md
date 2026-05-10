# CI/CD Design for sync-service

## 1. Overview

The backend application connected to MongoDB and deployed on GCP VMs.

The CI/CD pipeline is implemented using Jenkins.

Environments:

- QA
- Staging
- Production

Goals:

- Automated build and deployment
- Safe production releases
- Fast rollback
- Secure secret management
- Minimal downtime deployment

---

# 2. Branching Strategy

| Branch | Environment | Purpose |
|---|---|---|
| feature/* | Local/PR Validation | developer feature work |
| development | QA | Integration testing |
| staging | Staging | Pre-production testing |
| production | prod | Stable production code |

---

## Branch Flow

```text
feature/* → development → staging → prod
```

---

## Environment Mapping

| Branch | Deployment Target |
|---|---|
| development | QA VM |
| staging | Staging VM |
| prod | Production VM |

---

## Avoiding Accidental Production Deployments

### Methods Used

### 1. Protected prod Branch

- Only approved pull requests can merge
- Direct push disabled

---

### 2. Manual Approval in Jenkins

Production deployment requires:

- Manual approval
- Authorized DevOps/Admin users only

---

### 3. Separate Jenkins Credentials

Different credentials per environment:

- QA credentials
- Staging credentials
- Production credentials

---

### 4. Production Deployment Trigger Only From `prod`

Only `prod` branch triggers production deployment.

---

# 3. Jenkins Pipeline Design

## Pipeline Stages

```text
Checkout
↓
Build
↓
Unit Test
↓
Code Quality Scan
↓
Build Docker Image
↓
Push Docker Image
↓
Deploy
↓
Health Check
↓
Rollback (if failed)
```

---

# 4. PR vs Merge Workflow

## Pull Request Workflow

When PR is created:

### Pipeline Actions

- Checkout code
- Compile application
- Run unit tests
- Run static code analysis
- Build Docker image (without push)

Purpose:

- Validate code quality
- Prevent broken code merge

No deployment happens during PR.

---

## Merge Workflow

### Merge to `development`

Deploys automatically to QA.

### Merge to `staging`

Deploys automatically to Staging.

### Merge to `prod`

Requires manual approval before production deployment.

---

# 5. Rollback Strategy

Rollback is required if:

- Deployment fails
- Health check fails
- Application crash occurs

---

## Rollback Method

### Docker Image Versioning

Each build tagged with:

```text
build-number
git-commit-id
```

Example:

```text
sync-service:v145
sync-service:commit-a1b2c3
```

---

## Rollback Process

1. Stop failed container
2. Pull previous stable image
3. Start previous container
4. Run health check

---

## Automated Rollback

Jenkins triggers rollback automatically if:

```text
Health endpoint fails
OR
Container startup fails
```

---

# 6. Configuration Management

## Environment Specific Configurations

Separate Spring Boot config files:

```text
application-qa.yml
application-staging.yml
application-prod.yml
```

---

## Active Profile Selection

```bash
SPRING_PROFILES_ACTIVE=qa
```

or

```bash
SPRING_PROFILES_ACTIVE=prod
```

---

# 7. Secrets Handling

Sensitive values:

- MongoDB credentials
- API keys
- JWT secrets

---

## Secure Secret Storage

Secrets stored in:

- Jenkins Credentials Manager
- GCP Secret Manager

---

## Access Method

Injected during pipeline runtime.

Example:

```groovy
withCredentials([
  string(credentialsId: 'mongo-password', variable: 'MONGO_PASS')
])
```

---

## Security Best Practices

- Never store secrets in GitHub
- Use masked Jenkins credentials
- Rotate credentials periodically

---

# 8. Deployment Strategy

## Selected Strategy: Rolling Deployment

---

## Why Rolling Deployment?

Advantages:

- Minimal downtime
- Lower infrastructure cost
- Safer than recreate
- Easier implementation on GCP VM

---

## Comparison

| Strategy | Pros | Cons |
|---|---|---|
| Recreate | Simple | Downtime |
| Blue/Green | Zero downtime | Expensive |
| Rolling | Minimal downtime | Slightly complex |

---

# 9. Zero/Minimal Downtime Approach

Implementation:

- Multiple application instances
- Load balancer routes traffic
- Deploy instances gradually
- Health checks before traffic switch

---