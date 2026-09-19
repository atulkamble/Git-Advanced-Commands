# Git Practice — Reset, Revert & Restore

## Training Lab Guide

### Objective

In this lab, students will understand how to safely undo changes in Git using:

```text
git restore
git reset
git revert
```

These commands all help undo changes, but they operate at **different stages of the Git workflow**.

---

# 1. Git Workflow Architecture

```text
                         GIT WORKFLOW

 ┌──────────────────────┐
 │  Working Directory   │
 │                      │
 │  Create / Modify     │
 │  Files               │
 └──────────┬───────────┘
            │
            │ git add
            ▼
 ┌──────────────────────┐
 │    Staging Area      │
 │                      │
 │ Changes ready for    │
 │ commit               │
 └──────────┬───────────┘
            │
            │ git commit
            ▼
 ┌──────────────────────┐
 │   Local Repository   │
 │                      │
 │   Commit History     │
 └──────────┬───────────┘
            │
            │ git push
            ▼
 ┌──────────────────────┐
 │  Remote Repository   │
 │                      │
 │ GitHub / GitLab /    │
 │ Azure Repos          │
 └──────────────────────┘
```

### Undo Commands

```text
Working Directory
       │
       └──── git restore

Staging Area
       │
       └──── git restore --staged

Local Commit History
       │
       └──── git reset

Shared / Pushed History
       │
       └──── git revert
```

---

# 2. Points to Remember

| Command                | Purpose                                 |
| ---------------------- | --------------------------------------- |
| `git restore`          | Undo file changes                       |
| `git restore --staged` | Unstage a file                          |
| `git reset --soft`     | Remove commit but keep changes staged   |
| `git reset --mixed`    | Remove commit but keep changes unstaged |
| `git reset --hard`     | Remove commit and discard changes       |
| `git revert`           | Undo a commit by creating a new commit  |

Simple rule:

```text
File Change
    ↓
RESTORE

Local Commit
    ↓
RESET

Shared / Pushed Commit
    ↓
REVERT
```

---

# 3. Create Practice Repository

Create a new directory:

```bash
mkdir git-undo-practice
cd git-undo-practice
```

Initialize Git:

```bash
git init
```

Check:

```bash
git status
```

Configure Git identity if it is not already configured:

```bash
git config user.name "Atul Kamble"
git config user.email "your-email@example.com"
```

Verify:

```bash
git config user.name
git config user.email
```

---

# 4. Create Version 1

Create a file:

```bash
echo "Version 1" > app.txt
```

Check:

```bash
cat app.txt
```

Check Git status:

```bash
git status
```

Stage the file:

```bash
git add app.txt
```

Check:

```bash
git status
```

Commit:

```bash
git commit -m "Version 1"
```

Check history:

```bash
git log --oneline
```

Architecture:

```text
Working Directory
       │
       │ git add
       ▼
Staging Area
       │
       │ git commit
       ▼
┌─────────────────┐
│ C1 - Version 1  │ ← HEAD
└─────────────────┘
```

---

# 5. Create Version 2

Add another line:

```bash
echo "Version 2" >> app.txt
```

Check:

```bash
cat app.txt
```

Output:

```text
Version 1
Version 2
```

Stage and commit:

```bash
git add app.txt
git commit -m "Version 2"
```

Check:

```bash
git log --oneline
```

Architecture:

```text
C1
Version 1
   │
   ▼
C2
Version 2
   ↑
  HEAD
```

---

# 6. Create Version 3

```bash
echo "Version 3" >> app.txt

git add app.txt
git commit -m "Version 3"
```

Check:

```bash
git log --oneline
```

Example:

```text
c3c3c3c Version 3
b2b2b2b Version 2
a1a1a1a Version 1
```

Actual commit IDs will be different for every repository.

Architecture:

```text
[C1] Version 1
       │
       ▼
[C2] Version 2
       │
       ▼
[C3] Version 3
       ↑
      HEAD
```

---

# 7. Git Restore

## Purpose

`git restore` is mainly used to restore files in the working directory or remove files from the staging area.

It does **not normally change commit history**.

---

# 8. Practice — Restore Modified File

Add a wrong change:

```bash
echo "Wrong Change" >> app.txt
```

Check:

```bash
cat app.txt
```

Output:

```text
Version 1
Version 2
Version 3
Wrong Change
```

Check status:

```bash
git status
```

View the difference:

```bash
git diff
```

Architecture:

```text
Last Commit

Version 1
Version 2
Version 3

       │
       │ Modify
       ▼

Working Directory

Version 1
Version 2
Version 3
Wrong Change

       │
       │ git restore app.txt
       ▼

Version 1
Version 2
Version 3
```

Restore:

```bash
git restore app.txt
```

Verify:

```bash
cat app.txt
git status
```

