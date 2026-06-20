# Deployment Guide

## Local Development

### 1. Start Infrastructure
```bash
docker compose up -d
# Starts PostgreSQL, Redis, NATS
```

### 2. Configure Environment
```bash
cp .env.example .env
# Edit .env with your settings
```

### 3. Run Migrations
```bash
make migrate-up
```

### 4. Build & Run
```bash
make build
make run-all
```

### 5. Verify
```bash
curl http://localhost:8080/healthz
curl http://localhost:8080/metrics
```

---

## Production Deployment (AWS)

### Architecture
```
                    ┌─────────────┐
                    │ Cloudflare  │
                    │  CDN + WAF  │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │     ALB     │
                    └──────┬──────┘
                           │
              ┌────────────┼────────────┐
              │            │            │
        ┌─────▼────┐ ┌────▼─────┐ ┌────▼─────┐
        │ Gateway  │ │  Wallet  │ │  Engine  │
        │ (ECS)    │ │  (ECS)   │ │  (ECS)   │
        └─────┬────┘ └────┬─────┘ └────┬─────┘
              │            │            │
        ┌─────▼────────────▼────────────▼─────┐
        │            NATS Cluster              │
        │         (EC2 or managed)             │
        └─────┬────────────┬────────────┬─────┘
              │            │            │
        ┌─────▼────┐ ┌────▼─────┐ ┌────▼─────┐
        │   RDS    │ │ElastiCache│ │Settlement│
        │ Postgres │ │  Redis    │ │  (ECS)   │
        └──────────┘ └──────────┘ └──────────┘
```

### Prerequisites
- AWS CLI configured
- Terraform >= 1.5
- Docker
- Domain configured in Route 53

### Step 1: Create ECR Repositories
```bash
aws ecr create-repository --repository-name betting-platform/gateway
aws ecr create-repository --repository-name betting-platform/wallet
aws ecr create-repository --repository-name betting-platform/engine
aws ecr create-repository --repository-name betting-platform/settlement
aws ecr create-repository --repository-name betting-platform/games
```

### Step 2: Build & Push Images
```bash
# Login to ECR
aws ecr get-login-password | docker login --username AWS --password-stdin <account>.dkr.ecr.<region>.amazonaws.com

# Build and push
make docker-build

docker tag betting-platform/gateway:latest <account>.dkr.ecr.<region>.amazonaws.com/betting-platform/gateway:latest
docker push <account>.dkr.ecr.<region>.amazonaws.com/betting-platform/gateway:latest

# Repeat for all services...
```

### Step 3: Deploy Infrastructure
```bash
cd deployments/ke-prod

# Initialize Terraform
terraform init

# Plan
terraform plan -var="environment=production" -var="region=af-south-1"

# Apply
terraform apply -var="environment=production" -var="region=af-south-1"
```

### Step 4: Configure Secrets
```bash
# Store secrets in AWS Secrets Manager
aws secretsmanager create-secret \
  --name betting-platform/jwt-secret \
  --secret-string "$(openssl rand -hex 32)"

aws secretsmanager create-secret \
  --name betting-platform/mpesa-credentials \
  --secret-string '{"consumer_key":"...","consumer_secret":"..."}'
```

### Step 5: Run Migrations
```bash
# Run migration task in ECS
aws ecs run-task \
  --cluster betting-platform \
  --task-definition betting-platform-migrate \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-xxx],securityGroups=[sg-xxx]}"
```

### Step 6: Verify Deployment
```bash
# Check service health
curl https://api.yourdomain.com/healthz

# Check metrics
curl https://api.yourdomain.com/metrics
```

---

## Environment Variables (Production)

```bash
# Service
SERVICE_NAME=gateway
ENVIRONMENT=production
PORT=8080

# Database (RDS)
DATABASE_HOST=betting-platform.xxx.af-south-1.rds.amazonaws.com
DATABASE_PORT=5432
DATABASE_NAME=betting_db
DATABASE_SSL_MODE=require

# Redis (ElastiCache)
REDIS_HOST=betting-platform.xxx.cache.amazonaws.com
REDIS_PORT=6379

# NATS
NATS_URL=nats://nats-cluster:4222

# Security
JWT_SECRET=<from-aws-secrets-manager>
BCRYPT_COST=12

# Country
COUNTRY_CODE=KE
CURRENCY=KES

# Betting
MIN_BET_STAKE=10
MAX_BET_STAKE=100000
CRASH_HOUSE_EDGE=0.01

# Tax
TAX_GGR_RATE=0.15
TAX_WHT_RATE=0.20

# Payments (from Secrets Manager)
MPESA_CONSUMER_KEY=xxx
MPESA_CONSUMER_SECRET=xxx
MPESA_SHORTCODE=174379

# KYC
SMILE_ID_API_KEY=xxx
SMILE_ID_PARTNER_ID=xxx

# Logging
LOG_LEVEL=info
LOG_FORMAT=json
```

---

## Monitoring

### Prometheus + Grafana
```bash
# Access Grafana
kubectl port-forward svc/grafana 3000:3000

# Default dashboards:
# - Service Overview (request rate, latency, errors)
# - Wallet Operations (deposits, withdrawals, balance)
# - Betting Activity (bets placed, settled, odds)
# - Crash Game (rounds, multipliers, house edge)
# - Infrastructure (CPU, memory, connections)
```

### Alerts
| Alert | Threshold | Channel |
|-------|-----------|---------|
| High Error Rate | > 5% for 5min | PagerDuty + Slack |
| High Latency | p99 > 500ms for 5min | Slack |
| Database Connections | > 80% | PagerDuty |
| Wallet Service Down | 0 pods for 1min | PagerDuty + SMS |
| M-Pesa Callback Failures | > 10% for 10min | Slack |

---

## Scaling

### Horizontal Scaling
```bash
# Scale gateway service
aws ecs update-service \
  --cluster betting-platform \
  --service gateway \
  --desired-count 5

# Auto-scaling policy
aws application-autoscaling register-scalable-target \
  --service-namespace ecs \
  --resource-id service/betting-platform/gateway \
  --scalable-dimension ecs:service:DesiredCount \
  --min-count 2 \
  --max-count 20
```

### Database Scaling
- Enable Multi-AZ for RDS
- Add read replicas for analytics
- Use connection pooling (PgBouncer)
- Partition large tables by date

---

## Backup & Recovery

### Database Backups
```bash
# Automated RDS snapshots (daily)
# Retention: 30 days

# Manual snapshot
aws rds create-db-snapshot \
  --db-instance-identifier betting-platform \
  --db-snapshot-identifier betting-platform-$(date +%Y%m%d)
```

### Disaster Recovery
- RPO: 1 hour (automated backups)
- RTO: 30 minutes (restore from snapshot)
- Cross-region replication for critical data

---

## Security Checklist

- [ ] All default passwords changed
- [ ] JWT_SECRET is strong random (32+ chars)
- [ ] Database SSL enabled
- [ ] Redis AUTH enabled
- [ ] NATS TLS enabled
- [ ] Rate limiting configured
- [ ] WAF rules active (Cloudflare)
- [ ] Security headers (HSTS, CSP, X-Frame-Options)
- [ ] Secrets in AWS Secrets Manager (not env vars)
- [ ] VPC with private subnets for services
- [ ] Security groups restrict traffic
- [ ] CloudTrail enabled for audit
- [ ] GuardDuty enabled for threat detection
