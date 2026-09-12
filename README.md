# Git Advanced 

## 1. Overall Git Architecture

```text
                           DEVELOPER
                               │
                               ▼
                     ┌───────────────────┐
                     │ Working Directory │
                     └─────────┬─────────┘
                               │ git add
                               ▼
                     ┌───────────────────┐
                     │   Staging Area    │
                     │      (Index)      │
                     └─────────┬─────────┘
                               │ git commit
                               ▼
                     ┌───────────────────┐
                     │ Local Repository  │
                     └─────────┬─────────┘
                               │ git push
                               ▼
                     ┌───────────────────┐
                     │ Remote Repository │
                     │ GitHub / GitLab / │
                     │   Azure DevOps    │
                     └───────────────────┘
```

This should be the starting diagram because `restore`, `reset`, `stash`, branching, and commits make much more sense once learners understand these three local states.

---

# 2. Git Branches

**Definition:** A branch is a named reference to a commit that provides an independent line of development.

**Use case:** Create isolated development areas for features, bug fixes, releases, and hotfixes.

```text
                        feature/login
                             │
                             C ─── D
                            /
A ─────── B ───────────────+
│
main
```

Typical real-world structure:

```text
main
 │
 ├──────── feature/login
 │
 ├──────── feature/payment
 │
 └──────── hotfix/payment-bug
```

Practice:

```bash
git switch main

git switch -c feature/login

echo "Login Feature" > login.txt

git add login.txt
git commit -m "feat: add login feature"

git log --oneline --graph --all --decorate
```

Your existing notes correctly prefer `git switch` as the modern branch-switching command while retaining `checkout` for awareness. 

---

# 3. Git Merge

**Definition:** Merge integrates histories from branches. Depending on topology/options, it can fast-forward or create a merge commit.

**Use case:** A tested feature needs to be integrated into `main` or another integration branch.

### Before

```text
              C ─── D    feature/login
             /
A ─── B ─── E
            │
           main
```

### Merge commit case

```text
              C ─── D
             /       \
A ─── B ─── E ─────── M
                       │
                      main
```

Practice:

```bash
git switch main
git merge feature/login
```

Visualize:

```bash
git log --oneline --graph --all --decorate
```

**Important:** Don't teach that every merge creates `M`; a fast-forward merge may simply advance the branch pointer.

---

# 4. Git Rebase

**Definition:** Rebase reapplies a series of commits onto a different base. The replayed commits normally receive new commit IDs. ([Git][1])

**Use case:** Bring your feature branch on top of the latest `main` while maintaining a linear history.

### Before

```text
             C ─── D       feature
            /
A ─── B ─── E ─── F
                │
               main
```

Run from feature:

```bash
git switch feature
git rebase main
```

### After

```text
A ─── B ─── E ─── F ─── C' ─── D'
                  │             │
                 main         feature
```

The crucial concept is:

```text
C  → C'
D  → D'

same logical changes
different commit identities
```

Git's documentation describes rebase as collecting the relevant commits and replaying them one-by-one on the new upstream, conceptually similar to cherry-picking each one. ([Git][1])

### Rebase Conflict Flow

```text
git rebase main
       │
       ▼
   Conflict?
    /      \
   No      Yes
   │        │
   ▼        ▼
 Done   Edit files
            │
            ▼
       git add <file>
            │
            ▼
   git rebase --continue
```

Other controls:

```bash
git rebase --continue
git rebase --abort
git rebase --skip
```

These are the official conflict-control paths for a rebase. ([Git][2])

---

# 5. Merge vs Rebase

This is one of the most important comparison diagrams in the module.

### Merge

```text
       C ─── D
      /       \
A ─── B ─── E ─── M
```

### Rebase

```text
A ─── B ─── E ─── C' ─── D'
```

