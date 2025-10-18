# Git Advanced Commands Cheat Sheet

## สร้างและเปลี่ยน Branches

```bash
# สร้าง branch ใหม่และ checkout
git checkout -b feature/new-feature

# สลับ branch
git checkout develop

# ดู branch ทั้งหมด
git branch -a

# ลบ branch (local)
git branch -d feature/old-feature

# ลบ branch (remote)
git push origin --delete feature/old-feature

# เปลี่ยนชื่อ branch
git branch -m old-name new-name
```

## Sync และ Update

```bash
# Update จาก remote
git fetch origin

# Pull และ merge
git pull origin main

# Pull และ rebase
git pull --rebase origin main

# Sync กับ upstream (forked repo)
git fetch upstream
git checkout main
git merge upstream/main
```

## Commit และ Staging

```bash
# Add ไฟล์
git add file.txt
git add .

# Commit
git commit -m "feat: add new feature"

# Amend commit ล่าสุด
git commit --amend
git commit --amend --no-edit

# Unstage ไฟล์
git reset HEAD file.txt

# Discard changes
git checkout -- file.txt
git restore file.txt
```

## Rebase และ History

```bash
# Rebase branch
git rebase main

# Interactive rebase (3 commits)
git rebase -i HEAD~3

# Continue rebase หลังแก้ conflict
git rebase --continue

# Abort rebase
git rebase --abort

# Squash commits
git rebase -i HEAD~3
# แล้วเปลี่ยน pick เป็น squash
```

## Stash (เก็บงาน temporary)

```bash
# Stash changes
git stash

# Stash with message
git stash save "WIP: working on feature"

# List stashes
git stash list

# Apply stash
git stash apply
git stash pop

# Apply specific stash
git stash apply stash@{2}

# Delete stash
git stash drop stash@{0}
git stash clear
```

## Cherry Pick

```bash
# Cherry pick commit
git cherry-pick abc123

# Cherry pick multiple commits
git cherry-pick abc123 def456

# Cherry pick without committing
git cherry-pick -n abc123
```

## Log และ History

```bash
# Log แบบกราฟ
git log --graph --oneline --all --decorate

# Log ของไฟล์เฉพาะ
git log --follow -- path/to/file

# Log ของ author
git log --author="John Doe"

# Log ในช่วงเวลา
git log --since="2 weeks ago"

# Show commit
git show abc123

# Diff between branches
git diff main..develop
```

## Reset และ Revert

```bash
# Soft reset (เก็บ changes ไว้)
git reset --soft HEAD~1

# Mixed reset (unstage changes)
git reset HEAD~1

# Hard reset (ลบ changes)
git reset --hard HEAD~1

# Revert commit (สร้าง commit ใหม่)
git revert abc123
```

## Tags

```bash
# สร้าง tag
git tag v1.0.0
git tag -a v1.0.0 -m "Version 1.0.0"

# List tags
git tag
git tag -l "v1.*"

# Push tags
git push origin v1.0.0
git push origin --tags

# Delete tag
git tag -d v1.0.0
git push origin --delete v1.0.0
```

## Remote

```bash
# Show remotes
git remote -v

# Add remote
git remote add upstream https://github.com/user/repo.git

# Remove remote
git remote remove upstream

# Rename remote
git remote rename origin new-origin

# Update remote URL
git remote set-url origin https://new-url.git
```

## Merge

```bash
# Merge branch
git merge feature/new-feature

# Merge with no fast-forward
git merge --no-ff feature/new-feature

# Abort merge
git merge --abort

# Merge strategy
git merge -X theirs feature/new-feature
git merge -X ours feature/new-feature
```

## Conflict Resolution

```bash
# Show conflicts
git status

# Use merge tool
git mergetool

# Take ours version
git checkout --ours file.txt

# Take theirs version
git checkout --theirs file.txt

# Mark as resolved
git add file.txt
```

## Advanced

```bash
# Bisect (หา bug)
git bisect start
git bisect bad
git bisect good abc123
git bisect reset

# Reflog (ดู history ทั้งหมด)
git reflog
git reset --hard HEAD@{2}

# Worktree (multiple working directories)
git worktree add ../project-feature feature/new
git worktree list
git worktree remove ../project-feature

# Blame (ดูว่าใครเขียน)
git blame file.txt
git blame -L 10,20 file.txt

# Clean untracked files
git clean -n  # dry run
git clean -fd # force delete
```

## GitHub CLI

```bash
# PR
gh pr list
gh pr create
gh pr view 123
gh pr checkout 123
gh pr review 123 --approve
gh pr merge 123 --squash

# Issues
gh issue list
gh issue create
gh issue view 123
gh issue close 123

# Repo
gh repo view
gh repo clone user/repo
gh repo fork
```

## Aliases (เพิ่มใน ~/.gitconfig)

```bash
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
    sync = !git fetch origin && git rebase origin/main
    cleanup = !git branch --merged | grep -v '*' | xargs git branch -d
```

## Config

```bash
# User info
git config --global user.name "Your Name"
git config --global user.email "email@example.com"

# Default editor
git config --global core.editor "code --wait"

# Default branch
git config --global init.defaultBranch main

# Auto CRLF
git config --global core.autocrlf input  # Mac/Linux
git config --global core.autocrlf true   # Windows

# Show config
git config --list
git config --global --list
```

## Performance

```bash
# Shallow clone
git clone --depth 1 https://github.com/user/repo.git

# Partial clone
git clone --filter=blob:none https://github.com/user/repo.git

# Maintenance
git gc
git prune

# File system monitor
git config core.fsmonitor true
git config core.untrackedCache true
```

## Emergency

```bash
# Undo last commit (not pushed)
git reset --soft HEAD~1

# Undo last push (DANGEROUS!)
git push --force-with-lease

# Restore deleted branch
git reflog
git checkout -b recovered-branch abc123

# Remove file from all history
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch path/to/file" \
  --prune-empty --tag-name-filter cat -- --all
```

## Tips

- ใช้ `git status` บ่อยๆ
- ใช้ `git diff` ก่อน commit
- ใช้ `--no-pager` ถ้าไม่ต้องการ pager
- ใช้ `--help` เพื่อดู manual
- Commit เล็กๆ บ่อยๆ
- Push เป็นประจำเป็น backup
- Sync กับ main branch ทุกวัน
