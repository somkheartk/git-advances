# Git Flow Example - ตัวอย่างการใช้งานจริง

## สถานการณ์: พัฒนา E-commerce Platform

ทีมมี 15 คน แบ่งเป็น:
- Frontend Team (5 คน)
- Backend Team (5 คน)
- DevOps Team (2 คน)
- QA Team (3 คน)

## Timeline

### Sprint 1: เตรียมพัฒนา Version 2.0

```bash
# Week 1: Setup repository
git clone https://github.com/company/ecommerce.git
cd ecommerce

# สร้าง develop branch จาก main
git checkout -b develop
git push -u origin develop

# ตั้งค่า protected branches
# - main: requires 2 approvals
# - develop: requires 1 approval
```

### Week 2-3: Feature Development

```bash
# Frontend Team
# Developer 1: Shopping Cart
git checkout develop
git pull origin develop
git checkout -b feature/FE-101-shopping-cart
# ... coding ...
git add .
git commit -m "feat(cart): implement shopping cart UI"
git push origin feature/FE-101-shopping-cart
# Create PR to develop

# Developer 2: Product Listing
git checkout develop
git pull origin develop
git checkout -b feature/FE-102-product-listing
# ... coding ...
git commit -m "feat(products): add product listing with filters"
git push origin feature/FE-102-product-listing

# Developer 3: User Profile
git checkout develop
git pull origin develop
git checkout -b feature/FE-103-user-profile
# ... coding ...

# Backend Team
# Developer 4: Payment API
git checkout develop
git pull origin develop
git checkout -b feature/BE-201-payment-api
# ... coding ...
git commit -m "feat(api): implement Stripe payment integration"
git push origin feature/BE-201-payment-api

# Developer 5: Order Management
git checkout develop
git pull origin develop
git checkout -b feature/BE-202-order-management
# ... coding ...
```

### Week 4: Integration ใน Develop

```bash
# Merge features เข้า develop ตาม priority
git checkout develop
git pull origin develop

# Merge shopping cart (สำเร็จและ reviewed แล้ว)
git merge --no-ff feature/FE-101-shopping-cart
git push origin develop

# Merge payment API
git merge --no-ff feature/BE-201-payment-api
git push origin develop

# Testing ใน develop branch
# Run integration tests
npm run test:integration
```

### Week 5: Release Preparation

```bash
# สร้าง release branch
git checkout develop
git pull origin develop
git checkout -b release/v2.0.0
git push -u origin release/v2.0.0

# QA Team ทำ testing ใน release branch
# พบ bugs และ fix ใน release branch

# Bug fix 1: Cart calculation wrong
git checkout release/v2.0.0
git checkout -b bugfix/fix-cart-calculation
# ... fix bug ...
git commit -m "fix(cart): correct total price calculation"
git checkout release/v2.0.0
git merge --no-ff bugfix/fix-cart-calculation
git push origin release/v2.0.0

# Bug fix 2: Payment timeout
git checkout release/v2.0.0
git checkout -b bugfix/payment-timeout
# ... fix bug ...
git commit -m "fix(payment): increase timeout for payment gateway"
git checkout release/v2.0.0
git merge --no-ff bugfix/payment-timeout
git push origin release/v2.0.0
```

### Week 6: Release to Production

```bash
# QA approved! Ready to release

# Merge to main (production)
git checkout main
git pull origin main
git merge --no-ff release/v2.0.0

# Tag the release
git tag -a v2.0.0 -m "Release version 2.0.0 - E-commerce platform with shopping cart and payment"
git push origin main --tags

# Merge back to develop (ต้อง merge bug fixes กลับด้วย)
git checkout develop
git pull origin develop
git merge --no-ff release/v2.0.0
git push origin develop

# Delete release branch
git branch -d release/v2.0.0
git push origin --delete release/v2.0.0

# Deploy to production
# DevOps team triggers deployment pipeline
```

### Week 6 Day 3: Critical Bug in Production! 🔥

