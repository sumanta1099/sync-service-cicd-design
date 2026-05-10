# sync-service CI/CD Architecture

## Project Overview

sync-service is a Spring Boot backend application integrated with MongoDB and deployed on Google Cloud Platform using Cloud Run.

The delivery pipeline is managed through Jenkins to automate build, testing, deployment, and rollback activities across multiple environments.

### Deployment Environments

| Environment | Purpose |
|---|---|
| QA | Internal integration testing |
| Staging | Pre-production validation |
| Production | Live customer traffic |

---

# CI/CD Objectives

The pipeline is designed to achieve the following:

- Continuous integration and automated delivery
- Reliable and controlled production deployments
- Automated rollback during failures
- Secure configuration and secret handling
- Minimal service interruption during releases

---

# Git Branching Model

The repository follows an environment-based branching workflow.

| Branch | Usage | Deployment Target |
|---|---|---|
| feature/* | Individual feature development | PR validation only |
| development | QA testing | QA environment |
| staging | UAT and release testing | Staging environment |
| prod | Production-ready code | Production environment |

---

## Development Flow

```text
feature/* → development → staging → prod
```

---

# Deployment Control Measures

To reduce deployment risks in production, the following controls are implemented.

## Protected Production Branch

- Direct commits to `prod` are restricted
- Changes require pull request approval
- Mandatory code review before merge

---

## Manual Production Approval

Production deployments require manual confirmation from authorized users through Jenkins.

```groovy
input message: "Approve Production Deployment?"
```

---

## Branch-Based Deployment Restriction

Production deployment executes only when code is merged into the `prod` branch.

---

# Jenkins Pipeline Workflow

The CI/CD workflow follows the sequence below.

```text
Source Checkout
      ↓
Application Build
      ↓
Unit Testing
      ↓
Code Validation
      ↓
Docker Image Build
      ↓
Artifact Push
      ↓
Cloud Run Deployment
      ↓
Application Health Validation
      ↓
Automatic Rollback (If Required)
```

---

# Pull Request Validation

Whenever a pull request is opened, Jenkins performs validation checks before merge approval.

### Validation Steps

- Fetch source code
- Compile application
- Execute unit tests
- Run code quality analysis
- Build Docker image for verification

No deployment is triggered during pull request validation.

### Purpose

- Detect issues early
- Maintain branch stability
- Prevent faulty code merges

---

# Environment Deployment Flow

## development Branch

Deployment automatically targets the QA environment.

---

## staging Branch

Deployment automatically targets the staging environment for release verification.

---

## prod Branch

Production deployment requires:

- Successful pipeline execution
- Manual approval step
- Authorized deployment access

---

# Rollback Design

Rollback handling is implemented using Cloud Run revision management.

If deployment issues occur, traffic is redirected to the last stable revision automatically.

### Rollback Trigger Conditions

- Failed deployment
- Failed health endpoint validation
- Application startup issues
- Runtime container failure

---

# Cloud Run Revision Management

Every deployment creates a separate Cloud Run revision.

Example:

```text
sync-service-00012 → Newly deployed version
sync-service-00011 → Previous stable version
```

---

# Rollback Execution Flow

```text
Deploy New Revision
        ↓
Execute Health Validation
        ↓
Validation Failed
        ↓
Identify Previous Stable Revision
        ↓
Redirect Traffic
        ↓
Restore Stable Application
```

---

# Rollback Implementation

## Retrieve Previous Revision

```bash
gcloud run revisions list \
  --service=${SERVICE_NAME} \
  --region=${REGION}
```

---

## Select Stable Revision

```bash
sed -n '2p'
```

This command selects the previous stable revision from the revision list.

---

## Redirect Application Traffic

```bash
gcloud run services update-traffic ${SERVICE_NAME} \
  --region=${REGION} \
  --to-revisions=$PREVIOUS_REVISION=100
```

### Traffic Result

```text
100% → Stable Revision
0%   → Failed Revision
```

---

# Automated Rollback Handling

Rollback logic executes automatically from the Jenkins `post` failure block.

```groovy
post {
    failure {
        // rollback logic
    }
}
```

---

# Configuration Management

Separate Spring Boot configuration files are maintained for each environment.

```text
application-qa.yml
application-staging.yml
application-prod.yml
```

---

# Runtime Profile Selection

Environment-specific profiles are injected during deployment.

Example:

```bash
SPRING_PROFILES_ACTIVE=qa
```

or

```bash
SPRING_PROFILES_ACTIVE=prod
```

---

# Secret Management

Sensitive values are never stored inside source code repositories.

### Managed Secrets

- MongoDB credentials
- JWT secrets
- API tokens
- External service credentials

---

# Secret Storage Platforms

Secrets are maintained securely using:

- Jenkins Credentials Manager
- Google Cloud Secret Manager

---

# Runtime Secret Injection

```groovy
withCredentials([
  string(credentialsId: 'mongo-password', variable: 'MONGO_PASS')
])
```

---

# Security Practices

- Secrets excluded from Git repositories
- Credential masking enabled in Jenkins
- IAM-based access restriction
- Periodic credential rotation

---

# Deployment Strategy

The deployment model follows a rolling deployment approach.

## Benefits

- Reduced downtime
- Safer production rollout
- Controlled instance replacement
- Simplified recovery process

---

# Deployment Strategy Comparison

| Strategy | Advantage | Limitation |
|---|---|---|
| Recreate | Easy implementation | Full downtime |
| Blue/Green | Near-zero downtime | Higher infrastructure cost |
| Rolling | Minimal downtime | More deployment coordination |

---

# High Availability Approach

To minimize downtime during releases:

- Multiple service instances are maintained
- Traffic routing is controlled through load balancing
- Health validation occurs before traffic switch
- Failed revisions are automatically isolated

---

# Summary

The CI/CD implementation for `sync-service` provides:

- Automated software delivery
- Controlled release management
- Secure deployment handling
- Cloud Run revision-based rollback
- Environment-specific configuration management
- Reliable production deployment workflow

This architecture improves operational stability while supporting continuous delivery practices.