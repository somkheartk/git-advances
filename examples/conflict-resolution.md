# Conflict Resolution Examples - ตัวอย่างการแก้ Conflicts

## Scenario 1: Simple Merge Conflict

### สถานการณ์
- Developer A แก้ไฟล์ `user.js` เพิ่ม email validation
- Developer B แก้ไฟล์เดียวกัน เพิ่ม phone validation
- เกิด conflict เมื่อ merge

### ก่อนเกิด Conflict

**Developer A's branch (feature/email-validation):**
```javascript
// user.js
class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }
  
  validate() {
    if (!this.email.includes('@')) {
      throw new Error('Invalid email');
    }
    return true;
  }
}
```

**Developer B's branch (feature/phone-validation):**
```javascript
// user.js
class User {
  constructor(name, email, phone) {
    this.name = name;
    this.email = email;
    this.phone = phone;
  }
  
  validate() {
    if (this.phone.length < 10) {
      throw new Error('Invalid phone number');
    }
    return true;
  }
}
```

### เกิด Conflict

```bash
# Developer B tries to merge
git checkout develop
git pull origin develop
git merge feature/phone-validation  # A's changes already in develop
```

**Output:**
```
Auto-merging user.js
CONFLICT (content): Merge conflict in user.js
Automatic merge failed; fix conflicts and then commit the result.
```

### ไฟล์ที่มี Conflict Markers

```javascript
// user.js
class User {
<<<<<<< HEAD
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }
  
  validate() {
    if (!this.email.includes('@')) {
      throw new Error('Invalid email');
    }
=======
  constructor(name, email, phone) {
    this.name = name;
    this.email = email;
    this.phone = phone;
  }
  
  validate() {
    if (this.phone.length < 10) {
      throw new Error('Invalid phone number');
    }
>>>>>>> feature/phone-validation
    return true;
  }
}
```

### วิธีแก้ (Manual Resolution)

```bash
# เปิดไฟล์ user.js และแก้เป็น:
```

```javascript
// user.js - รวม changes จากทั้งสองฝ่าย
class User {
  constructor(name, email, phone) {
    this.name = name;
    this.email = email;
    this.phone = phone;
  }
  
  validate() {
    // รวมทั้งสอง validations
    if (!this.email.includes('@')) {
      throw new Error('Invalid email');
    }
    if (this.phone && this.phone.length < 10) {
      throw new Error('Invalid phone number');
    }
    return true;
  }
}
```

```bash
# หลังแก้เสร็จ
git add user.js
git commit -m "Merge: resolve conflict between email and phone validation"
git push origin develop
```

---

## Scenario 2: Rebase Conflict

### สถานการณ์
- Feature branch ทำงานนาน 2 สัปดาห์
- develop branch มีการเปลี่ยนแปลงเยอะ
- ต้อง rebase เพื่อให้ history สะอาด

### Commands

```bash
# อยู่ใน feature branch
git checkout feature/long-running-feature
git fetch origin
git rebase origin/develop
```

**Output:**
```
First, rewinding head to replay your work on top of it...
Applying: Add user authentication
error: could not apply abc123... Add user authentication
Resolve all conflicts manually, mark them as resolved with
"git add/rm <conflicted_files>", then run "git rebase --continue".
```

### แก้ Conflict ทีละ Commit

```bash
# ดูไฟล์ที่ conflict
git status

# แก้ไข conflicts ในไฟล์
# ลบ conflict markers (<<<<<<, ======, >>>>>>)

# Mark as resolved
git add conflicted-file.js

# Continue rebase
git rebase --continue

# ถ้ามี conflicts ใน commit ถัดไป ทำซ้ำ
# หรือ skip commit ถ้าไม่จำเป็น
git rebase --skip

# หรือ abort ทั้งหมด
git rebase --abort
```

### Tips สำหรับ Rebase Conflicts

```bash
# ดู commit ที่กำลัง apply
git log -1

# ดู changes ที่ต้อง apply
git show

# ใช้ merge tool
git mergetool

# หลัง resolve ทุก commits
git push origin feature/long-running-feature --force-with-lease
```

---

## Scenario 3: Binary File Conflicts

### สถานการณ์
- สอง developers แก้ไฟล์ image/logo เดียวกัน
- Git ไม่สามารถ auto-merge binary files ได้

```bash
# Conflict message
CONFLICT (content): Merge conflict in assets/logo.png
```

### วิธีแก้

```bash
# Option 1: เลือกเอา version จาก branch หนึ่ง
git checkout --ours assets/logo.png     # เอา version ของเรา
git checkout --theirs assets/logo.png   # เอา version ของพวกเขา

# Option 2: ใช้ version ใหม่ทั้งหมด
# Copy ไฟล์ใหม่เข้ามาแทน
cp ~/new-logo.png assets/logo.png

# Mark as resolved
git add assets/logo.png
git commit -m "Resolve: use new logo design"
```

---

## Scenario 4: Multiple File Conflicts

### สถานการณ์
- Merge ครั้งใหญ่ มี conflicts ใน 15 ไฟล์

```bash
git merge feature/major-refactoring
```

**Output:**
```
CONFLICT (content): Merge conflict in src/user.js
CONFLICT (content): Merge conflict in src/auth.js
CONFLICT (content): Merge conflict in src/api.js
... (12 more files)
Automatic merge failed; fix conflicts and then commit the result.
```

### Strategy

```bash
# 1. ดูภาพรวมก่อน
git status

# 2. แยก fix ตาม priority
# - Critical files ก่อน (auth, api)
# - Less critical files ทีหลัง (tests, docs)

# 3. ใช้ merge tool สำหรับ complex conflicts
git mergetool

# 4. สำหรับ simple conflicts แก้ manual
code src/user.js

# 5. Mark resolved ทีละไฟล์
git add src/auth.js
git add src/api.js
# ... etc

# 6. Test ก่อน commit
npm test

# 7. Commit เมื่อแก้ครบแล้ว
git commit -m "Merge: resolve conflicts from major refactoring"
```

