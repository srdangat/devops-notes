# Git Commands Cheat Sheet

| **Category** | **Command** | **Description / Use** | **Example** |
|--------------|------------|---------------------|------------|
| **Setup & Config** | `git init` | Initialize a new Git repository | `git init` |
|  | `git config` | Set username/email or view config | `git config --global user.name "Your Name"`<br>`git config --global user.email "your@email.com"`<br>`git config --global --list` |
| **Basic Workflow** | `git add` | Stage files for commit | `git add file.txt` |
|  | `git commit` | Commit staged changes with a message | `git commit -m "Commit message"` |
|  | `git status` | Check current repository status | `git status` |
|  | `git log` | View commit history | `git log` |
|  | `git log --oneline` | Compact one-line-per-commit log | `git log --oneline` |
|  | `git log --graph` | Visual commit graph | `git log --graph` |
|  | `git diff` | Show unstaged changes | `git diff` |
|  | `git diff --staged` | Show staged changes | `git diff --staged` |
|  | `git show <commit>` | Show details of a commit | `git show <commit-hash>` |
|  | `git restore` | Restore file to last committed state | `git restore file.txt` |
|  | `git rm --cached` | Remove file from tracking but keep it locally | `git rm --cached file.txt` |
| **Branching** | `git branch` | List or create branches | `git branch` |
|  | `git checkout <branch>` | Switch to branch | `git checkout feature-1` |
|  | `git switch <branch>` | Switch to branch (modern) | `git switch feature-1` |
|  | `git checkout -b <branch>` | Create and switch to new branch | `git checkout -b feature-2` |
|  | `git branch -d <branch>` | Delete a local branch | `git branch -d feature-2` |
| **Remote** | `git remote add origin` | Link local repo to remote | `git remote add origin <url>` |
|  | `git push` | Push local commits to remote | `git push -u origin main` |
|  | `git pull` | Fetch & merge changes from remote | `git pull origin main` |
|  | `git fetch` | Download changes from remote without merging | `git fetch origin` |
|  | `git clone` | Clone a remote repository | `git clone <url>` |
|  | Fork workflow | Keep fork in sync | `git remote add upstream <url>`<br>`git fetch upstream`<br>`git merge upstream/main`<br>`git push origin main` |
| **Merging & Rebasing** | `git merge <branch>` | Merge branch into current | `git merge feature-login` |
|  | `git merge --squash <branch>` | Combine all branch commits into one | `git merge --squash feature-profile`<br>`git commit -m "Add feature-profile"` |
|  | `git rebase <branch>` | Reapply commits on top of another branch | `git checkout feature-dashboard`<br>`git rebase main` |
| **Stash & Cherry Pick** | `git stash` | Save uncommitted changes | `git stash` |
|  | `git stash list` | List stashed changes | `git stash list` |
|  | `git stash pop` | Apply stashed changes & remove from stash | `git stash pop` |
|  | `git stash apply <stash>` | Apply stashed changes without removing | `git stash apply stash@{1}` |
|  | `git cherry-pick <commit>` | Apply specific commit from another branch | `git cherry-pick <commit-hash>` |
| **Reset & Revert** | `git reset --soft HEAD~1` | Undo commit, keep changes staged | `git reset --soft HEAD~1` |
|  | `git reset --mixed HEAD~1` | Undo commit, unstage changes | `git reset --mixed HEAD~1` |
|  | `git reset --hard HEAD~1` | Undo commit, discard all changes | `git reset --hard HEAD~1` |
|  | `git revert <commit>` | Undo a commit by creating a new commit | `git revert <commit-hash>` |
| **GH CLI Setup**               | `gh auth login`    | Authenticate GitHub CLI with your account    | `gh auth login`                                                                             |
|                                | `gh auth status`   | Check authentication status & active account | `gh auth status`                                                                            |
| **GH CLI Repo**                | `gh repo create`   | Create a new GitHub repository               | `gh repo create my-repo --public --description "Test repo" --confirm`                       |
|                                | `gh repo clone`    | Clone a repository using GitHub CLI          | `gh repo clone username/repo-name`                                                          |
|                                | `gh repo view`     | View repository details in terminal          | `gh repo view username/repo-name`                                                           |
|                                | `gh repo list`     | List all repositories for a user or org      | `gh repo list username`                                                                     |
|                                | `gh repo delete`   | Delete a repository (use with caution!)      | `gh repo delete my-repo --confirm`                                                          |
| **GH CLI Issue**               | `gh issue create`  | Create a new issue in a repository           | `gh issue create --title "Bug found" --body "Steps to reproduce..." --label "bug"`          |
|                                | `gh issue list`    | List all issues in a repo                    | `gh issue list --repo username/repo-name`                                                   |
|                                | `gh issue view`    | View a specific issue                        | `gh issue view 12 --repo username/repo-name`                                                |
|                                | `gh issue close`   | Close an issue                               | `gh issue close 12 --repo username/repo-name`                                               |
|                                | `gh issue comment` | Add a comment to an issue                    | `gh issue comment 12 --body "Checked automatically."`                                       |
| **GH CLI Pull Request**        | `gh pr create`     | Create a pull request from terminal          | `gh pr create --base main --head feature-branch --title "Add feature" --body "Description"` |
|                                | `gh pr list`       | List all open pull requests                  | `gh pr list --repo username/repo-name`                                                      |
|                                | `gh pr view`       | View details of a PR                         | `gh pr view 5 --repo username/repo-name`                                                    |
|                                | `gh pr merge`      | Merge a pull request                         | `gh pr merge 5 --merge`<br>`gh pr merge 5 --squash`<br>`gh pr merge 5 --rebase`             |
|                                | `gh pr review`     | Review someone else's PR                     | `gh pr review 5 --approve`<br>`gh pr review 5 --comment "Looks good"`                       |
| **GH CLI Actions / Workflows** | `gh run list`      | List workflow runs                           | `gh run list --repo username/repo-name`                                                     |
|                                | `gh run view`      | View a specific workflow run                 | `gh run view 12345 --repo username/repo-name`                                               |
|                                | `gh workflow list` | List all workflows in a repository           | `gh workflow list --repo username/repo-name`                                                |