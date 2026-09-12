## 🔁 **Git Commands – Differences Table**

| **Command**  | **Purpose**                 | **Works On**         | **Key Difference / When to Use**        |
| ------------ | --------------------------- | -------------------- | --------------------------------------- |
| `git clone`  | Copy a remote repository    | Remote → Local       | Creates a new local repo with history   |
| `git init`   | Initialize empty repository | Local                | Used when starting from scratch         |
| `git status` | Show repo status            | Working tree         | Shows modified, staged, untracked files |
| `git add`    | Stage files                 | Working → Staging    | Prepares files for commit               |
| `git commit` | Save snapshot               | Staging → Local repo | Creates a commit with message           |
| `git push`   | Upload commits              | Local → Remote       | Shares changes with team                |
| `git pull`   | Fetch + merge               | Remote → Local       | Updates local branch                    |
| `git fetch`  | Download changes only       | Remote → Local       | Does **not** merge automatically        |
| `git merge`  | Combine branches            | Branch → Branch      | Preserves commit history                |
| `git rebase` | Reapply commits             | Branch → Branch      | Creates linear history                  |

---

## 🌿 **Branching Commands**

| **Command**           | **Purpose**          | **Key Difference**     |
| --------------------- | -------------------- | ---------------------- |
| `git branch`          | List/create branches | Does not switch branch |
| `git checkout branch` | Switch branch        | Older method           |
| `git switch branch`   | Switch branch        | Safer & newer          |
| `git checkout -b dev` | Create + switch      | One-step operation     |
| `git switch -c dev`   | Create + switch      | Modern alternative     |

---

## 🔄 **Undo & Fix Commands**

| **Command**                | **Use Case**             | **Effect**             |
| -------------------------- | ------------------------ | ---------------------- |
| `git restore file`         | Discard local changes    | Working directory only |
| `git reset file`           | Unstage file             | Staging → Working      |
| `git reset --soft HEAD~1`  | Undo commit              | Keeps changes staged   |
| `git reset --mixed HEAD~1` | Undo commit              | Keeps changes unstaged |
| `git reset --hard HEAD~1`  | Remove commit completely | ⚠️ Deletes changes     |
| `git revert <commit>`      | Undo safely              | Creates new commit     |

---

## 🔍 **Logs & History**

| **Command**         | **Purpose**      | **Best Use Case**  |
| ------------------- | ---------------- | ------------------ |
| `git log`           | Full history     | Deep inspection    |
| `git log --oneline` | Short history    | Quick view         |
| `git show`          | Commit details   | Inspect one commit |
| `git diff`          | File differences | Before commit      |
| `git blame file`    | Who changed what | Debugging          |

---

## 🧹 **Cleanup Commands**

| **Command**     | **Purpose**            | **Notes**            |
| --------------- | ---------------------- | -------------------- |
| `git clean -n`  | Preview delete         | Safe check           |
| `git clean -f`  | Delete untracked files | ⚠️ Permanent         |
| `git rm file`   | Delete tracked file    | Staged automatically |
| `git stash`     | Save work temporarily  | Useful before pull   |
| `git stash pop` | Restore stashed work   | Removes stash        |

---

## 🔐 **Remote Commands**

| **Command**                 | **Purpose**  | **Notes**              |
| --------------------------- | ------------ | ---------------------- |
| `git remote -v`             | Show remotes | Verify URLs            |
| `git remote add origin url` | Add remote   | First-time setup       |
| `git push -u origin main`   | Set upstream | Simplifies future push |

---

## 💡 **Interview Tip**

> **Use `git revert` for shared branches, `git reset` for local mistakes**
> **Use `git fetch` before risky merges**


# 📑 **Git Advanced Commands — Complete Practice Guide (Mini-Lab)**

> 🎯 **Goal**
> Master **stash, rebase, cherry-pick, reset, reflog, tags, clean, blame, diff, bisect, squash merge**
> — exactly how **senior DevOps engineers use Git daily**.

---

## 🧠 Git Advanced Architecture Refresher

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/1%2AXurNAi3h2jpD67Pq2OgmrQ%402x.png)