The `Wrong Change` line is removed.

### Important

```text
git restore app.txt
```

means:

```text
Discard unstaged changes
        ↓
Restore file from index/staging version
```

Uncommitted changes discarded this way may be difficult or impossible to recover, so students should check `git diff` first.

---

# 9. Practice — Restore Staged File

Make another change:

```bash
echo "New Feature" >> app.txt
```

Stage:

```bash
git add app.txt
```

Check:

```bash
git status
```

Architecture:

```text
Working Directory
       │
       │ git add app.txt
       ▼
┌─────────────────────┐
│    Staging Area     │
│                     │
│ New Feature staged  │
└──────────┬──────────┘
           │
           │ git restore --staged app.txt
           ▼
┌─────────────────────┐
│ Working Directory   │
│                     │
│ New Feature remains │
│ but is NOT staged   │
└─────────────────────┘
```

Unstage:

```bash
git restore --staged app.txt
```

Check:

```bash
git status
```

The modification still exists.

Check:

```bash
cat app.txt
```

Now completely discard it:

```bash
git restore app.txt
```

Verify:

```bash
git status
cat app.txt
```

---

# 10. Restore Commands Summary

```text
git restore app.txt
        │
        ▼
Discard unstaged changes


git restore --staged app.txt
        │
        ▼
Remove changes from staging
        │
        ▼
Keep changes in working directory
```

---

# 11. Git Reset

## Purpose

`git reset` moves the current branch reference/`HEAD` to another commit.

Reset is commonly used to correct **local commit history before it has been shared**.

Three important modes:

```text
--soft

--mixed

--hard
```

---

# 12. Reset Architecture

Current history:

```text
C1 ───────── C2 ───────── C3
                          ↑
                         HEAD
```

Running:

```bash
git reset HEAD~1
```

moves the current branch back one commit:

```text
C1 ───────── C2
             ↑
            HEAD

             C3
             ↑
       no longer pointed
       to by the branch
```

`HEAD~1` means:

```text
HEAD     = Current commit

HEAD~1   = One commit before HEAD

HEAD~2   = Two commits before HEAD
```

---

# 13. Git Reset --soft

Create Version 4:

```bash
echo "Version 4" >> app.txt

git add app.txt
git commit -m "Version 4"
```

Check:

```bash
git log --oneline
```

Architecture:

```text
C1 ─── C2 ─── C3 ─── C4
                    ↑
                   HEAD
```

Run:

```bash
git reset --soft HEAD~1
```

Result:

```text
C1 ─── C2 ─── C3
              ↑
             HEAD

Version 4 changes
       │
       ▼
   Staging Area
```

Check:

```bash
git status
git log --oneline
```

### Result

```text
Commit removed from current branch history
Changes preserved
Changes remain staged
```

Check staged changes:

```bash
git diff --staged
```

Recommit:

```bash
git commit -m "Updated Version 4"
```

---

# 14. Git Reset --mixed

`--mixed` is the **default reset mode**.

Create Version 5:

```bash
echo "Version 5" >> app.txt

git add app.txt
git commit -m "Version 5"
```

Check:

```bash
git log --oneline
```

Run:

```bash
git reset --mixed HEAD~1
```

You can also simply use:

```bash
git reset HEAD~1
```

Both perform a mixed reset.

Architecture:

```text
BEFORE

C1 ─── C2 ─── C3 ─── C4 ─── C5
                           ↑
                          HEAD


git reset --mixed HEAD~1


AFTER

C1 ─── C2 ─── C3 ─── C4
                    ↑
                   HEAD

Version 5 changes
       │
       ▼
Working Directory

NOT STAGED
```

Check:

```bash
git status
```

Check changes:

```bash
git diff
```

### Result

```text
Commit removed from current branch history
Changes preserved
Changes are NOT staged
```

To prepare for the next exercise, recommit:

```bash
git add app.txt
git commit -m "Version 5"
```

---

# 15. Git Reset --hard

## Important

`--hard` is destructive.

It resets:

```text
Commit
+
Staging Area
+
Working Directory
```

Check history:

```bash
git log --oneline
```

Suppose:

```text
C1 ─── C2 ─── C3 ─── C4 ─── C5
                           ↑
                          HEAD
```

Run:

```bash
git reset --hard HEAD~1
```

Architecture:

```text
BEFORE

C1 ─── C2 ─── C3 ─── C4 ─── C5
                           ↑
                          HEAD


           RESET --hard
                 │
                 ▼


AFTER

C1 ─── C2 ─── C3 ─── C4
                    ↑
                   HEAD


C5 removed from current branch history

C5 changes discarded
```

Check:

```bash
git log --oneline
cat app.txt
git status
```

### Result

```text
Commit removed from current branch history
Staged changes reset
Working-directory tracked changes reset
```

