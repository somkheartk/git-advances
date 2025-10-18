# Team Workflows - Workflows สำหรับทีมขนาดต่างๆ

## Small Team (3-5 คน) - GitHub Flow

### ข้อดี
- Simple และเข้าใจง่าย
- เหมาะกับ Continuous Deployment
- Review process รวดเร็ว

### Workflow

```bash
# 1. เริ่มจาก main branch
git checkout main
git pull origin main

# 2. สร้าง feature branch
git checkout -b feature/add-user-settings

# 3. ทำงานและ commit เป็นระยะ
git add .
git commit -m "feat: add user settings page"
git push origin feature/add-user-settings

# 4. สร้าง Pull Request
gh pr create --title "Add user settings page" --body "..."

# 5. ทีมอื่น review
# - Comment ถ้ามีคำถาม
# - Request changes ถ้าต้องแก้
# - Approve ถ้าโอเค

# 6. หลัง approve merge เข้า main
gh pr merge --squash

# 7. Deploy to production (automatic via CI/CD)
```

### Best Practices

```bash
# Branch naming
feature/short-description
fix/bug-description
docs/update-readme

# Commit messages
git commit -m "feat: add new feature"
git commit -m "fix: resolve bug in login"
git commit -m "docs: update API documentation"

# Delete branches หลัง merge
git branch -d feature/add-user-settings
git push origin --delete feature/add-user-settings

# Clean up local branches
git fetch --prune
git branch -vv | grep ': gone]' | awk '{print $1}' | xargs git branch -D
```

---

## Medium Team (10-20 คน) - Git Flow

### ข้อดี
- Structured และมีระเบียบ
- แยก development และ production ชัดเจน
- รองรับ release schedule

### Branch Structure

```
main (production)
  └── develop (integration)
        ├── feature/* (features)
        ├── release/* (release prep)
        └── hotfix/* (urgent fixes)
```

### Workflow สำหรับ Features

```bash
# Feature Team Lead
# ประสานงานว่าใครทำอะไร
Team A: feature/payment-integration
Team B: feature/user-dashboard
Team C: feature/reporting

# Developer workflow
git checkout develop
git pull origin develop
git checkout -b feature/JIRA-123-payment-stripe

# Daily sync
git checkout develop
git pull origin develop
git checkout feature/JIRA-123-payment-stripe
git merge develop  # หรือ rebase develop

# สำเร็จแล้ว
git push origin feature/JIRA-123-payment-stripe
# สร้าง PR to develop
# หลัง merge ลบ branch
```

### Workflow สำหรับ Release

```bash
# Release Manager
# 2 สัปดาห์ก่อน release date

# สร้าง release branch
git checkout develop
git pull origin develop
git checkout -b release/v2.0.0
git push -u origin release/v2.0.0

# Announce to team
# "Code freeze for v2.0.0"
# "Only bug fixes allowed"

# QA testing
# พบ bugs

# Developer fix bugs
git checkout release/v2.0.0
git checkout -b bugfix/fix-payment-error
# ... fix ...
git checkout release/v2.0.0
git merge bugfix/fix-payment-error

# Release day
git checkout main
git merge --no-ff release/v2.0.0
git tag -a v2.0.0 -m "Release v2.0.0"
git push origin main --tags

# Merge back to develop
git checkout develop
git merge --no-ff release/v2.0.0
git push origin develop

# Delete release branch
git push origin --delete release/v2.0.0
```

### Workflow สำหรับ Hotfix

```bash
# Production bug! 🔥
# On-call engineer

# สร้าง hotfix branch จาก main
git checkout main
git pull origin main
git checkout -b hotfix/critical-sql-injection

# Fix the bug
# ... coding ...
git commit -m "fix: patch SQL injection vulnerability"

# Fast-track review
# Security team reviews
# QA tests in hotfix branch

# Merge to main
git checkout main
git merge --no-ff hotfix/critical-sql-injection
git tag -a v2.0.1 -m "Hotfix v2.0.1 - Security patch"
git push origin main --tags

# Deploy immediately
./deploy-production.sh

# Merge back to develop
git checkout develop
git merge --no-ff hotfix/critical-sql-injection
git push origin develop

# Notify team
```

---

## Large Team (50+ คน) - Monorepo + Microservices

### Organization Structure

```
company-monorepo/
├── services/
│   ├── user-service/      (Team A - 5 people)
│   ├── payment-service/   (Team B - 5 people)
│   ├── order-service/     (Team C - 5 people)
│   └── notification-service/ (Team D - 5 people)
├── packages/
│   ├── shared-ui/         (Frontend Platform - 8 people)
│   ├── shared-utils/      (Infrastructure - 3 people)
│   └── design-system/     (Design - 4 people)
├── apps/
│   ├── web-app/           (Web Team - 10 people)
│   └── mobile-app/        (Mobile Team - 8 people)
└── .github/
    ├── CODEOWNERS
    └── workflows/
```

