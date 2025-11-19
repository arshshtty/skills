---
name: devops-cicd
description: Design and implement CI/CD pipelines, containerization, and deployment strategies. Apply when setting up automation, improving deployment processes, or implementing infrastructure as code.
---

# DevOps & CI/CD Skill

Build reliable, automated pipelines that enable fast, safe, and consistent software delivery from development to production.

## Core Principles

### 1. **Automate Everything**
- Manual processes are error-prone
- If you do it twice, automate it
- Treat automation code like production code

### 2. **Fail Fast**
- Catch issues early in the pipeline
- Quick feedback loops
- Stop the line on failure

### 3. **Infrastructure as Code**
- Version control everything
- Reproducible environments
- Self-documenting infrastructure

### 4. **Continuous Improvement**
- Measure everything
- Optimize bottlenecks
- Learn from failures

## CI/CD Pipeline Design

### Pipeline Stages

```
┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐
│  Build  │ → │  Test   │ → │ Security│ → │ Deploy  │ → │ Monitor │
└─────────┘   └─────────┘   └─────────┘   └─────────┘   └─────────┘
```

### Stage 1: Build

**Purpose**: Compile code, install dependencies, create artifacts

**Best Practices**:
```yaml
build:
  steps:
    - Install dependencies
    - Compile/transpile code
    - Generate artifacts
    - Create container images
    - Cache dependencies for speed

  Goals:
    - Fast (< 5 minutes)
    - Deterministic (same input = same output)
    - Produces versioned artifacts
```

**Dependency Management**:
```
- Use lock files (package-lock.json, Pipfile.lock)
- Pin versions for reproducibility
- Cache dependencies between builds
- Regular dependency updates (Dependabot)
```

### Stage 2: Test

**Test Pyramid in CI**:
```
Fast → Slow:

1. Unit Tests (seconds)
   - Run on every commit
   - Fast feedback
   - High coverage

2. Integration Tests (minutes)
   - Run on every PR
   - Test component interactions
   - Use test containers

3. E2E Tests (minutes to hours)
   - Run before deploy
   - Critical paths only
   - May run in parallel
```

**Best Practices**:
```yaml
test:
  unit:
    trigger: every commit
    timeout: 5 minutes
    parallelism: high

  integration:
    trigger: every PR
    timeout: 15 minutes
    requires: test containers

  e2e:
    trigger: before deploy
    timeout: 30 minutes
    parallelism: medium
```

### Stage 3: Security

**Security Scanning**:
```
1. Static Analysis (SAST)
   - Scan source code for vulnerabilities
   - Check for hardcoded secrets
   - Code quality issues

2. Dependency Scanning
   - Check for known CVEs
   - Outdated packages
   - License compliance

3. Container Scanning
   - Base image vulnerabilities
   - Configuration issues
   - Secrets in images

4. Dynamic Analysis (DAST)
   - Run against deployed app
   - Find runtime vulnerabilities
   - API security testing
```

### Stage 4: Deploy

**Deployment Environments**:
```
Development → Staging → Production

Development:
  - Auto-deploy on merge to main
  - Latest code always running
  - Shared or ephemeral

Staging:
  - Production-like environment
  - Pre-production validation
  - Performance testing

Production:
  - Manual approval (or auto with gates)
  - Gradual rollout
  - Monitoring and rollback ready
```

### Stage 5: Monitor

**Post-Deployment Verification**:
```
- Health checks pass
- No error rate increase
- Performance within bounds
- Key metrics stable
- Alerts configured
```

## Deployment Strategies

### Blue-Green Deployment

```
           ┌─────────────┐
Traffic → │ Load Balancer │
           └──────┬──────┘
          ┌───────┴───────┐
          ▼               ▼
    ┌──────────┐    ┌──────────┐
    │  Blue    │    │  Green   │
    │ (Active) │    │  (Idle)  │
    └──────────┘    └──────────┘

Process:
1. Green has new version
2. Test Green
3. Switch traffic Blue → Green
4. Blue becomes idle (rollback target)
```

**Pros**:
- Instant rollback
- Zero downtime
- Full environment testing

**Cons**:
- Double infrastructure cost
- Database migrations tricky
- Session management challenges

### Canary Deployment

```
Traffic → Load Balancer
              │
         ┌────┴────┐
         ▼         ▼
    ┌────────┐ ┌────────┐
    │ Stable │ │ Canary │
    │  95%   │ │   5%   │
    └────────┘ └────────┘

Process:
1. Deploy to small % (canary)
2. Monitor for issues
3. Gradually increase %
4. Full rollout or rollback
```

