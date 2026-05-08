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
