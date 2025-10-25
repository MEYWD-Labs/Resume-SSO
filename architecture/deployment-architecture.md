# Deployment Architecture

## CI/CD Pipeline

```
┌─────────────────────────────────────────────────────────────┐
│                  Deployment Pipeline                         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│ Development → Staging → Production                          │
│                                                              │
│ Per Repository Pipeline:                                    │
│ ────────────────────────                                   │
│                                                              │
│ 1. Code Push to GitHub                                     │
│    └─► GitHub Actions Triggered                            │
│                                                             │
│ 2. CI Pipeline                                             │
│    ├─ Lint & Format Check                                  │
│    ├─ Unit Tests                                           │
│    ├─ Integration Tests                                    │
│    ├─ Security Scan (Snyk)                                 │
│    └─ Build Artifacts                                      │
│                                                             │
│ 3. Deploy to Staging                                       │
│    ├─ Wrangler Deploy (Workers)                            │
│    ├─ Pages Deploy (Frontend)                              │
│    ├─ D1 Migrations                                        │
│    └─ Smoke Tests                                          │
│                                                             │
│ 4. Production Deployment                                   │
│    ├─ Manual Approval Required                             │
│    ├─ Blue-Green Deployment                                │
│    ├─ Health Checks                                        │
│    ├─ Rollback on Failure                                  │
│    └─ Post-Deploy Monitoring                               │
│                                                             │
│ Infrastructure as Code:                                    │
│ ───────────────────────                                   │
│ - Terraform for Cloudflare resources                       │
│ - Wrangler.toml for Workers config                         │
│ - GitHub Actions for CI/CD                                 │
│ - Docker for local development                             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```