| Feature                        | Merge                     | Rebase                                   |
| ------------------------------ | ------------------------- | ---------------------------------------- |
| Main goal                      | Integrate histories       | Replay commits on new base               |
| History                        | Preserves branch topology | Usually linearizes history               |
| Commit IDs rewritten           | Existing commits remain   | Replayed commits get new IDs             |
| Merge commit                   | Possible                  | Usually no merge commit in simple rebase |
| Shared history                 | Generally safer           | Avoid rewriting published/shared work    |
| Feature branch synchronization | Good                      | Excellent for clean feature history      |

Your source already frames merge as history-preserving and rebase as linear-history oriented. 

---

# 6. Git Cherry-Pick

**Definition:** Cherry-pick applies the changes introduced by selected commit(s) onto the current branch, creating new commit(s).

**Use case:** A hotfix exists on another branch, but you don't want the entire branch.

```text
                 C ─── D ─── E     develop
                /
A ─── B ───────+
│
main
```

Need only `D`:

```bash
git switch main
git cherry-pick <D-commit-id>
```

Result:

```text
                 C ─── D ─── E
                /
A ─── B ─────── D'
               │
              main
```

```text
D  ──changes──► D'
```

This is particularly suitable for **targeted hotfix/backport scenarios**, which matches your existing material. 

---

# 7. Conflict Resolution

**Definition:** A conflict occurs when Git cannot safely combine competing changes automatically.

### Typical Scenario

```text
                     config.txt
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
             main                feature
              │                     │
     Environment=Staging   Environment=Development
              │                     │
              └──────────┬──────────┘
                         ▼
                     CONFLICT
                         │
                         ▼
                  Developer decides
                         │
                         ▼
                      git add
                         │
                         ▼
                 Continue / Commit
```

Typical markers:

```text
<<<<<<< HEAD
Environment=Staging
=======
Environment=Development
>>>>>>> feature/config
```

Resolve the file and stage it:

```bash
git add config.txt
```

For merge:

```bash
git commit
```

For rebase:

```bash
git rebase --continue
```

---

# 8. Git Stash

**Definition:** Stash records working-directory/index changes temporarily so you can work from a cleaner state.

**Real-world use case:** You're halfway through feature development when an urgent production issue arrives.

```text
Feature Development
        │
        │ unfinished
        ▼
Working Directory
        │
        │ git stash
        ▼
┌──────────────────┐
│   Stash Stack    │
│ stash@{0}        │
│ stash@{1}        │
└──────────────────┘
        │
        ▼
Clean working state
        │
        ▼
Work on Hotfix
        │
        ▼
git stash pop
        │
        ▼
Continue Feature
```

Practice:

```bash
git stash push -m "login feature WIP"

git stash list

git stash show -p stash@{0}

git stash pop
```

Your notes correctly distinguish `apply` from `pop`: `apply` reapplies without removing the stash entry, while successful `pop` applies and removes it. 

---

# 9. Git Reset

**Definition:** Reset changes where the current branch/HEAD points and, depending on mode, also updates the index and working tree.

### Starting State

```text
A ─── B ─── C
            ▲
            │
       HEAD / main
```

Run:

```bash
git reset HEAD~1
```

Conceptually:

```text
A ─── B ─── C
      ▲
      │
 HEAD / main
```

The key difference is what happens to `C`'s changes.

### Reset Modes

```text
                   git reset HEAD~1
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
        --soft          --mixed           --hard
          │                │                │
          ▼                ▼                ▼
      Keep staged      Keep changes      Discard corresponding
                         unstaged         tracked changes
```

| Mode      | HEAD moves | Index                   | Working tree     | Typical use         |
| --------- | ---------: | ----------------------- | ---------------- | ------------------- |
| `--soft`  |        Yes | Changes retained staged | Retained         | Rebuild a commit    |
| `--mixed` |        Yes | Reset                   | Changes retained | Re-stage/rework     |
| `--hard`  |        Yes | Reset                   | Reset            | Discard local state |

Your notes correctly flag `reset --hard` as dangerous. 

---

# 10. Git Revert

**Definition:** Revert records a new commit that reverses changes introduced by an earlier commit.

**Use case:** A problematic commit has already reached a shared branch.