### CODEOWNERS

```
# .github/CODEOWNERS

# Default owners for everything
* @tech-leads @architects

# Service teams
/services/user-service/ @team-users @tech-lead-alice
/services/payment-service/ @team-payments @tech-lead-bob
/services/order-service/ @team-orders @tech-lead-charlie

# Platform teams
/packages/shared-ui/ @team-frontend-platform @tech-lead-diana
/packages/design-system/ @team-design @design-lead-eve

# Application teams
/apps/web-app/ @team-web @tech-lead-frank
/apps/mobile-app/ @team-mobile @tech-lead-grace

# Infrastructure and configuration
/.github/ @team-devops @infra-lead
/docker/ @team-devops @infra-lead
/kubernetes/ @team-devops @infra-lead

# Documentation
/docs/ @tech-writers @everyone
```

### Workflow แต่ละ Team

```bash
# Team A: User Service
git checkout main
git pull origin main
git checkout -b feature/users/add-2fa

# ทำงานเฉพาะใน services/user-service/
# CI runs tests เฉพาะ service นี้

git add services/user-service/
git commit -m "feat(users): add two-factor authentication"
git push origin feature/users/add-2fa

# สร้าง PR
# Review by @team-users และ @tech-lead-alice
# Approval ต้อง 2 คน จาก team

# Merge และ deploy เฉพาะ user-service
```

### CI/CD Configuration

```yaml
# .github/workflows/ci.yml
name: Monorepo CI

on:
  pull_request:
  push:
    branches: [main, develop]

jobs:
  detect-changes:
    runs-on: ubuntu-latest
    outputs:
      user-service: ${{ steps.changes.outputs.user-service }}
      payment-service: ${{ steps.changes.outputs.payment-service }}
      web-app: ${{ steps.changes.outputs.web-app }}
    steps:
      - uses: actions/checkout@v2
      - uses: dorny/paths-filter@v2
        id: changes
        with:
          filters: |
            user-service:
              - 'services/user-service/**'
            payment-service:
              - 'services/payment-service/**'
            web-app:
              - 'apps/web-app/**'

  test-user-service:
    needs: detect-changes
    if: needs.detect-changes.outputs.user-service == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Test User Service
        run: |
          cd services/user-service
          npm test

  test-payment-service:
    needs: detect-changes
    if: needs.detect-changes.outputs.payment-service == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Test Payment Service
        run: |
          cd services/payment-service
          npm test
```

### Cross-team Dependencies

```bash
# Scenario: Payment Service ต้องการใช้ User Service API

# Option 1: Feature Branches Coordination
# Team Users สร้าง API endpoint ใหม่
git checkout -b feature/users/add-payment-webhook-api

# Team Payments รอ API พร้อม
# ระหว่างนี้ใช้ mock

# เมื่อ User Service merge แล้ว
# Payment Service สามารถ integrate ได้

# Option 2: Feature Flags
# User Service deploy API ใหม่ (feature flag off)
# Payment Service integrate (feature flag off)
# เมื่อทั้งสอง ready เปิด feature flag
```

### Release Strategy

```bash
# Release Train (ทุก 2 สัปดาห์)

# Week 1-2: Development
# ทุกทีมทำ features ใน feature branches

# Friday Week 2: Code Freeze
# สร้าง release branch
git checkout main
git pull origin main
git checkout -b release/2024.02.15
git push -u origin release/2024.02.15

# Week 3: Testing และ Bug Fixes
# QA test ทั้ง monorepo
# Bugs ถูก fix ใน release branch

# Friday Week 3: Release
git checkout main
git merge --no-ff release/2024.02.15
git tag -a v2024.02.15 -m "Release 2024.02.15"
git push origin main --tags

# Incremental deployment
# Deploy services ทีละตัว
./deploy.sh user-service
./deploy.sh payment-service
# ... etc

# Monitor แต่ละ service
# Rollback ถ้ามีปัญหา
```

---

## Remote Team (Distributed Globally)

### Time Zone Challenges

```bash
# Team Members:
# - Alice (UTC+0) London
# - Bob (UTC-8) San Francisco  
# - Charlie (UTC+9) Tokyo
# - Diana (UTC+5:30) India

# Strategy: Async Communication
```

### Async Workflow

