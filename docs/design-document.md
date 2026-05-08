# CI/CD Pipeline Design - sync-service

## 1. Branching Strategy

### Branch Model

main (production)
├── staging
├── qa
└── feature/*


### Environment Mapping

| Branch | Environment | Auto-deploy | Protection |
|--------|------------|-------------|------------|
| feature/* | None | No | PR required |
| qa | QA | Yes, on push | None |
| staging | Staging | Yes, on push | PR approval |
| main | Production | Manual only | 2 approvals + status checks |

### Preventing Accidental Production Deployments
- Protected branches: main requires PR with 2 approvals
- Deployment gate: Manual approval step in Jenkins
- Separate GCP projects per environment
- No direct commits to main

## 2. Jenkins Pipeline Stages

### Pipeline Flow

Checkout → Unit Tests → Build → Security Scan → Push to Registry → Deploy → Health Check


### PR vs Merge Behavior

| Event | Actions |
|-------|---------|
| Pull Request Created | Run unit tests, static analysis (NO deployment) |
| Merge to qa | Full pipeline + auto-deploy to QA |
| Merge to staging | Full pipeline + auto-deploy to staging |
| Merge to main | Full pipeline + MANUAL approval + deploy to prod |

## 3. Rollback Strategy

### Automatic Rollback Triggers
- Health check fails (3 consecutive failures)
- Error rate exceeds 5% in 1 minute
- Response time > 2 seconds for 5 minutes

### Rollback Process
```bash
# 1. Detect failure in new version
# 2. Switch traffic back to previous version
kubectl rollout undo deployment/sync-service

# 3. Verify rollback
curl -f https://api.cloudeagle.com/health

# Rollback complete within 60 seconds

Rollback Timeline

0-30s: Detect failure

30-60s: Automatic rollback initiated

60-90s: Traffic switched back

90-120s: Verification complete


Configuration Management

Environment-Specific Configs

config/
├── application-qa.yml
├── application-staging.yml
└── application-prod.yml

Secrets Management (GCP Secret Manager)
Store secrets per environment:

gcloud secrets create prod-mongodb-uri
gcloud secrets create prod-api-key


Inject into Kubernetes pods:

apiVersion: v1
kind: Pod
spec:
  containers:
  - env:
    - name: MONGODB_URI
      valueFrom:
        secretKeyRef:
          name: mongodb-secret
          key: uri


Deployment Strategy: Blue-Green

Why Blue-Green Over Others?

Strategy	Downtime	Rollback Speed	Complexity
Blue-Green	Zero	Instant (seconds)	Medium
Rolling Update	Zero	Slow (minutes)	Low
Recreate	30-60s	Slow	Low


Blue-Green chosen because:

✅ Zero downtime during deployment

✅ Instant rollback (just switch traffic)

✅ Can test new version before routing traffic

✅ No database migration conflicts


Zero-Downtime Approach

# Step 1: Deploy green version alongside blue
kubectl apply -f deployment-green.yaml

# Step 2: Wait for readiness
kubectl wait --for=condition=ready pod -l version=green

# Step 3: Run smoke tests on green
kubectl exec green-pod -- ./smoke-tests.sh

# Step 4: Switch traffic (zero downtime)
kubectl patch service sync-service -p '{"spec":{"selector":{"version":"green"}}}'

# Step 5: Keep blue for 30 minutes (rollback window) 
