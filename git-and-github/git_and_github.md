# Git & GitHub


# 1. What is Git?
> Git is a distributed version control system that tracks file changes and enables collaborative development.

✔️ Distributed (DVCS)  
✔️ Fast & local operations  
✔️ Snapshot-based tracking  

---

# 2. Three Areas of Git
> Git manages changes across three main stages.

1. **Working Directory** → Where files are edited  
2. **Staging Area (Index)** → Prepared changes  
3. **Repository (.git)** → Permanent commit history  

---

# 3. Git File Lifecycle
> Files move through defined states during development.

Untracked → Modified → Staged → Committed

Check status:
```bash
git status
```

---

# 4. Basic Workflow

```bash
git init
git add .
git commit -m "message"
git push origin main
```

Daily workflow:
1. Pull latest changes  
2. Create branch  
3. Work & commit  
4. Push branch  
5. Open Pull Request  

---

# 5. Branching Concepts
> Branching allows parallel development without affecting the main codebase.

Create & switch:
```bash
git switch -b feature-login
```

List branches:
```bash
git branch
```

Delete branch:
```bash
git branch -d feature-login
```

---

# 6. Merge vs Rebase

### Merge
```bash
git merge feature
```
✔️ Preserves history  
✔️ Creates merge commit  

### Rebase
```bash
git rebase main
```
✔️ Cleaner linear history  
❗ Rewrites commit history  

👉 Never rebase shared public branches.

---

# 7. Branching Strategies

---

## Git Flow
> Structured branching model for release-based projects.

Branches:
- main → Production
- develop → Integration branch
- feature/* → New features
- release/* → Release preparation
- hotfix/* → Emergency fixes

Best for:
✔️ Large teams  
✔️ Versioned releases  
✔️ Enterprise projects  

---

## GitHub Flow
> Simple workflow focused on continuous deployment.

Flow:
1. Create branch from main  
2. Commit changes  
3. Open PR  
4. Review & merge  
5. Deploy  

Best for:
✔️ CI/CD environments  
✔️ Web applications  
✔️ Small-medium teams  

---

## Trunk-Based Development
> Developers commit small changes frequently to main (trunk).

- Short-lived branches  
- Feature flags  
- Continuous integration  

---

# 8. Remote Repository Concepts

Add remote:
```bash
git remote add origin https://github.com/user/repo.git
```

View remotes:
```bash
git remote -v
```

---

## Change Remote URL 

```bash
git remote set-url origin NEW_URL
```

Used when:
- Migrating repository
- Switching HTTPS to SSH
- Updating organization URL

---

# 9. Fetch vs Pull

| Command | Meaning |
|----------|----------|
| git fetch | Downloads changes only |
| git pull | Fetch + Merge |

Best practice:
```bash
git fetch
git merge origin/main
```

---

# 10. Reset vs Revert

### Reset (History rewrite)
```bash
git reset --hard HEAD~1
```

### Revert (Safe undo)
```bash
git revert <commit>
```

👉 Use revert for shared branches.

---

# 11. Stashing

Temporarily save work:

```bash
git stash
git stash pop
```

Useful when switching branches mid-work.

---

# 12. Authentication Methods 

---

## HTTPS + Personal Access Token (PAT)

> Password authentication is deprecated; use PAT instead.

Steps:
1. Generate PAT from GitHub  
2. Use HTTPS repo URL  
3. Use PAT instead of password  

 Avoid embedding your Personal Access Token (PAT) directly inside the remote URL.

---

## Do NOT Use (Insecure)

```bash
https://<TOKEN>@github.com/user/repo.git
```

This may expose your token in:
- `.git/config`
- Terminal history
- Shared screenshots or logs

---

## Recommended Approach

Set the remote without embedding the token:

```bash
git remote set-url origin https://github.com/user/repo.git
```

Then push normally:

```bash
git push
```

Git will prompt you for:

```
Username → your GitHub username
Password → your Personal Access Token (PAT)
```


---

## SSH Key-Based Authentication (Recommended)

> Uses public-private key pair for secure access.

### Generate SSH Key
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

### Add to SSH Agent
```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

### Add Public Key to GitHub
```bash
cat ~/.ssh/id_ed25519.pub
```

Use SSH remote:
```
git@github.com:user/repo.git
```

✔️ More secure  

---

# 13. Git Log & Diff

View history:
```bash
git log --oneline --graph --all
```

View changes:
```bash
git diff
git diff --staged
```

---

# 14. Cherry-Pick

> Apply specific commit to another branch.

```bash
git cherry-pick <commit-hash>
```

---

# 15. Git Best Practices

> Keep history clean, work safely, and collaborate efficiently.

✔️ Make small, meaningful commits  
✔️ Use feature branches (never commit directly to `main`)  
✔️ Pull before pushing  
✔️ Avoid rebasing shared/public branches  
✔️ Use `git revert` instead of `reset` for public history  
✔️ Never commit secrets or sensitive files  
✔️ Use `.gitignore` properly  
✔️ Prefer SSH over HTTPS + PAT  
✔️ Delete merged branches  
✔️ Protect the `main` branch with PR reviews