```bash
# Alice (London - 9 AM)
git checkout -b feature/new-api
# ... work 4 hours ...
git commit -m "feat: add new API endpoint (WIP)"
git push origin feature/new-api

# Create PR with detailed description
gh pr create --title "Add new API endpoint" \
  --body "## Changes
  - Added new endpoint /api/users
  - Added tests
  - Updated documentation
  
  ## Questions for reviewers
  - Is the error handling sufficient?
  - Should we add rate limiting?
  
  ## Testing
  - All tests passing
  - Manual testing done
  
  @bob @charlie please review when you're online"

# Bob (San Francisco - 1 AM -> goes to bed)
# Wakes up at 8 AM PST (4 PM London)

# Bob reviews
gh pr review 123 --comment --body "
Looks good overall! Few comments:

1. Line 45: Consider adding input validation
2. Line 67: This error message could be more specific
3. Tests look good

@alice please address when you're online
"

# Charlie (Tokyo - next day 9 AM)
# Wakes up (midnight London, 4 PM San Francisco previous day)

# Charlie reviews
gh pr review 123 --approve --body "
Code looks good!
Tests are comprehensive.
Documentation is clear.

LGTM! 🚀
"

# Alice (next day 9 AM London)
# Addresses feedback
git checkout feature/new-api
# ... make changes ...
git commit -m "fix: address review comments"
git push origin feature/new-api

# Auto-merge after approvals
gh pr merge 123 --squash
```

### Best Practices for Remote Teams

```bash
# 1. Over-communicate in PR descriptions
# ไม่มีโอกาสถามตอบแบบ real-time

# 2. Record demos
# ทำ screen recording แสดงการทำงาน
# ใส่ใน PR description

# 3. Use PR templates
# .github/pull_request_template.md

# 4. Async standups
# ใช้ GitHub Discussions หรือ Slack
# แต่ละคนโพสต์ update ประจำวัน

# 5. Document decisions
# ใช้ GitHub Discussions สำหรับ architecture decisions
# ใช้ ADR (Architecture Decision Records)

# 6. Overlap hours
# กำหนดเวลา overlap ที่ทุกคน available
# เช่น 2 PM UTC = 
#   - 2 PM London
#   - 6 AM San Francisco
#   - 11 PM Tokyo (challenging!)
#   - 7:30 PM India
```

### Git Commit Practices for Async Teams

```bash
# Commits should be self-explanatory
git commit -m "feat(api): add user search endpoint

Added new endpoint GET /api/users/search
- Supports query parameter: name, email
- Returns paginated results
- Added tests and documentation

Resolves #123
See ADR-015 for design decisions"

# ไม่ควรเป็น
git commit -m "update"
git commit -m "fix bug"
git commit -m "changes"
```

---

## Comparison Table

| Aspect | Small Team (3-5) | Medium Team (10-20) | Large Team (50+) | Remote Team |
|--------|------------------|---------------------|------------------|-------------|
| **Strategy** | GitHub Flow | Git Flow | Monorepo | Async Workflow |
| **Branch Protection** | Basic | Moderate | Strict | Very Strict |
| **Required Reviewers** | 1 | 2 | 2-3 + CODEOWNERS | 2-3 + timezone aware |
| **CI/CD** | Simple | Pipeline per service | Matrix of services | Async notifications |
| **Release Frequency** | Daily/Continuous | Weekly/Bi-weekly | Bi-weekly train | Bi-weekly |
| **Merge Strategy** | Squash | No-FF | Squash + No-FF | Squash |
| **Communication** | Slack + calls | Slack + meetings | Docs + Slack | Async + overlap hours |

---

## Tools และ Integrations

### Code Review Tools

```bash
# GitHub CLI
gh pr list
gh pr view 123
gh pr review 123 --approve
gh pr merge 123 --squash

# Reviewdog (automated code review)
# .github/workflows/reviewdog.yml
- name: Run reviewdog
  uses: reviewdog/action-eslint@v1
  with:
    reporter: github-pr-review
```

### Communication Tools

```yaml
# Slack Integration
# .github/workflows/notify.yml
- name: Notify Slack
  uses: 8398a7/action-slack@v3
  with:
    status: ${{ job.status }}
    text: 'PR #${{ github.event.number }} merged to main'
    webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

### Monitoring Tools

```bash
# Git Analytics
git shortlog -sn  # commits per author
git log --since="1 month ago" --oneline | wc -l  # commits last month

# GitHub CLI
gh api repos/:owner/:repo/stats/contributors

# Third-party
# - GitPrime / Pluralsight Flow
# - Haystack
# - LinearB
```

---

## Summary

**เลือก Workflow ตามขนาดทีม:**

- **Small (3-5)**: GitHub Flow - Simple, fast
- **Medium (10-20)**: Git Flow - Structured, release-based
- **Large (50+)**: Monorepo - Scalable, service-based
- **Remote**: Async - Documentation, communication

**Key Success Factors:**
1. ชัด process ที่เหมาะกับทีม
2. Automate testing และ deployment
3. Document decisions และ architecture
4. Regular retrospectives และปรับปรุง
5. Tools ที่ support workflow