**Typical Rollout**:
```
1%  → Monitor 10 min → No issues →
5%  → Monitor 30 min → No issues →
25% → Monitor 1 hour → No issues →
50% → Monitor 1 hour → No issues →
100%
```

**Pros**:
- Lower risk
- Real user feedback
- Gradual rollout

**Cons**:
- Longer deployment time
- Multiple versions running
- Need good monitoring

### Rolling Deployment

```
Instance 1: v1 → v2
Instance 2: v1 → (waiting) → v2
Instance 3: v1 → (waiting) → v2

Process:
1. Take instance out of rotation
2. Deploy new version
3. Health check
4. Add back to rotation
5. Repeat for next instance
```

**Pros**:
- No extra infrastructure
- Zero downtime
- Gradual rollout

**Cons**:
- Mixed versions during deploy
- Slower rollback
- Must handle version compatibility

### Feature Flags

```javascript
// Separate deployment from release
if (featureFlags.isEnabled('newCheckout', user)) {
  return newCheckoutFlow(cart);
} else {
  return oldCheckoutFlow(cart);
}
```

**Use Cases**:
- Gradual feature rollout
- A/B testing
- Kill switch for problems
- Trunk-based development

**Best Practices**:
- Clean up old flags
- Don't overuse (complexity)
- Test both paths
- Have flag naming conventions

## Containerization

### Docker Best Practices

**Efficient Dockerfile**:
```dockerfile
# Use specific version (not latest)
FROM node:20-alpine

# Set working directory
WORKDIR /app

# Copy dependency files first (caching)
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Use non-root user
USER node

# Document the port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:3000/health || exit 1

# Use exec form for signals
CMD ["node", "server.js"]
```

**Optimization Tips**:
```
Layer Caching:
- Order from least to most changing
- Dependencies before code
- Use .dockerignore

Image Size:
- Use alpine/slim base images
- Multi-stage builds
- Remove dev dependencies
- Clean up in same layer as install

Security:
- Non-root user
- Specific versions
- Scan for vulnerabilities
- Don't include secrets
```

**Multi-Stage Build**:
```dockerfile
# Build stage
FROM node:20 AS builder
WORKDIR /app
COPY . .
RUN npm ci && npm run build

# Production stage
FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
USER node
CMD ["node", "dist/server.js"]
```

### Container Orchestration

**Kubernetes Key Concepts**:
```
Pod:        Smallest deployable unit (one or more containers)
Deployment: Manages pod replicas and updates
Service:    Stable network endpoint for pods
ConfigMap:  Configuration data
Secret:     Sensitive data
Ingress:    External access routing
```

**Basic Deployment**:
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
        image: myapp:1.0.0
        ports:
        - containerPort: 3000
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
```

## Infrastructure as Code

### IaC Principles

**Benefits**:
```
- Version controlled
- Reproducible
- Self-documenting
- Reviewable
- Testable
```

**Best Practices**:
```
1. Everything in version control
2. Use modules for reusability
3. Separate environments
4. Use remote state
5. Plan before apply
6. Use workspaces or directories per environment
```

### Terraform Example

**Basic Structure**:
```
infrastructure/
├── modules/
│   ├── vpc/
│   ├── database/
│   └── compute/
├── environments/
│   ├── dev/
│   ├── staging/
│   └── prod/
└── main.tf
```

**Sample Configuration**:
```hcl
# Configure provider
provider "aws" {
  region = var.aws_region
}

# Use modules
module "vpc" {
  source = "./modules/vpc"
  cidr   = var.vpc_cidr
}

module "database" {
  source     = "./modules/database"
  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnet_ids
}

# Outputs
output "database_endpoint" {
  value = module.database.endpoint
}
```

### GitOps

**Principles**:
```
1. Declarative: Describe desired state
2. Versioned: Git is source of truth
3. Automated: Apply changes automatically
4. Observable: Verify and alert on drift
```

**Workflow**:
```
1. Developer commits to config repo
2. CI validates changes
3. Merge to main
4. GitOps operator detects change
5. Operator applies to cluster
6. Operator reports status
```

## Pipeline Configuration

### GitHub Actions Example

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run tests
        run: npm test

      - name: Build
        run: npm run build

      - name: Upload artifact
        uses: actions/upload-artifact@v3
        with:
          name: build
          path: dist/

  security:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run security scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

  deploy-staging:
    needs: [build, security]
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - name: Download artifact
        uses: actions/download-artifact@v3
        with:
          name: build

      - name: Deploy to staging
        run: |
          # Deploy commands here

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Download artifact
        uses: actions/download-artifact@v3
        with:
          name: build

      - name: Deploy to production
        run: |
          # Deploy commands with approval
```

