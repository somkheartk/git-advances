# CI/CD Integration with Git Workflows

## GitHub Actions Examples

### Basic CI Pipeline

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Run linter
        run: npm run lint
        
      - name: Run tests
        run: npm test
        
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/coverage-final.json
```

### Advanced Pipeline with Multiple Jobs

```yaml
# .github/workflows/advanced-ci.yml
name: Advanced CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run lint

  test-unit:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node-version: [16, 18, 20]
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm ci
      - run: npm run test:unit

  test-integration:
    runs-on: ubuntu-latest
    needs: [lint, test-unit]
    services:
      postgres:
        image: postgres:14
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run test:integration
        env:
          DATABASE_URL: postgres://postgres:postgres@localhost:5432/test

  build:
    runs-on: ubuntu-latest
    needs: [test-integration]
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v3
        with:
          name: build-output
          path: dist/

  deploy-staging:
    runs-on: ubuntu-latest
    needs: [build]
    if: github.ref == 'refs/heads/develop'
    environment:
      name: staging
      url: https://staging.example.com
    steps:
      - uses: actions/download-artifact@v3
        with:
          name: build-output
      - name: Deploy to staging
        run: |
          # Deploy commands
          echo "Deploying to staging..."

  deploy-production:
    runs-on: ubuntu-latest
    needs: [build]
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://example.com
    steps:
      - uses: actions/download-artifact@v3
        with:
          name: build-output
      - name: Deploy to production
        run: |
          # Deploy commands
          echo "Deploying to production..."
```

### Monorepo CI with Path Filtering

```yaml
# .github/workflows/monorepo-ci.yml
name: Monorepo CI

on:
  pull_request:
  push:
    branches: [main]

jobs:
  changes:
    runs-on: ubuntu-latest
    outputs:
      frontend: ${{ steps.filter.outputs.frontend }}
      backend: ${{ steps.filter.outputs.backend }}
      shared: ${{ steps.filter.outputs.shared }}
    steps:
      - uses: actions/checkout@v3
      - uses: dorny/paths-filter@v2
        id: filter
        with:
          filters: |
            frontend:
              - 'packages/frontend/**'
            backend:
              - 'packages/backend/**'
            shared:
              - 'packages/shared/**'

  test-frontend:
    needs: changes
    if: needs.changes.outputs.frontend == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm test --workspace=packages/frontend

  test-backend:
    needs: changes
    if: needs.changes.outputs.backend == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm test --workspace=packages/backend

  test-shared:
    needs: changes
    if: needs.changes.outputs.shared == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm ci
      - run: npm test --workspace=packages/shared
```

### Release Automation

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    tags:
      - 'v*'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
          
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
          
      - run: npm ci
      - run: npm run build
      - run: npm test
      
      - name: Create Release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ github.ref }}
          release_name: Release ${{ github.ref }}
          body: |
            Changes in this release:
            - Feature 1
            - Feature 2
            - Bug fix 3
          draft: false
          prerelease: false
          
      - name: Publish to npm
        run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

## GitLab CI Examples

### Basic Pipeline

```yaml
# .gitlab-ci.yml
stages:
  - lint
  - test
  - build
  - deploy

variables:
  NODE_VERSION: "18"

lint:
  stage: lint
  image: node:${NODE_VERSION}
  script:
    - npm ci
    - npm run lint
  only:
    - merge_requests
    - main
    - develop

test:
  stage: test
  image: node:${NODE_VERSION}
  script:
    - npm ci
    - npm test
  coverage: '/Statements\s*:\s*(\d+\.\d+)%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml
  only:
    - merge_requests
    - main
    - develop

build:
  stage: build
  image: node:${NODE_VERSION}
  script:
    - npm ci
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 hour
  only:
    - main
    - develop

deploy_staging:
  stage: deploy
  image: alpine:latest
  script:
    - apk add --no-cache openssh-client
    - echo "Deploying to staging..."
    # Deploy commands
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - develop

deploy_production:
  stage: deploy
  image: alpine:latest
  script:
    - echo "Deploying to production..."
    # Deploy commands
  environment:
    name: production
    url: https://example.com
  when: manual
  only:
    - main
```

### Advanced GitLab Pipeline

```yaml
# .gitlab-ci.yml
include:
  - template: Security/SAST.gitlab-ci.yml
  - template: Security/Dependency-Scanning.gitlab-ci.yml

stages:
  - lint
  - test
  - security
  - build
  - deploy

variables:
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: ""

.node_base:
  image: node:18
  before_script:
    - npm ci
  cache:
    key: ${CI_COMMIT_REF_SLUG}
    paths:
      - node_modules/

lint:
  extends: .node_base
  stage: lint
  script:
    - npm run lint
    - npm run format:check

test:unit:
  extends: .node_base
  stage: test
  script:
    - npm run test:unit
  coverage: '/All files[^|]*\|[^|]*\s+([\d\.]+)/'
  artifacts:
    reports:
      junit: junit.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

test:integration:
  extends: .node_base
  stage: test
  services:
    - postgres:14
  variables:
    POSTGRES_DB: test_db
    POSTGRES_USER: test_user
    POSTGRES_PASSWORD: test_pass
    DATABASE_URL: postgres://test_user:test_pass@postgres:5432/test_db
  script:
    - npm run test:integration