```bash
# พบว่า payment system มีปัญหาร้ายแรง
# ลูกค้าไม่สามารถชำระเงินได้

# สร้าง hotfix branch จาก main ทันที
git checkout main
git pull origin main
git checkout -b hotfix/v2.0.1-payment-critical

# แก้ไข bug
# ปัญหา: API key expired
# แก้: อัพเดท API key และเพิ่ม expiry check

git add .
git commit -m "fix(payment): update API key and add expiry validation"

# Test ใน hotfix branch
npm test
npm run test:payment

# Merge เข้า main ทันที
git checkout main
git merge --no-ff hotfix/v2.0.1-payment-critical

# Tag hotfix version
git tag -a v2.0.1 -m "Hotfix 2.0.1 - Critical payment API fix"
git push origin main --tags

# Deploy to production ทันที (priority สูงสุด)
# Monitor เป็นเวลา 1 ชั่วโมง

# เมื่อมั่นใจว่าแก้แล้ว merge กลับเข้า develop
git checkout develop
git merge --no-ff hotfix/v2.0.1-payment-critical
git push origin develop

# Delete hotfix branch
git branch -d hotfix/v2.0.1-payment-critical
git push origin --delete hotfix/v2.0.1-payment-critical
```

### Week 7: ทีมทำงานต่อใน develop

```bash
# ทีมเริ่มพัฒนา features ใหม่สำหรับ v2.1.0
git checkout develop
git pull origin develop  # จะได้ hotfix มาด้วย

# New features for v2.1.0
git checkout -b feature/FE-104-wishlist
git checkout -b feature/BE-203-recommendation-engine
git checkout -b feature/FE-105-product-reviews

# วงจรซ้ำไปเรื่อยๆ
```

## Diagram ของ Process

```
main (production)
  |
  v2.0.0 ────────────────────────> v2.0.1 (hotfix)
  |                                   |
  └─── develop ─────────────────────┘
         |
         ├─── feature/FE-101-shopping-cart ──> merged
         ├─── feature/FE-102-product-listing ──> merged
         ├─── feature/BE-201-payment-api ──> merged
         └─── release/v2.0.0 ──> merged to main & develop
                |
                ├─── bugfix/fix-cart-calculation ──> merged
                └─── bugfix/payment-timeout ──> merged
```

## Lessons Learned

### ✅ สิ่งที่ทำได้ดี
1. มี release branch แยกชัดเจน ทำให้ QA test ได้สะดวก
2. Hotfix process รวดเร็ว แก้ได้ใน 2 ชั่วโมง
3. Merge bug fixes กลับเข้า develop ทำให้ไม่สูญหาย
4. Protected branches ป้องกัน direct push

### ⚠️ ปัญหาที่เจอ
1. Feature branches บางตัวเก็บไว้นานเกินไป (3 weeks) conflict เยอะ
   - **Solution**: Sync กับ develop ทุกวัน
2. บาง developer ลืม merge hotfix เข้า feature branch ของตัวเอง
   - **Solution**: ส่ง notification เมื่อมี hotfix
3. Release branch มี bug เยอะ เพราะ testing ใน develop ไม่ดีพอ
   - **Solution**: เพิ่ม integration tests ใน CI/CD

### 🎯 Improvements for Next Sprint
1. ใช้ feature flags สำหรับ features ที่ยังไม่เสร็จ
2. Setup staging environment ที่ map กับ release branch
3. Automated deployment สำหรับ hotfix
4. Daily sync meeting สำหรับ merge conflicts
5. Better commit message conventions

## Metrics

```
Sprint 1 Statistics:
- Features completed: 5/7 (71%)
- Bugs in release: 8 (Medium severity)
- Hotfixes needed: 1 (Critical)
- Average feature development time: 1.5 weeks
- Time from release branch to production: 5 days
- Merge conflicts: 12 (resolved in < 1 hour each)
```

## Next Steps

```bash
# เตรียม Sprint 2 สำหรับ v2.1.0
git checkout develop
git pull origin develop

# Plan features for v2.1.0:
# - Wishlist functionality
# - Product recommendations
# - Product reviews and ratings
# - Advanced search
# - Mobile app integration

# Estimated timeline: 4 weeks
# Release date: 6 weeks from now
```