![Image](https://wac-cdn.atlassian.com/dam/jcr%3A4e576671-1b7f-43db-afb5-cf8db8df8e4a/01%20What%20is%20git%20rebase.svg?cdnVersion=3145)

![Image](https://i0.wp.com/css-tricks.com/wp-content/uploads/2021/11/pasted-image-0-8.png?resize=1600%2C554\&ssl=1)

### 🔁 Conceptual Flow

```
Working Directory
   ↓ (git add)
Staging Area
   ↓ (git commit)
Local Repository
   ↓ (push)
Remote Repository
```

Advanced commands **manipulate history**, not just files.

---

# 📦 LAB SETUP — Create Practice Repository

### 🎯 Objective

Create a **safe sandbox** to experiment without fear.

```bash
mkdir git-advanced-practice
cd git-advanced-practice
git init
```

### Create base file

```bash
echo "Line 1" > file.txt
git add file.txt
git commit -m "Initial commit"
```

✅ **Outcome**

* Repository initialized
* One clean commit exists

---

# 📌 1️⃣ Git Stash — Temporarily Save Uncommitted Changes

### 💡 Why Git Stash?

When:

* You’re coding ✍️
* Suddenly asked to fix prod 🔥
* You **don’t want to commit half-done work**

---

### 🧪 Step-by-Step Practice

```bash
echo "Temporary work" >> file.txt
git status
```

📌 File is **modified but not committed**

---

### Save work to stash

```bash
git stash
```

✔️ Working directory becomes clean

---

### List stashes

```bash
git stash list
```

---

### Inspect stash

```bash
git stash show
git stash show -p stash@{0}
```

---

### Restore stash

```bash
git stash apply
# OR
git stash pop
```

📌 `pop` = apply + delete
📌 `apply` = apply only

---

### Advanced Stash Commands (VERY IMPORTANT)

```bash
nano temp.txt
git stash save "temp experiment"

git stash -u      # include untracked files
git stash -a      # include ignored files
git stash clear   # delete all stashes
```

---

### Create a branch from stash (real-world lifesaver)

```bash
git stash branch feature-from-stash stash@{0}
```

🧠 **Use case**: Turn half-done work into a proper feature branch

---

# 📌 2️⃣ Git Rebase — Rewrite History Cleanly

![Image](https://wac-cdn.atlassian.com/dam/jcr%3A1896adb1-5d49-419a-9b50-3a36adac186c/09.svg?cdnVersion=3140)

![Image](https://wac-cdn.atlassian.com/dam/jcr%3A4e576671-1b7f-43db-afb5-cf8db8df8e4a/01%20What%20is%20git%20rebase.svg?cdnVersion=3124)

### 💡 Why Rebase?

* Clean linear history
* Avoid noisy merge commits
* Mandatory before PR merge in many orgs

---

### 🧪 Practice Scenario

#### Create feature branch

```bash
git checkout -b feature
echo "Feature line" >> file.txt
git add file.txt
git commit -m "Feature commit"
```

---

#### Update main branch

```bash
git checkout main
echo "Main line" >> file.txt
git add file.txt
git commit -m "Main commit"
```

---

#### Rebase feature onto main

```bash
git checkout feature
git rebase main
```

📌 Git reapplies feature commits **on top of main**

---

### Verify history

```bash
git log --oneline --graph --all
```

✅ Linear, professional history

⚠️ **Golden Rule**

> ❌ Never rebase shared branches (`main`, `dev`)

---

# 📌 3️⃣ Git Cherry-Pick — Copy a Specific Commit

### 💡 Real DevOps Use Case

* Hotfix applied in `dev`
* Need **only that fix** in `main`

---

```bash
git log --oneline
git cherry-pick <commit-hash>
```

✔️ Only selected commit is applied
❌ No full branch merge

---

# 📌 4️⃣ Git Reset & Reflog — Undo Mistakes Safely

### 🧪 Create a test commit

```bash
echo "Test line" >> file.txt
git add file.txt
git commit -m "Test commit"
```

---

### Reset hard (dangerous)

```bash
git reset --hard HEAD~1
```

😱 Commit disappears from log

---

### Recover using reflog

```bash
git reflog
```

```bash
git checkout <commit-hash-from-reflog>
```

🧠 **Reflog = Git black box recorder**

---

# 📌 5️⃣ Git Tag — Versioning & Releases

### 💡 Why Tags?

* Mark releases (`v1.0`, `v2.1`)
* CI/CD uses tags for deployments

---

```bash
git tag v1.0
git tag -a v1.1 -m "Version 1.1 Release"
git tag
```

Push tags:

```bash
git push origin --tags
```

---

# 📌 6️⃣ Git Clean — Remove Junk Files

### ⚠️ Dangerous but useful

```bash
touch temp.log
git status
```

Dry run:

```bash
git clean -nd
```

Delete:

```bash
git clean -fd
```

---

# 📌 7️⃣ Git Blame — Who Changed This Line?

```bash
git blame file.txt
```

🧠 Used in:

* Debugging
* Audits
* Incident analysis

---

# 📌 8️⃣ Git Diff — Compare Branches

```bash
git diff main..feature
```

Shows:

* Line-by-line differences
* What will change after merge

---

# 📌 9️⃣ Git Bisect — Find Bug-Introducing Commit

![Image](https://edrawcloudpublicus.s3.amazonaws.com/work/1905656/2022-3-23/1647997801/main.png)

![Image](https://belev.dev/static/3f8cc369ca0c0b0e8c8c1bb864becd34/a6c62/git-bisect.jpg)

### 💡 Why Bisect?

Binary search through commits
➡️ Finds bug in **minutes instead of hours**

---

```bash
git bisect start
git bisect bad
git bisect good <known-good-commit>
```

Test each checkout:

```bash
git bisect good
# or
git bisect bad
```

Finish:

```bash
git bisect reset
```

---

# 📌 🔟 Squash Merge — Clean Production History

### 💡 Use Case

Multiple messy feature commits → **1 clean commit in main**

```bash
git checkout main
git merge --squash feature
git commit -m "Merged feature as one commit"
```

---

# 📊 Visualize Everything

```bash
git log --oneline --graph --all
```

---

# 📁 Final Directory Structure

```
git-advanced-practice/
├── file.txt
└── .git/
```

---

# 🧪 BONUS — Run Everything as a Script

### `git-practice.sh`

```bash
#!/bin/bash

mkdir git-advanced-practice
cd git-advanced-practice || exit
git init

echo "Line 1" > file.txt
git add file.txt
git commit -m "Initial commit"

echo "Temporary work" >> file.txt
git stash
git stash pop

git checkout -b feature
echo "Feature line" >> file.txt
git add file.txt
git commit -m "Feature commit"

git checkout main
echo "Main line" >> file.txt
git add file.txt
git commit -m "Main commit"

git checkout feature
git rebase main

git log --oneline --graph --all
```

```bash
chmod +x git-practice.sh
./git-practice.sh
```

---
Below is a **production-focused Git command set**—the commands you actually rely on when working with **live repositories, CI/CD pipelines, hotfixes, and team workflows**.

---

## 🔐 Repository Safety & Configuration (Production Basics)

```bash
git status
git config --global user.name "Your Name"
git config --global user.email "you@email.com"
git config --global pull.rebase true
git config --global core.autocrlf input
```

✅ Prevents accidental commits, merge chaos, and environment issues.

---

## 🌱 Branching & Release Control (Very Important)

```bash
git branch
git branch -a
git checkout main
git checkout -b feature/login
git switch main
git switch -c hotfix/payment-bug
git branch -d feature/login
```

**Production rule:**

* `main / master` → production
* `develop` → staging
* `feature/*` → development
* `hotfix/*` → urgent prod fixes

---

## ⬇️ Syncing with Remote (Daily Production Use)

```bash
git fetch origin
git pull origin main
git pull --rebase origin develop
```

✅ `fetch` first → inspect → then merge/rebase safely.

---

## 📦 Commit Like a Professional

```bash
git add .
git add file.txt
git commit -m "fix: resolve null pointer in auth service"
git commit --amend
```

**Best practice**

* Use meaningful messages
* Avoid multiple unrelated changes in one commit

---

## 🔁 Merge & Rebase (Controlled Production Flow)

```bash
git merge develop
git merge --no-ff feature/login
git rebase develop
git rebase -i HEAD~3
```

✔ `merge` → production/stable
✔ `rebase` → clean feature history

---

## 🚀 Pushing Code (Production-Safe)

```bash
git push origin feature/login
git push origin main
git push --force-with-lease
```

⚠️ Never use `--force` on shared branches
✔ `--force-with-lease` is safer

---

## 🏷️ Versioning & Releases (Critical for Prod)

```bash
git tag
git tag v1.0.0
git tag -a v1.1.0 -m "production release"
git push origin v1.1.0
git push --tags
```

Used for:

* Rollbacks
* Release tracking
* CI/CD deployments

---

## 🔍 Investigation & Debugging (Prod Incidents)

```bash
git log --oneline --graph --all
git log -p
git blame app.py
git diff
git diff main..develop
```

🔥 `git blame` is gold during outages.

---

## ⏪ Rollback & Recovery (MOST IMPORTANT)

```bash
git revert <commit-id>
git reset --soft HEAD~1
git reset --hard HEAD~1
git checkout <commit-id>
```

### Safe Production Rule:

✔ Use **`git revert`**
❌ Avoid `reset --hard` on shared branches

---

## 🧹 Clean & Maintenance

```bash
git stash
git stash apply
git stash drop
git clean -fd
```

Useful before:

* Hotfixes
* Branch switches
* Emergency debugging

---

## 🔐 Production Git Workflow (Recommended)

```bash
feature → develop → main → tag → deploy
```

Example:

```bash
git checkout develop
git merge feature/api
git checkout main
git merge develop
git tag v2.0.1
git push origin main --tags
```

---

## 🧠 Commands Every DevOps Engineer MUST Know

| Scenario             | Command              |
| -------------------- | -------------------- |
| Rollback prod        | `git revert`         |
| Check who broke code | `git blame`          |
| Clean history        | `git rebase -i`      |
| Safe force push      | `--force-with-lease` |
| Emergency fix        | `hotfix/*` branch    |
| Release deploy       | `git tag`            |

---



## 🎯 What You’ve Achieved

✅ Real Git internals understanding
✅ Senior-level commands practiced
✅ CI/CD-ready Git workflows
✅ Interview + production confidence