test:e2e:
  extends: .node_base
  stage: test
  services:
    - name: selenium/standalone-chrome:latest
      alias: chrome
  script:
    - npm run test:e2e
  artifacts:
    when: on_failure
    paths:
      - cypress/videos/
      - cypress/screenshots/
    expire_in: 1 week

build:docker:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA $CI_REGISTRY_IMAGE:latest
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - docker push $CI_REGISTRY_IMAGE:latest
  only:
    - main
    - develop

deploy:staging:
  stage: deploy
  image: alpine:latest
  before_script:
    - apk add --no-cache curl
  script:
    - curl -X POST $STAGING_WEBHOOK_URL
  environment:
    name: staging
    url: https://staging.example.com
    on_stop: stop_staging
  only:
    - develop

stop_staging:
  stage: deploy
  image: alpine:latest
  script:
    - echo "Stopping staging environment"
  environment:
    name: staging
    action: stop
  when: manual
  only:
    - develop

deploy:production:
  stage: deploy
  image: alpine:latest
  before_script:
    - apk add --no-cache curl
  script:
    - curl -X POST $PRODUCTION_WEBHOOK_URL
  environment:
    name: production
    url: https://example.com
  when: manual
  only:
    - main
    - tags
```

## Branch Protection Rules

### GitHub Branch Protection

```yaml
# กำหนดผ่าน GitHub UI หรือ API
# Settings > Branches > Add rule

Rules for 'main' branch:
- ✓ Require pull request reviews before merging
  - Required approving reviews: 2
  - Dismiss stale pull request approvals when new commits are pushed
  - Require review from Code Owners
- ✓ Require status checks to pass before merging
  - Require branches to be up to date before merging
  - Status checks: [lint, test, build]
- ✓ Require signed commits
- ✓ Include administrators
- ✓ Restrict who can push to matching branches
  - Teams/Users: [tech-leads, devops-team]
- ✓ Allow force pushes: Nobody
- ✓ Allow deletions: No
```

### GitLab Protected Branches

```yaml
# Settings > Repository > Protected Branches

main:
  Allowed to merge: Maintainers
  Allowed to push: No one
  Allowed to force push: No
  Code owner approval: Required

develop:
  Allowed to merge: Developers + Maintainers
  Allowed to push: No one
  Allowed to force push: No
  Code owner approval: Not required
```

## Pre-commit Hooks

### Using Husky

```json
// package.json
{
  "scripts": {
    "prepare": "husky install"
  },
  "devDependencies": {
    "husky": "^8.0.0",
    "lint-staged": "^13.0.0"
  },
  "lint-staged": {
    "*.js": ["eslint --fix", "prettier --write"],
    "*.{json,md}": ["prettier --write"]
  }
}
```

```bash
# .husky/pre-commit
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npx lint-staged
npm test
```

```bash
# .husky/commit-msg
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

# Validate commit message format
npx --no -- commitlint --edit "$1"
```

### Pre-push Hook

```bash
# .husky/pre-push
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

# Run full test suite before pushing
npm test

# Check if branch is up to date
BRANCH=$(git rev-parse --abbrev-ref HEAD)
git fetch origin

LOCAL=$(git rev-parse @)
REMOTE=$(git rev-parse @{u})

if [ $LOCAL != $REMOTE ]; then
    echo "Error: Branch is not up to date. Please pull first."
    exit 1
fi
```

## Semantic Versioning with CI

```yaml
# .github/workflows/semantic-release.yml
name: Semantic Release

on:
  push:
    branches:
      - main

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
          
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
          
      - run: npm ci
      
      - name: Semantic Release
        uses: cycjimmy/semantic-release-action@v3
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
        with:
          extra_plugins: |
            @semantic-release/changelog
            @semantic-release/git
```

```json
// .releaserc.json
{
  "branches": ["main"],
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    "@semantic-release/changelog",
    [
      "@semantic-release/npm",
      {
        "npmPublish": true
      }
    ],
    [
      "@semantic-release/git",
      {
        "assets": ["package.json", "CHANGELOG.md"],
        "message": "chore(release): ${nextRelease.version} [skip ci]\n\n${nextRelease.notes}"
      }
    ],
    "@semantic-release/github"
  ]
}
```

## Summary

**Key Integration Points:**

1. **CI/CD Pipeline** - Automated testing และ deployment
2. **Branch Protection** - ป้องกัน direct commits และ enforce reviews
3. **Pre-commit Hooks** - Validate code ก่อน commit
4. **Semantic Release** - Automate versioning และ changelog
5. **Code Coverage** - Track test coverage ตลอดเวลา
6. **Security Scanning** - Detect vulnerabilities ก่อน merge

**Best Practices:**

- ใช้ CI/CD สำหรับทุก branch
- Protected branches สำหรับ main และ develop
- Automated testing ก่อน merge
- Code review เป็นส่วนหนึ่งของ pipeline
- Deployment automation ลด human errors
- Monitor และ alert เมื่อมีปัญหา