### Warning

Before running:

```bash
git reset --hard
```

always check:

```bash
git status
git diff
git diff --staged
```

Do not use `--hard` casually on important work.

---

# 16. Reset Modes Comparison

```text
                     git reset
                         │
          ┌──────────────┼──────────────┐
          │              │              │
          ▼              ▼              ▼
       --soft          --mixed        --hard
          │              │              │
          ▼              ▼              ▼
 Move HEAD          Move HEAD       Move HEAD

 Keep staged        Unstage         Reset index
 changes            changes         and tracked
                                     worktree
```

| Reset Type | Commit    | Staging     | Working Changes         |
| ---------- | --------- | ----------- | ----------------------- |
| `--soft`   | Move HEAD | Keep staged | Keep                    |
| `--mixed`  | Move HEAD | Unstage     | Keep                    |
| `--hard`   | Move HEAD | Reset       | Discard tracked changes |

---

# 17. Git Revert

## Purpose

`git revert` reverses the changes introduced by a commit by creating a **new commit**.

Unlike `reset`, it does not rewrite the existing history.

This makes it suitable for commits that have already been pushed/shared.

---

# 18. Create a Commit for Revert Practice

At this stage, create a new commit:

```bash
echo "Bad Feature" >> app.txt

git add app.txt
git commit -m "Add bad feature"
```

Check:

```bash
git log --oneline
```

Example:

```text
e5e5e5e Add bad feature
d4d4d4d Updated Version 4
c3c3c3c Version 3
b2b2b2b Version 2
a1a1a1a Version 1
```

Check file:

```bash
cat app.txt
```

---

# 19. Revert Latest Commit

Run:

```bash
git revert HEAD
```

Git may open the configured text editor so you can confirm the revert commit message.

Save and close the editor.

Alternatively, for a simple lab where you want to accept the generated message without opening an editor:

```bash
git revert --no-edit HEAD
```

Check:

```bash
git log --oneline
```

Example:

```text
f6f6f6f Revert "Add bad feature"
e5e5e5e Add bad feature
d4d4d4d Updated Version 4
c3c3c3c Version 3
```

Architecture:

```text
BEFORE

C1 ─── C2 ─── C3 ─── C4 ─── C5
                           ↑
                          HEAD

C5 = Add bad feature


git revert HEAD


AFTER

C1 ─── C2 ─── C3 ─── C4 ─── C5 ─── C6
                                  ↑
                                 HEAD

C5 = Add bad feature

C6 = Revert "Add bad feature"
     reverses C5
```

The original commit still exists.

---

# 20. Revert Specific Commit

Find commit ID:

```bash
git log --oneline
```

Then:

```bash
git revert <commit-id>
```

Example:

```bash
git revert e5e5e5e
```

Do not copy the example hash blindly. Students should use the commit ID generated in their own repository.

---

# 21. Reset vs Revert

## Git Reset

```text
BEFORE

C1 ─── C2 ─── C3
              ↑
             HEAD


git reset --hard HEAD~1


AFTER

C1 ─── C2
       ↑
      HEAD

C3 is no longer part of
the current branch history.
```

## Git Revert

```text
BEFORE

C1 ─── C2 ─── C3
              ↑
             HEAD


git revert HEAD


AFTER

C1 ─── C2 ─── C3 ─── C4
                     ↑
                    HEAD

C3 = Original commit

C4 = New commit that
     reverses C3
```

Main difference:

```text
RESET
  ↓
Moves branch / HEAD
  ↓
Can rewrite visible branch history


REVERT
  ↓
Keeps old commit
  ↓
Creates new undo commit
  ↓
Preserves shared history
```

---

# 22. Restore vs Reset vs Revert

| Command                        | Main Purpose                          | History                       | Common Use                           |
| ------------------------------ | ------------------------------------- | ----------------------------- | ------------------------------------ |
| `git restore app.txt`          | Discard unstaged file changes         | No change                     | Wrong local file modification        |
| `git restore --staged app.txt` | Unstage changes                       | No change                     | Accidentally ran `git add`           |
| `git reset --soft HEAD~1`      | Move HEAD back                        | Rewrites local branch history | Redo last commit, keep staged        |
| `git reset --mixed HEAD~1`     | Move HEAD back                        | Rewrites local branch history | Redo last commit, keep unstaged      |
| `git reset --hard HEAD~1`      | Move HEAD and discard tracked changes | Rewrites local branch history | Completely discard local commit/work |
| `git revert HEAD`              | Reverse commit                        | Preserves history             | Undo shared/pushed commit            |

---

# 23. Complete Architecture

