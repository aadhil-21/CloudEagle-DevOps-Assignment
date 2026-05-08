# CloudEagle DevOps Assignment

## Project Structure
- `/docs` - Design documents
- `/jenkins` - Jenkins pipeline definitions  
- `/infrastructure` - Terraform and K8s configs
- `/diagrams` - Architecture diagrams

## Solution Overview
CI/CD pipeline for Spring Boot sync-service with MongoDB on GCP GKE.
- Blue-green deployments with zero downtime
- Multi-environment setup (qa/staging/prod)
- Auto-scaling with HPA
- Monitoring with GCP Operations Suite

# CloudEagle DevOps Assignment

## Solution Overview
Complete CI/CD pipeline and infrastructure design for Spring Boot sync-service on GCP.

**Key Features:**
- ✅ Blue-green deployments with zero downtime
- ✅ Multi-environment CI/CD (qa → staging → production)
- ✅ Automatic rollback on failure (60 seconds)
- ✅ GKE Autopilot with auto-scaling (3-20 pods)
- ✅ MongoDB Atlas with multi-zone replication
- ✅ Comprehensive monitoring with GCP Operations Suite

---

## Part 1: CI/CD Pipeline

### Files
- [Design Document](docs/design-document.md)
- [Jenkins Pipeline](jenkins/Jenkinsfile)

### Pipeline Stages

Checkout → Unit Tests → Build → Security Scan → Push → Deploy → Health Check


---

## Part 2: Infrastructure Design

### Architecture Diagram

```mermaid
graph TB
    User[Internet User]
    LB[Cloud Load Balancer]

    subgraph GCP[Google Cloud Platform]
        subgraph VPC[VPC: 10.0.0.0/16]
            subgraph GKE[GKE Autopilot Cluster]
                Pod1[sync-service Pod 1]
                Pod2[sync-service Pod 2]
                Pod3[sync-service Pod 3]
                HPA[Auto-scaler: 3-20 pods]
            end
        end
    end

    subgraph Atlas[MongoDB Atlas]
        Primary[(Primary Node)]
        Secondary1[(Secondary Node)]
        Secondary2[(Secondary Node)]
    end

    Secrets[GCP Secret Manager]

    User --> LB
    LB --> Pod1
    LB --> Pod2
    LB --> Pod3

    Pod1 --> Primary
    Pod2 --> Primary
    Pod3 --> Primary

    Primary --> Secondary1
    Primary --> Secondary2

    Pod1 -.-> Secrets
    Pod2 -.-> Secrets
    Pod3 -.-> Secrets


1. Compute Platform: GKE Autopilot
Decision: Google Kubernetes Engine (GKE) Autopilot

Why not Compute Engine or Cloud Run?

Requirement	GKE Autopilot	Compute Engine	Cloud Run
Auto-scaling	✅ Built-in	❌ Manual setup	✅ Native
Zero-downtime	✅ Blue-green	⚠️ Complex	✅ Built-in
Cost	$$ (pay per pod)	$ (pay idle)	$$ (per request)


Configuration:

min_replicas: 3
max_replicas: 20
cpu_threshold: 70%
memory_threshold: 80%


2. Database: MongoDB Atlas
Decision: MongoDB Atlas (M30 cluster)

Why Atlas:

Automated backups (point-in-time recovery)

Multi-zone failover (99.99% SLA)

VPC Peering for private connectivity

Configuration:

Tier: M30 (4 vCPU, 8GB RAM per node)
Nodes: 3 across us-central1 zones
Storage: 100GB SSD
Backup: Continuous (7-day) + Daily snapshots (30-day)


3. Networking
VPC Design:

cloud-eagle-vpc (10.0.0.0/16)
├── subnet-qa (10.0.0.0/20)
├── subnet-staging (10.0.16.0/20)
└── subnet-prod (10.0.32.0/20)

Traffic Flow:

Internet → Cloud Load Balancer (443) → Istio Ingress Gateway → sync-service pods (8080) → MongoDB Atlas (VPC Peering)


4. Security & Secrets
IAM Roles:

Service Account	Permissions
sync-service-prod	Secret Accessor, Logs Writer
jenkins-deploy	Kubernetes Engine Admin
monitoring-viewer	Monitoring Viewer

Secrets Management:

# Secrets stored per environment
gcloud secrets create prod-mongodb-uri
gcloud secrets create prod-api-key

# Injected into pods at runtime
kubectl create secret generic app-secrets --from-literal=mongodb-uri=$(gcloud secrets versions access latest --secret=prod-mongodb-uri)


5. Monitoring & Logging
Stack: GCP Operations Suite

Component	Tool	Retention
Logs	Cloud Logging	30 days
Metrics	Cloud Monitoring	6 weeks
Key Metrics to Alert:

Error rate > 5% for 2 minutes → PagerDuty

Response time > 2 seconds → Slack

CPU/Memory > 80% → Slack


6. Cost Breakdown
Monthly Estimate: $1,160

Service	Cost
GKE Autopilot (5 pods avg)	$300
MongoDB Atlas M30	$700
Load Balancer	$20
Cloud NAT	$40
Logging (100GB)	$100
Total	$1,160


7. Disaster Recovery
RTO: 15 minutes

RPO: 1 hour

Multi-region: us-central1 → us-east1

Backup testing: Weekly automated restore


Submission Info

Repository:  https://github.com/aadhil-21/CloudEagle-DevOps-Assignment.git
Date: 09/05/2026


