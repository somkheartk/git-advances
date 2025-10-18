# Git Advances - คู่มือการใช้ Git ขั้นสูง

คู่มือสำหรับการใช้งาน Git ในระดับ Advanced พร้อม Use Cases สำหรับทีมขนาดใหญ่และ Workflows ระดับมืออาชีพ

## สารบัญ

1. [Git Branching Strategies](#git-branching-strategies)
2. [Advanced Git Commands](#advanced-git-commands)
3. [Team Collaboration Workflows](#team-collaboration-workflows)
4. [Conflict Resolution](#conflict-resolution)
5. [Code Review Best Practices](#code-review-best-practices)
6. [Use Cases สำหรับทีมใหญ่](#use-cases-สำหรับทีมใหญ่)
7. [Git Best Practices](#git-best-practices)

---

## Git Branching Strategies

### 1. Git Flow (แนะนำสำหรับโปรเจคที่มี Release Schedule ชัดเจน)

```
main (production)
  |
  └─── develop (integration branch)
         |
         ├─── feature/user-authentication
         ├─── feature/payment-system
         └─── release/v1.2.0
```

**Branch Types:**
- **main/master**: Production-ready code
- **develop**: Integration branch สำหรับ development
- **feature/**: Feature branches (สร้างจาก develop)
- **release/**: Release preparation (สร้างจาก develop)
- **hotfix/**: Urgent fixes (สร้างจาก main)

**Workflow:**
```bash
# สร้าง Feature Branch
git checkout develop
git checkout -b feature/new-feature

# เมื่อทำงานเสร็จ merge กลับเข้า develop
git checkout develop
git merge --no-ff feature/new-feature
git branch -d feature/new-feature

# สร้าง Release Branch
git checkout develop
git checkout -b release/v1.2.0
# ทำ bug fixes ใน release branch

# Merge release เข้า main และ develop
git checkout main
git merge --no-ff release/v1.2.0
git tag -a v1.2.0 -m "Release version 1.2.0"
git checkout develop
git merge --no-ff release/v1.2.0
git branch -d release/v1.2.0

# Hotfix
git checkout main
git checkout -b hotfix/critical-bug
# Fix the bug
git checkout main
git merge --no-ff hotfix/critical-bug
git tag -a v1.2.1 -m "Hotfix 1.2.1"
git checkout develop
git merge --no-ff hotfix/critical-bug
git branch -d hotfix/critical-bug
```

### 2. GitHub Flow (แนะนำสำหรับ Continuous Deployment)

```
main (production)
  |
  ├─── feature/add-login
  ├─── feature/fix-navbar
  └─── feature/update-readme
```

**Workflow:**
```bash
# สร้าง branch จาก main
git checkout main
git pull origin main
git checkout -b feature/descriptive-name

# Commit และ push เป็นประจำ
git add .
git commit -m "Add: user login functionality"
git push origin feature/descriptive-name

# สร้าง Pull Request บน GitHub
# หลัง code review ผ่าน ให้ merge เข้า main
# Deploy ทันทีหลัง merge
```

### 3. GitLab Flow (แนะนำสำหรับ Environment-based deployment)

```
main (development)
  |
  ├─── production (production environment)
  ├─── staging (staging environment)
  └─── feature/new-dashboard
```

**Workflow:**
```bash
# Development in main branch
git checkout main
git checkout -b feature/new-feature

# Merge to main when ready
git checkout main
git merge feature/new-feature

# Deploy to staging
git checkout staging
git merge main

# Deploy to production when stable
git checkout production
git merge staging
```

---

## Advanced Git Commands

### Interactive Rebase (จัดระเบียบ commits ก่อน merge)

```bash
# แก้ไข 3 commits ล่าสุด
git rebase -i HEAD~3

# ใน editor จะเห็น:
# pick abc123 First commit
# pick def456 Second commit
# pick ghi789 Third commit

# เปลี่ยนเป็น:
# pick abc123 First commit
# squash def456 Second commit  # รวมเข้า commit แรก
# reword ghi789 Third commit    # เปลี่ยนข้อความ
```

### Cherry Pick (เลือก commit จาก branch อื่นมาใช้)

```bash
# ดู commit hash ที่ต้องการ
git log --oneline

# Cherry pick commit นั้นมาใช้
git cherry-pick abc123

# Cherry pick หลาย commits
git cherry-pick abc123 def456 ghi789

# Cherry pick โดยไม่ commit ทันที
git cherry-pick -n abc123
```

### Stash (เก็บงานที่ทำค้างไว้ชั่วคราว)

```bash
# เก็บการเปลี่ยนแปลงปัจจุบัน
git stash

# เก็บพร้อมตั้งชื่อ
git stash save "WIP: working on login feature"

# ดูรายการ stash
git stash list

# นำ stash กลับมาใช้
git stash apply              # นำล่าสุดมาใช้ แต่ยังเก็บ stash ไว้
git stash pop                # นำล่าสุดมาใช้และลบ stash
git stash apply stash@{2}    # นำ stash ลำดับที่ 2 มาใช้

# ลบ stash
git stash drop stash@{0}
git stash clear              # ลบทั้งหมด
```

### Reflog (กู้คืนงานที่หายไป)

```bash
# ดูประวัติทุกการเปลี่ยนแปลง
git reflog

# กู้คืนไปยัง state ที่ต้องการ
git reset --hard HEAD@{2}

# สร้าง branch จาก commit เก่าที่หายไป
git branch recovered-branch abc123
```

### Bisect (หา commit ที่ทำให้เกิด bug)

```bash
# เริ่มต้น bisect
git bisect start

# บอกว่า commit ปัจจุบันมี bug
git bisect bad

# บอกว่า commit เก่ายังไม่มี bug
git bisect good abc123

# Git จะ checkout ไปที่ commit กลางๆ ให้ทดสอบ
# ทดสอบแล้วบอก good หรือ bad
git bisect good   # หรือ
git bisect bad

# ทำซ้ำจนเจอ commit ที่เป็นสาเหตุ
# เสร็จแล้วกลับสู่สภาพปกติ
git bisect reset
```

### Worktree (ทำงานหลาย branch พร้อมกัน)

```bash
# สร้าง worktree ใหม่
git worktree add ../git-advances-feature feature/new-feature

# ดูรายการ worktree
git worktree list

# ลบ worktree
git worktree remove ../git-advances-feature
```

### Advanced Log และ Diff

```bash
# ดู log แบบกราฟ
git log --graph --oneline --all --decorate

# ดู log เฉพาะไฟล์
git log --follow -- path/to/file

# ดู log ของ author เฉพาะคน
git log --author="John Doe"

# ดู log ในช่วงเวลา
git log --since="2 weeks ago" --until="yesterday"

# ดู diff ระหว่าง branch
git diff main..develop

# ดู diff เฉพาะชื่อไฟล์
git diff --name-only main..develop

# ดู diff แบบ word-by-word
git diff --word-diff
```

---

## Team Collaboration Workflows

### Use Case 1: ทีมพัฒนา Feature ใหม่แบบ Parallel

**สถานการณ์:** มีนักพัฒนา 10 คน ทำงาน 5 features พร้อมกัน

```bash
# แต่ละคนสร้าง feature branch ของตัวเอง
Developer 1: git checkout -b feature/user-profile
Developer 2: git checkout -b feature/payment-gateway
Developer 3: git checkout -b feature/notification-system
Developer 4: git checkout -b feature/search-functionality
Developer 5: git checkout -b feature/admin-dashboard

# Sync กับ develop เป็นประจำเพื่อลด conflicts
git checkout develop
git pull origin develop
git checkout feature/user-profile
git merge develop
# แก้ conflicts ถ้ามี
git push origin feature/user-profile

# เมื่อ feature เสร็จ สร้าง Pull Request
# หลัง code review ผ่าน merge เข้า develop
```

**Best Practices:**
- ตั้งชื่อ branch ให้ descriptive: `feature/TICKET-123-user-authentication`
- Commit บ่อยๆ แต่ละ commit ควรทำงานได้สมบูรณ์
- Push ขึ้น remote เป็นประจำเป็น backup
- Sync กับ base branch อย่างน้อยวันละครั้ง

### Use Case 2: Release Management สำหรับทีมใหญ่

**สถานการณ์:** เตรียม release version 2.0.0 โดยมี features หลายตัวรวมกัน

```bash
# สร้าง release branch จาก develop
git checkout develop
git pull origin develop
git checkout -b release/v2.0.0

# Freeze feature development
# อนุญาตเฉพาะ bug fixes และ documentation

# Bug fixes ใน release branch
git checkout release/v2.0.0
git checkout -b bugfix/login-error
# Fix bug
git checkout release/v2.0.0
git merge bugfix/login-error

# QA testing ใน release branch
# เมื่อ QA ผ่าน merge เข้า main และ develop

# Merge to main
git checkout main
git pull origin main
git merge --no-ff release/v2.0.0
git tag -a v2.0.0 -m "Release version 2.0.0"
git push origin main --tags

# Merge back to develop
git checkout develop
git merge --no-ff release/v2.0.0
git push origin develop

# Delete release branch
git branch -d release/v2.0.0
git push origin --delete release/v2.0.0
```

### Use Case 3: Hotfix ฉุกเฉินใน Production

**สถานการณ์:** เจอ critical bug ใน production ต้องแก้ทันที

```bash
# สร้าง hotfix branch จาก main
git checkout main
git pull origin main
git checkout -b hotfix/critical-security-fix

# แก้ bug แล้ว test ให้แน่ใจ
git add .
git commit -m "Fix: critical security vulnerability in authentication"

# Merge เข้า main (production)
git checkout main
git merge --no-ff hotfix/critical-security-fix
git tag -a v2.0.1 -m "Hotfix 2.0.1 - Security fix"
git push origin main --tags

# Merge เข้า develop ด้วย
git checkout develop
git merge --no-ff hotfix/critical-security-fix
git push origin develop

# Delete hotfix branch
git branch -d hotfix/critical-security-fix
git push origin --delete hotfix/critical-security-fix

# Deploy to production ทันที
```

### Use Case 4: Code Review Process

**Workflow สำหรับทีมใหญ่:**

```bash
# Developer สร้าง Pull Request
git checkout -b feature/new-payment-method
# เขียนโค้ด
git add .
git commit -m "Add: Stripe payment integration"
git push origin feature/new-payment-method

# สร้าง PR บน GitHub/GitLab พร้อมกับ:
# - Description ที่ชัดเจน
# - Link to ticket/issue
# - Screenshots (ถ้ามีการเปลี่ยนแปลง UI)
# - Test coverage

# Reviewer checkout PR มา review
git fetch origin
git checkout feature/new-payment-method

# หรือใช้ GitHub CLI
gh pr checkout 123

# Reviewer ให้ feedback
# - Request changes ถ้ามีปัญหา
# - Approve ถ้าโอเค
# - Comment สำหรับคำแนะนำ

# Developer แก้ไขตาม feedback
git add .
git commit -m "Fix: address review comments"
git push origin feature/new-payment-method

# หลัง approve ให้ merge (ใช้ Squash merge ถ้าต้องการ history สะอาด)
# GitHub จะทำการ merge และลบ branch อัตโนมัติ
```

---

## Conflict Resolution

### กลยุทธ์การแก้ Conflicts

#### 1. Merge Conflicts

```bash
# เมื่อเกิด conflict ระหว่าง merge
git checkout develop
git merge feature/new-feature

# Auto-merging file.txt
# CONFLICT (content): Merge conflict in file.txt
# Automatic merge failed; fix conflicts and then commit the result.

# ดูไฟล์ที่มี conflict
git status

# แก้ไข conflict ในไฟล์
# ลบ conflict markers (<<<<<<, ======, >>>>>>)
# เลือกเก็บโค้ดส่วนไหน

# หลังแก้เสร็จ
git add file.txt
git commit -m "Merge: resolve conflicts between develop and feature/new-feature"
```

**ไฟล์ที่มี conflict จะมีลักษณะ:**
```
<<<<<<< HEAD
// โค้ดจาก branch ปัจจุบัน
=======
// โค้ดจาก branch ที่ merge เข้ามา
>>>>>>> feature/new-feature
```

#### 2. ใช้ Merge Tool

```bash
# ตั้งค่า merge tool (เช่น VS Code, meld, kdiff3)
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'

# เมื่อเจอ conflict ใช้ merge tool
git mergetool

# หลังแก้เสร็จ
git commit
```

#### 3. Rebase Conflicts

```bash
# เมื่อเกิด conflict ระหว่าง rebase
git rebase develop

# CONFLICT (content): Merge conflict in file.txt
# error: could not apply abc123...

# แก้ conflict ในไฟล์
# หลังแก้เสร็จ
git add file.txt
git rebase --continue

# หรือ skip commit นี้
git rebase --skip

# หรือ ยกเลิก rebase
git rebase --abort
```

#### 4. ป้องกัน Conflicts

**Best Practices:**
```bash
# 1. Sync บ่อยๆ
git checkout develop
git pull origin develop
git checkout feature/my-feature
git merge develop

# 2. แบ่งงานให้ชัดเจน ไม่ให้คนละคนแก้ไฟล์เดียวกัน

# 3. ใช้ feature flags แทนการ comment code

# 4. Commit เล็กๆ บ่อยๆ แทน commit ใหญ่ทีเดียว

# 5. Code review ก่อน merge เข้า main branch
```

---

## Code Review Best Practices

### สำหรับ Developer (Author)

**ก่อนสร้าง PR:**
```bash
# ตรวจสอบ changes ของตัวเอง
git diff develop..feature/my-feature

# Rebase เพื่อให้ history สะอาด
git rebase -i develop

# รัน tests
npm test  # หรือ pytest, mvn test, etc.

# Lint code
npm run lint

# Push
git push origin feature/my-feature
```

**PR Description Template:**
```markdown
## What
สิ่งที่เปลี่ยนแปลงในการ PR นี้

## Why
เหตุผลที่ต้องเปลี่ยน / ticket ที่เกี่ยวข้อง

## How
วิธีการทำงานของโค้ดใหม่

## Testing
- [ ] Unit tests ผ่าน
- [ ] Integration tests ผ่าน
- [ ] Manual testing ผ่าน

## Screenshots (ถ้ามีการเปลี่ยน UI)

## Checklist
- [ ] Code follows style guide
- [ ] Documentation updated
- [ ] No console.log / debug code
- [ ] Tests added/updated
```

### สำหรับ Reviewer

**Review Checklist:**
```
✓ Code quality และ readability
✓ ตรงตาม requirement หรือไม่
✓ มี test coverage เพียงพอ
✓ Performance considerations
✓ Security issues
✓ Error handling
✓ Documentation ครบถ้วน
✓ Breaking changes
```

**Review Commands:**
```bash
# Checkout PR มา review ในเครื่อง
gh pr checkout 123

# หรือ
git fetch origin pull/123/head:pr-123
git checkout pr-123

# ดู changes
git diff develop..pr-123

# Test locally
npm install
npm test
npm start

# Comment บน GitHub พร้อม suggestions
```

**Feedback Guidelines:**
- ให้ feedback แบบ constructive
- Explain "why" ไม่ใช่แค่ "what"
- ชมในสิ่งที่ทำดี
- Request changes ถ้าจำเป็น Approve ถ้าโอเค
- Nitpick comments ควรระบุว่าไม่จำเป็นต้องแก้

---

## Use Cases สำหรับทีมใหญ่

### Use Case 5: Monorepo กับทีมหลายสิบคน

**โครงสร้าง:**
```
monorepo/
├── packages/
│   ├── frontend/
│   ├── backend/
│   ├── mobile/
│   └── shared/
├── .github/
│   └── CODEOWNERS
└── .gitignore
```

**CODEOWNERS File:**
```
# Frontend team
/packages/frontend/ @team-frontend

# Backend team
/packages/backend/ @team-backend

# Mobile team
/packages/mobile/ @team-mobile

# Everyone can edit shared
/packages/shared/ @team-frontend @team-backend @team-mobile
```

**Workflow:**
```bash
# แต่ละทีมทำงานในส่วนของตัวเอง
git checkout -b feature/frontend-dashboard
# Changes only in packages/frontend/

# CI/CD runs tests เฉพาะส่วนที่เปลี่ยน
# Deploy เฉพาะ package ที่เปลี่ยน
```

### Use Case 6: Feature Flags สำหรับ Progressive Rollout

```bash
# Merge feature ที่ยังไม่เสร็จเข้า main โดยใช้ feature flag
git checkout -b feature/new-payment-system

# Code with feature flag
if (featureFlags.newPaymentSystem) {
  // โค้ดใหม่
} else {
  // โค้ดเก่า
}

# Merge เข้า main ทันที (feature flag off)
git checkout main
git merge feature/new-payment-system

# Deploy to production
# ค่อยๆ เปิด feature flag ให้ user บางส่วน

# เมื่อมั่นใจแล้ว เปิดให้ทุกคน
# ลบโค้ดเก่าและ feature flag ออก
```

### Use Case 7: Long-running Feature Branches

**สถานการณ์:** Feature ใหญ่ที่ใช้เวลาพัฒนาหลายเดือน

```bash
# สร้าง feature branch
git checkout -b feature/major-redesign

# Sub-features แยก branch ย่อย
git checkout -b feature/major-redesign/header
git checkout -b feature/major-redesign/footer
git checkout -b feature/major-redesign/sidebar

# Merge sub-features เข้า feature branch
git checkout feature/major-redesign
git merge feature/major-redesign/header

# Sync กับ develop เป็นประจำ (สำคัญมาก!)
git checkout develop
git pull origin develop
git checkout feature/major-redesign
git merge develop
# แก้ conflicts เรื่อยๆ แทนที่จะปล่อยทิ้งไว้

# เมื่อเสร็จทั้งหมด merge เข้า develop
git checkout develop
git merge --no-ff feature/major-redesign
```

### Use Case 8: การจัดการ Dependencies Across Teams

```bash
# Team A พัฒนา shared library
git checkout -b feature/shared-auth-library
# พัฒนา library
git push origin feature/shared-auth-library

# Team B ต้องใช้ library ของ Team A ก่อนจะเสร็จ
# สร้าง temporary dependency
git checkout -b feature/use-new-auth
# Add dependency to feature branch ของ Team A
git merge feature/shared-auth-library

# หรือใช้ npm link / yarn link สำหรับ development

# เมื่อ Team A merge เสร็จ Team B ก็ sync ตาม
```

---

## Git Best Practices

### Commit Messages

**Format:**
```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
- `feat`: Feature ใหม่
- `fix`: Bug fix
- `docs`: Documentation
- `style`: Formatting, semicolons, etc
- `refactor`: Code refactoring
- `test`: เพิ่ม tests
- `chore`: Build, dependencies, etc

**Examples:**
```bash
git commit -m "feat(auth): add OAuth2 authentication"
git commit -m "fix(payment): resolve timeout issue in payment gateway"
git commit -m "docs(readme): update installation instructions"
git commit -m "refactor(api): simplify user controller logic"
```

### Branch Naming

**Conventions:**
```
feature/TICKET-123-short-description
bugfix/TICKET-456-fix-login-error
hotfix/critical-security-patch
release/v1.2.0
experiment/try-new-framework
```

### Git Hooks

**Pre-commit Hook Example:**
```bash
# .git/hooks/pre-commit
#!/bin/sh

# Run linter
npm run lint

# Run tests
npm test

# Check for console.log
if git diff --cached | grep -E "console\.(log|debug|info)"; then
    echo "Error: console.log found. Please remove it."
    exit 1
fi
```

**Setup with Husky:**
```bash
npm install --save-dev husky
npx husky install
npx husky add .git/hooks/pre-commit "npm test"
npx husky add .git/hooks/pre-push "npm run lint"
```

### .gitignore Best Practices

```bash
# Dependencies
node_modules/
vendor/

# Build output
dist/
build/
*.log

# Environment
.env
.env.local

# IDE
.vscode/
.idea/
*.swp

# OS
.DS_Store
Thumbs.db

# Test coverage
coverage/
```

### Git Aliases (เพิ่มความเร็ว)

```bash
# เพิ่มใน ~/.gitconfig
[alias]
    st = status
    co = checkout
    br = branch
    ci = commit
    lg = log --graph --oneline --all --decorate
    last = log -1 HEAD
    unstage = reset HEAD --
    undo = reset --soft HEAD^
    amend = commit --amend --no-edit
    please = push --force-with-lease
    
# ใช้งาน
git st
git lg
git amend
```

### Performance Tips สำหรับ Large Repositories

```bash
# Shallow clone (ไม่เอา history ทั้งหมด)
git clone --depth 1 https://github.com/user/repo.git

# Partial clone (ไม่ download ไฟล์ใหญ่ทั้งหมด)
git clone --filter=blob:none https://github.com/user/repo.git

# Git maintenance
git gc
git prune

# Enable file system monitor
git config core.fsmonitor true
git config core.untrackedCache true
```

### Security Best Practices

```bash
# ไม่ commit secrets
# ใช้ .gitignore สำหรับ .env files

# ตรวจสอบ secrets ใน history
git log -p | grep -i "password"

# ถ้า commit secrets ไปแล้ว ต้อง rewrite history
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch path/to/secret-file" \
  --prune-empty --tag-name-filter cat -- --all

# หรือใช้ BFG Repo-Cleaner (เร็วกว่า)
bfg --delete-files secret-file.txt

# Force push (ระวัง!)
git push origin --force --all
git push origin --force --tags

# Sign commits (สำหรับ security)
git config --global user.signingkey YOUR_GPG_KEY
git config --global commit.gpgsign true
git commit -S -m "Signed commit"
```

---

## เครื่องมือเสริมสำหรับทีมใหญ่

### 1. GitHub Actions / GitLab CI/CD

**Example Workflow:**
```yaml
name: CI/CD Pipeline

on:
  pull_request:
    branches: [develop, main]
  push:
    branches: [develop, main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run tests
        run: npm test
      - name: Run linter
        run: npm run lint
      - name: Check coverage
        run: npm run coverage

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Build
        run: npm run build

  deploy:
    needs: build
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to production
        run: ./deploy.sh
```

### 2. Git Large File Storage (LFS)

```bash
# สำหรับไฟล์ใหญ่ (images, videos, datasets)
git lfs install

# Track ไฟล์ประเภทที่ต้องการ
git lfs track "*.psd"
git lfs track "*.mp4"

# Commit .gitattributes
git add .gitattributes
git commit -m "chore: setup Git LFS"
```

### 3. Protected Branches

**GitHub Settings:**
```
Settings > Branches > Add rule

Rules:
☑ Require pull request reviews before merging
☑ Require status checks to pass before merging
☑ Require branches to be up to date before merging
☑ Require signed commits
☑ Include administrators
☑ Restrict who can push to matching branches
```

---

## สรุป

การใช้ Git ในระดับ Advanced สำหรับทีมขนาดใหญ่ต้องอาศัย:

1. **Branching Strategy ที่เหมาะสม** - เลือก Git Flow, GitHub Flow หรือ GitLab Flow ตามลักษณะโปรเจค
2. **Clear Workflows** - กำหนด process ชัดเจนสำหรับ feature development, release, และ hotfix
3. **Code Review Process** - Review ก่อน merge เสมอ
4. **Automation** - ใช้ CI/CD, Git hooks, และ automated testing
5. **Communication** - ใช้ PR descriptions, commit messages ที่ชัดเจน
6. **Best Practices** - ตั้งชื่อ branch และ commit ตาม convention
7. **Conflict Management** - Sync บ่อยๆ และแก้ conflicts ทันที
8. **Security** - ไม่ commit secrets, ใช้ signed commits

### แนะนำสำหรับทีมใหม่

1. เริ่มจาก GitHub Flow (ง่ายที่สุด)
2. เพิ่ม Code Review Process
3. Setup CI/CD Pipeline
4. เพิ่ม Protected Branches
5. ค่อยๆ ขยับไปยัง Git Flow เมื่อโปรเจคใหญ่ขึ้น

---

## Resources

- [Pro Git Book](https://git-scm.com/book/en/v2)
- [GitHub Flow Guide](https://guides.github.com/introduction/flow/)
- [Git Flow Original Post](https://nvie.com/posts/a-successful-git-branching-model/)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [Oh Shit, Git!?!](https://ohshitgit.com/) - แก้ปัญหา Git ต่างๆ

---

**Happy Coding! 🚀**