### Before

```text
A ─── B ─── C
            │
         Bad change
```

### Revert

```bash
git revert <C-commit-id>
```

### After

```text
A ─── B ─── C ─── D
            │     │
           Bad   Revert C
```

Notice:

```text
C IS STILL IN HISTORY
```

That's why revert is normally the clearer choice for undoing changes on shared history. Your notes already emphasize the same production rule. 

---

# 11. Git Restore

**Definition:** Restore changes file contents in the working tree and/or restores the index from another source; it does not move the branch pointer like reset.

### Discard Working-Tree Change

```text
Repository / Index
        │
        │ git restore file.txt
        ▼
Working Directory
```

```bash
git restore README.md
```

### Unstage

```text
             Staging Area
                  │
                  │ git restore --staged
                  ▼
        file no longer staged

Working-tree modification remains
```

```bash
git restore --staged README.md
```

---

# 12. Reset vs Revert vs Restore

This should be your **main rollback chart**.

```text
                         SOMETHING WENT WRONG
                                  │
                  ┌───────────────┼───────────────┐
                  │               │               │
             Local History    Shared History    File State
                  │               │               │
                  ▼               ▼               ▼
                RESET           REVERT          RESTORE
                  │               │               │
             Move branch      New undo        Restore /
              pointer          commit          unstage file
```

| Question                        | Reset                 | Revert         | Restore     |
| ------------------------------- | --------------------- | -------------- | ----------- |
| Operates mainly on              | Branch/HEAD + state   | Commit changes | Files/index |
| Moves branch pointer            | Yes                   | No             | No          |
| Creates new commit              | No                    | Yes            | No          |
| Rewrites visible branch history | Can                   | No             | No          |
| Good for local mistake          | **Yes**               | Possible       | File-level  |
| Good for shared/pushed commit   | Usually avoid         | **Yes**        | No          |
| Discard file modification       | Not the clearest tool | No             | **Yes**     |
| Unstage file                    | Possible              | No             | **Yes**     |

---

# 13. Complete Decision Flow

```text
                       WHAT DO YOU NEED?
                              │
        ┌─────────────────────┼──────────────────────┐
        │                     │                      │
  Develop separately?   Integrate changes?      Undo something?
        │                     │                      │
        ▼                     ▼                      ▼
      BRANCH          ┌───────┼────────┐       Is it pushed/
                      │       │        │        shared history?
                    Whole   Clean    Single           │
                    Branch  History  Commit       ┌───┴───┐
                      │       │        │          │       │
                    MERGE   REBASE  CHERRY-PICK  No      Yes
                                                   │       │
                                                   ▼       ▼
                                                 RESET   REVERT

Unfinished work?
      │
      ▼
    STASH

Wrong file change / wrong staging?
      │
      ▼
   RESTORE
```

# 14. Recommended Lab Flow

For the current module, I would teach it in this order:

```text
1. Repository / Working Tree / Index
                    ↓
2. Branch
                    ↓
3. Merge
                    ↓
4. Rebase
                    ↓
5. Merge vs Rebase
                    ↓
6. Cherry-Pick
                    ↓
7. Conflict Resolution
                    ↓
8. Stash
                    ↓
9. Reset
                    ↓
10. Revert
                    ↓
11. Restore
                    ↓
12. Reset vs Revert vs Restore
                    ↓
13. Final Real-World Challenge
```

I would **not mix `reflog`, `bisect`, `blame`, tags, `clean`, interactive rebase, and squash merge into this session**. They are useful and are already present in your source material—for example, your notes cover reflog recovery, tags, clean, blame, bisect, and squash merge.    They fit better as **Git Advanced – Level 2**, while this module stays focused on **branching + integration + conflict handling + rollback/recovery**.

[1]: https://git-scm.com/docs/git-rebase?utm_source=chatgpt.com "Git - git-rebase Documentation"
[2]: https://git-scm.com/docs/git-rebase/2.53.0.html?utm_source=chatgpt.com "Git - git-rebase Documentation"
