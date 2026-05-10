# Infrastructure Design for sync-service on GCP

# 1. Overview

The `sync-service` of a Spring Boot backend application connected to MongoDB.

The infrastructure is designed with the following goals:

- Auto-scaling
- Secure access
- High availability
- Startup-friendly cost optimization

---

# 2. Compute Choice

## Selected Compute Option: Cloud Run

---

## Why Cloud Run?

Cloud Run is chosen because:

| Feature | Benefit |
|---|---|
| Serverless | No VM management |
| Auto-scaling | Scales automatically based on traffic |
| Cost-effective | Pay only when requests come |
| Easy deployment | Simple container deployment |
| HTTPS by default | Built-in secure access |
| Managed infrastructure | Lower DevOps overhead |

---

## Why Not GKE?

| GKE Limitation | Reason |
|---|---|
| Higher operational complexity | Requires Kubernetes management |
| More expensive | Cluster nodes always running |
| Not startup friendly | More infrastructure overhead |

---

## Why Not Compute Engine?

| Compute Engine Limitation | Reason |
|---|---|
| Manual scaling | Requires autoscaling configuration |
| VM maintenance | OS patching & management needed |
| Lower efficiency | Idle VM cost |

---

# 3. Proposed Architecture

```text
Users
   │
   ▼
Cloud Load Balancer(public subnet)
   │
   ▼
Cloud Run (sync-service containers)
   │
   ▼
MongoDB Atlas(private subnet)
   │
   ▼
Cloud Logging + Cloud Monitoring
```

---

# 4. MongoDB Hosting Approach

## Selected Option: MongoDB Atlas

---

## Why MongoDB Atlas?

Advantages:

- Fully managed MongoDB
- Automated backups
- Auto-scaling
- High availability
- Easy integration with GCP
- Reduced operational effort

---

## Alternative Option

Self-hosted MongoDB on Compute Engine.

Not selected because:

- Higher maintenance
- Backup responsibility
- Scaling complexity
- Security management overhead

---

# 5. Networking Design

## VPC Setup

A dedicated VPC will be created.

Components:

- Public subnet
- Private subnet
- Firewall rules

---

## Traffic Flow

```text
Internet
   │
HTTPS
   ▼
Load Balancer
   │
   ▼
Cloud Run Service
   │
Private Access
   ▼
MongoDB Atlas
```

---

## Security Controls

### Firewall Rules

Allow:

- HTTPS (443)
- Internal communication only

Deny:

- Unauthorized inbound traffic

---

## Ingress Configuration

Cloud Load Balancer handles:

- SSL termination
- HTTPS routing
- Traffic balancing

---

# 6. Secrets & IAM

## Secrets Management

Sensitive data stored in:

- GCP Secret Manager

Secrets include:

- MongoDB credentials
- API keys
- JWT secrets

---

## Access Control

Cloud Run service account receives:

- Least privilege access
- Secret Manager read-only permissions

---

## IAM Best Practices

- Separate service accounts
- Role-based access control
- No hardcoded credentials

---

# 7. Logging & Monitoring

## Logging Stack

### GCP Cloud Logging

Collects:

- Application logs
- Container logs
- Request logs

---

## Monitoring Stack

### GCP Cloud Monitoring

Tracks:

- CPU usage
- Memory usage
- Request count
- Error rates
- Response latency

---

## Alerting

Alerts configured for:

- High CPU
- Application downtime
- High error rate
- Failed deployments

---

<!-- # 8. Auto-Scaling Design

Cloud Run automatically scales based on:

- HTTP traffic
- Concurrent requests
- CPU utilization

---

## Scaling Benefits

- Handles traffic spikes automatically
- Reduces idle cost
- Improves application availability

---

# 9. High Availability

High availability achieved through:

- Managed Cloud Run platform
- Multiple container instances
- Managed MongoDB Atlas replication

---

# 10. Cost Optimization

Startup-friendly cost optimization techniques:

| Optimization | Benefit |
|---|---|
| Cloud Run | Pay-per-use |
| MongoDB Atlas shared tier | Lower DB cost |
| Autoscaling | No idle server cost |
| Managed services | Reduced operational effort |

---

# 11. Security Best Practices

Implemented security measures:

- HTTPS everywhere
- Secret Manager integration
- IAM least privilege
- Private DB access
- Firewall restrictions

---

# 12. Conclusion

This infrastructure design provides:

- Secure deployment
- Automatic scaling
- Low operational overhead
- Startup-friendly cost
- High availability
- Managed monitoring and logging -->