```text
                         GIT WORKFLOW

                  Create / Modify File
                           │
                           ▼
                ┌────────────────────┐
                │ Working Directory  │
                └─────────┬──────────┘
                          │
                 git add  │
                          ▼
                ┌────────────────────┐
                │    Staging Area    │
                └─────────┬──────────┘
                          │
              git commit  │
                          ▼
                ┌────────────────────┐
                │ Local Repository   │
                │ Commit History     │
                └─────────┬──────────┘
                          │
                git push  │
                          ▼
                ┌────────────────────┐
                │ Remote Repository  │
                │ GitHub / GitLab /  │
                │ Azure Repos        │
                └────────────────────┘


                    UNDO OPERATIONS

Modified file
     │
     └──────────────► git restore


Staged file
     │
     └──────────────► git restore --staged


Local commit
     │
     └──────────────► git reset


Shared / pushed commit
     │
     └──────────────► git revert
```

---

# 24. Easy Decision Flow

```text
                 I MADE A MISTAKE
                        │
                        ▼
                 Was it committed?
                   /          \
                 NO            YES
                 │              │
                 ▼              ▼
          Was it staged?    Was it shared/
             /    \           pushed?
           NO      YES         /    \
           │        │        NO      YES
           ▼        ▼        │        │
       restore   restore     ▼        ▼
        file     --staged   RESET   REVERT
```

Commands:

```bash
# Discard unstaged file changes
git restore app.txt

# Remove file from staging
git restore --staged app.txt

# Undo local commit and keep changes staged
git reset --soft HEAD~1

# Undo local commit and keep changes unstaged
git reset HEAD~1

# Undo local commit and discard tracked changes
git reset --hard HEAD~1

# Safely reverse latest shared commit
git revert HEAD
```

---

# 25. Useful Validation Commands

Students should frequently use these commands during the lab:

```bash
git status
```

Shows the current state of the working directory and staging area.

```bash
git log --oneline
```

Shows compact commit history.

```bash
git diff
```

Shows unstaged changes.

```bash
git diff --staged
```

Shows staged changes.

```bash
cat app.txt
```

Shows file content on Linux/macOS/Git Bash.

A useful graphical history command is:

```bash
git log --oneline --graph --decorate --all
```

Example:

```text
* f6f6f6f (HEAD -> main) Revert "Add bad feature"
* e5e5e5e Add bad feature
* d4d4d4d Updated Version 4
* c3c3c3c Version 3
* b2b2b2b Version 2
* a1a1a1a Version 1
```

---

# 26. Important Points for Students

1. **Restore works mainly with files.** Use it to discard working-directory changes or unstage changes.

2. **Reset moves the current branch/HEAD.** Use it carefully when correcting local commit history.

3. **`git reset --soft` keeps changes staged.**

4. **`git reset --mixed` keeps changes but unstages them.** `--mixed` is the default mode.

5. **`git reset --hard` discards tracked working-tree changes.** Check your work before using it.

6. **Revert does not delete the original commit.** It creates a new commit that reverses the selected commit.

7. **Prefer revert for shared history.** Resetting commits that others already use can cause collaboration problems.

8. **Commit IDs are unique.** The hashes shown in training examples will differ on each student's machine.

9. **Always inspect before undoing:**

```bash
git status
git log --oneline
git diff
git diff --staged
```

---

# 27. Final Cheat Sheet

```text
┌─────────────────────────────────────────────────────────┐
│                    GIT UNDO CHEAT SHEET                 │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Wrong file change                                      │
│       ↓                                                 │
│  git restore app.txt                                    │
│                                                         │
│  Accidentally staged                                    │
│       ↓                                                 │
│  git restore --staged app.txt                           │
│                                                         │
│  Undo local commit + keep staged                        │
│       ↓                                                 │
│  git reset --soft HEAD~1                                │
│                                                         │
│  Undo local commit + keep unstaged                      │
│       ↓                                                 │
│  git reset HEAD~1                                       │
│                                                         │
│  Undo local commit + discard tracked changes            │
│       ↓                                                 │
│  git reset --hard HEAD~1                                │
│                                                         │
│  Undo shared/pushed commit                              │
│       ↓                                                 │
│  git revert HEAD                                        │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### One-Line Memory Trick

```text
RESTORE = Files

RESET   = Local history / HEAD

REVERT  = Safe undo using a new commit
```

### Recommended Training Flow

```text
Create Repository
      ↓
Create Version 1
      ↓
Create Version 2
      ↓
Create Version 3
      ↓
Practice Restore
      ↓
Practice Restore --staged
      ↓
Practice Reset --soft
      ↓
Practice Reset --mixed
      ↓
Practice Reset --hard
      ↓
Create Bad Commit
      ↓
Practice Revert
      ↓
Compare Reset vs Revert
      ↓
Review git log --graph
```

This sequence keeps the lab **simple, repeatable, and internally consistent**, while letting students see the effect of each undo command directly.