---

## Scenario 5: Conflict ใน Package Dependencies

### สถานการณ์
- Branch A update React 17 → 18
- Branch B update Jest 26 → 27
- Conflict ใน package.json และ package-lock.json

### package.json Conflict

```json
{
  "dependencies": {
<<<<<<< HEAD
    "react": "^18.0.0",
    "react-dom": "^18.0.0",
    "jest": "^26.0.0"
=======
    "react": "^17.0.0",
    "react-dom": "^17.0.0",
    "jest": "^27.0.0"
>>>>>>> feature/update-jest
  }
}
```

### วิธีแก้

```bash
# Option 1: เลือก version ที่ต้องการ manual
# แก้ package.json เป็น:
{
  "dependencies": {
    "react": "^18.0.0",
    "react-dom": "^18.0.0",
    "jest": "^27.0.0"
  }
}

# Option 2: ใช้ npm/yarn regenerate
git checkout --theirs package-lock.json
npm install

# Test ว่าทำงานด้วยกัน
npm test

# Commit
git add package.json package-lock.json
git commit -m "Merge: update React 18 and Jest 27"
```

---

## Scenario 6: Deleted vs Modified Conflict

### สถานการณ์
- Branch A ลบไฟล์ `old-api.js`
- Branch B แก้ไฟล์เดียวกัน

```bash
CONFLICT (modify/delete): old-api.js deleted in HEAD and modified in feature/update-api.
```

### วิธีแก้

```bash
# ดูว่าการแก้ไขมีความสำคัญไหม
git show feature/update-api:old-api.js

# Option 1: ลบทิ้งถ้าไม่จำเป็น
git rm old-api.js

# Option 2: เก็บไว้ถ้าจำเป็น
git add old-api.js

# Option 3: ย้าย logic ไปไฟล์ใหม่
# เอา changes จากไฟล์เก่ามาใส่ในไฟล์ใหม่
git show feature/update-api:old-api.js > /tmp/changes.txt
# แก้ new-api.js ตาม changes
git rm old-api.js
git add new-api.js

git commit -m "Merge: migrate old-api changes to new-api"
```

---

## Best Practices เพื่อลด Conflicts

### 1. Sync บ่อยๆ

```bash
# ทุกเช้าก่อนเริ่มทำงาน
git checkout develop
git pull origin develop
git checkout feature/my-feature
git merge develop

# หรือ rebase
git rebase develop
```

### 2. เลือก Merge Strategy ที่เหมาะสม

```bash
# For long-running features: rebase
git rebase develop

# For collaboration branches: merge
git merge --no-ff develop

# For simple updates: fast-forward
git merge --ff-only develop
```

### 3. แบ่งงานให้ชัดเจน

```bash
# ใช้ CODEOWNERS
# .github/CODEOWNERS
/src/frontend/ @frontend-team
/src/backend/ @backend-team
/src/shared/ @frontend-team @backend-team

# แต่ละทีมทำงานในส่วนของตัวเอง
```

### 4. ใช้ Feature Flags

```javascript
// แทนที่จะ comment โค้ดเก่า
if (featureFlags.newFeature) {
  // โค้ดใหม่
} else {
  // โค้ดเก่า
}

// Merge เข้า main ได้ทันที
// ค่อยเปิด feature flag ทีหลัง
```

### 5. Code Review ก่อน Merge

```bash
# ตรวจสอบก่อน merge
git fetch origin
git diff origin/develop..feature/my-feature

# Review changes ให้ละเอียด
# ถ้ามีการเปลี่ยนแปลงที่ overlap กับ branch อื่น
# ให้ sync ก่อน
```

---

## Tools ช่วยแก้ Conflicts

### 1. VS Code (Built-in)
- แสดง conflict markers ชัดเจน
- Buttons: Accept Current / Accept Incoming / Accept Both
- Inline diff view

### 2. Git Kraken
- Visual merge conflict resolution
- 3-way merge view
- Syntax highlighting

### 3. Meld (Linux)
```bash
# Install
sudo apt install meld

# Configure
git config --global merge.tool meld

# Use
git mergetool
```

### 4. P4Merge (Cross-platform)
```bash
# Download from Perforce
# Configure
git config --global merge.tool p4merge
git config --global mergetool.p4merge.path "/usr/local/bin/p4merge"
```

### 5. Vim with fugitive
```bash
# In vim
:Gdiff
:Gwrite  # Accept current
:Gread   # Accept incoming
```

---

## Emergency: ยกเลิก Merge ที่ผิดพลาด

```bash
# ถ้ายัง commit ไม่ได้
git merge --abort

# ถ้า commit ไปแล้วแต่ยังไม่ push
git reset --hard HEAD~1

# ถ้า push ไปแล้ว (ระวัง!)
git revert -m 1 HEAD
git push origin develop

# หรือสร้าง branch ใหม่จาก commit ก่อนหน้า
git checkout -b fix-bad-merge HEAD~1
```

## Summary

**Key Takeaways:**
1. Conflict เป็นเรื่องปกติ ไม่ต้องกลัว
2. แก้ทีละไฟล์ ทดสอบบ่อยๆ
3. เข้าใจทั้งสอง changes ก่อนตัดสินใจ
4. ใช้ tools ช่วยถ้า conflict ซับซ้อน
5. ป้องกันดีกว่าแก้ - sync บ่อยๆ
6. ถ้าไม่แน่ใจ ถามเจ้าของโค้ด