### Pipeline Best Practices

**Speed Optimization**:
```
- Cache dependencies
- Parallelize tests
- Use smaller base images
- Skip unnecessary steps on PRs
- Use incremental builds
```

**Reliability**:
```
- Retry flaky steps
- Timeout limits
- Clear error messages
- Artifact versioning
- Idempotent deployments
```

**Security**:
```
- Secrets in vault, not code
- Least privilege for CI
- Sign artifacts
- Audit trail
- Secure runners
```

## Secret Management

### Best Practices

**Never**:
```
- Commit secrets to git
- Log secrets
- Pass secrets via command line
- Use same secrets in all environments
- Share secrets in plain text
```

**Do**:
```
- Use secret managers (Vault, AWS Secrets Manager)
- Rotate secrets regularly
- Use different secrets per environment
- Audit access
- Encrypt at rest and in transit
```

### Secret Rotation

```
1. Create new secret
2. Update application to use new
3. Verify application works
4. Disable old secret
5. Delete old secret after grace period
```

## Monitoring & Observability

### Three Pillars

**Logs**:
```
- Structured logging (JSON)
- Correlation IDs
- Appropriate levels
- Centralized collection
```

**Metrics**:
```
- RED: Rate, Errors, Duration
- USE: Utilization, Saturation, Errors
- Business metrics
- SLI/SLO tracking
```

**Traces**:
```
- Distributed tracing
- Request flow
- Latency breakdown
- Dependency mapping
```

### Key Metrics to Track

**Pipeline Metrics**:
```
- Build frequency
- Build duration
- Build success rate
- Test coverage
- Deployment frequency
- Lead time
- MTTR (Mean Time to Recovery)
- Change failure rate
```

**Application Metrics**:
```
- Request rate
- Error rate
- Latency (p50, p95, p99)
- Saturation
- Availability
```

## Disaster Recovery

### Backup Strategy

**3-2-1 Rule**:
```
3 copies of data
2 different storage types
1 offsite location
```

**Test Restores**:
```
- Schedule regular restore tests
- Document restore procedure
- Measure recovery time
- Verify data integrity
```

### Rollback Procedures

**Automated Rollback**:
```
if (deploymentFailed || metricsUnhealthy) {
  rollback to previous version
  alert team
  create incident
}
```

**Manual Rollback Checklist**:
```
1. Identify the issue
2. Decide: rollback vs forward fix
3. Notify stakeholders
4. Execute rollback
5. Verify system health
6. Post-mortem
```

## DevOps Checklist

### Pipeline Essentials
- [ ] Version control for all code and config
- [ ] Automated builds on commit
- [ ] Automated tests in pipeline
- [ ] Security scanning
- [ ] Artifact versioning
- [ ] Environment parity

### Deployment
- [ ] Zero-downtime deployments
- [ ] Rollback capability
- [ ] Feature flags for releases
- [ ] Staged rollouts
- [ ] Health checks
- [ ] Post-deploy verification

### Infrastructure
- [ ] Infrastructure as Code
- [ ] Immutable infrastructure
- [ ] Auto-scaling configured
- [ ] Disaster recovery plan
- [ ] Backup and restore tested

### Observability
- [ ] Centralized logging
- [ ] Metrics and dashboards
- [ ] Alerting configured
- [ ] Runbooks for incidents
- [ ] On-call rotation

### Security
- [ ] Secrets in secure storage
- [ ] Least privilege access
- [ ] Audit logging
- [ ] Regular secret rotation
- [ ] Vulnerability scanning

## Anti-Patterns

**Snowflake Servers**
- Each server manually configured
- Fix: Infrastructure as Code

**Long-Lived Feature Branches**
- Merge conflicts, integration pain
- Fix: Trunk-based development

**Manual Deployments**
- Error-prone, slow
- Fix: Automated pipelines

**Testing in Production**
- Customers find bugs
- Fix: Proper test environments

**Ignoring Failed Builds**
- Broken window syndrome
- Fix: Fix immediately or revert

---

**Remember**: DevOps is about culture as much as tools. Aim for fast feedback, continuous improvement, and shared responsibility between development and operations.
