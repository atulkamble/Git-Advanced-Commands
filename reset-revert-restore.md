## Git Practice — Reset, Revert, Restore

These three commands are used to **undo changes**, but at different levels.

### 1. Create a Practice Repository

```bash
mkdir git-undo-practice
cd git-undo-practice

git init

git config user.name "Atul Kamble"
git config user.email "your-email@example.com"
```

Create the first file:

```bash
echo "Version 1" > app.txt

git add app.txt
git commit -m "Version 1"
```

Create two more commits:

```bash
echo "Version 2" >> app.txt
git add app.txt
git commit -m "Version 2"

echo "Version 3" >> app.txt
git add app.txt
git commit -m "Version 3"
```

Check history:

```bash
git log --oneline
```

Example:

```text
c3c3c3c Version 3
b2b2b2b Version 2
a1a1a1a Version 1
```

---

# 2. Git Restore

**Purpose:** Undo changes in files or remove files from the staging area.

Modify the file:

```bash
echo "Wrong change" >> app.txt

git status
git diff
cat app.txt
```

Discard the uncommitted change:

```bash
git restore app.txt
```

Verify:

```bash
cat app.txt
git status
```

### Restore a Staged File

Make another change:

```bash
echo "New feature" >> app.txt

git add app.txt
git status
```

Remove it from staging:

```bash
git restore --staged app.txt
```

The modification still exists, but it is no longer staged.

Discard it completely:

```bash
git restore app.txt
```

**Remember:**

```text
git restore file
        ↓
Discard working-directory changes

git restore --staged file
        ↓
Remove file from staging
```

---

# 3. Git Reset

**Purpose:** Move the current branch/HEAD to an earlier commit.

First create another commit:

```bash
echo "Version 4" >> app.txt
git add app.txt
git commit -m "Version 4"

git log --oneline
```

## Soft Reset

```bash
git reset --soft HEAD~1
```

Check:

```bash
git status
git log --oneline
```

Result:

```text
Commit removed
Changes remain staged
```

You can recommit:

```bash
git commit -m "Updated Version 4"
```

---

## Mixed Reset

Create another commit first if necessary:

```bash
echo "Version 5" >> app.txt
git add app.txt
git commit -m "Version 5"
```

Now:

```bash
git reset HEAD~1
```

or:

```bash
git reset --mixed HEAD~1
```

Result:

```text
Commit removed
Changes remain
Changes are NOT staged
```

Check:

```bash
git status
```

---

## Hard Reset

First commit your current changes:

```bash
git add .
git commit -m "Version 5"
```

Then:

```bash
git log --oneline
```

Reset:

```bash
git reset --hard HEAD~1
```

Result:

```text
Commit removed
Staged changes removed
Working-directory changes removed
```

**Warning:** `--hard` can destroy uncommitted work.

---

# 4. Git Revert

**Purpose:** Undo a commit by creating a **new commit**.

Check history:

```bash
git log --oneline
```

Suppose:

```text
c3c3c3c Version 3
b2b2b2b Version 2
a1a1a1a Version 1
```

Revert the latest commit:

```bash
git revert HEAD
```

Or revert a specific commit:

```bash
git revert c3c3c3c
```

Check:

```bash
git log --oneline
```

You may see:

```text
d4d4d4d Revert "Version 3"
c3c3c3c Version 3
b2b2b2b Version 2
a1a1a1a Version 1
```

The original commit is **not deleted**. Git creates a new commit that reverses it.

---

# Reset vs Revert vs Restore

| Command             | Main Purpose            | Changes History? | Typical Use                   |
| ------------------- | ----------------------- | ---------------: | ----------------------------- |
| `git restore`       | Undo file changes       |               No | Working directory/staging     |
| `git reset --soft`  | Remove commit           |              Yes | Keep changes staged           |
| `git reset --mixed` | Remove commit           |              Yes | Keep changes unstaged         |
| `git reset --hard`  | Remove commit + changes |              Yes | Completely discard local work |
| `git revert`        | Reverse a commit        |     No rewriting | Shared/pushed history         |

### Easy Way to Remember

```text
RESTORE
   ↓
Files

RESET
   ↓
HEAD / Local commit history

REVERT
   ↓
Create a new commit that undoes an old commit
```

For a **local commit that has not been pushed**, `reset` is commonly used. For a **commit already pushed/shared with others**, `revert` is generally safer because it preserves history.
