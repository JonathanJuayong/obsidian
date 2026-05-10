Version Control System

# Git Cheat Sheet

## Setup & Configuration

```bash
git config --global user.name "Your Name"       # Set global username
git config --global user.email "you@email.com"  # Set global email
git config --global core.editor vim             # Set default editor
git config --list                               # List all config settings
```

---

## Creating & Cloning Repositories

```bash
git init                        # Initialize a new local repo
git init <directory>            # Init a repo in a specific folder
git clone <url>                 # Clone a remote repo
git clone <url> <directory>     # Clone into a specific folder
```

---

## Staging & Committing

```bash
git status                      # Show working tree status
git add <file>                  # Stage a specific file
git add .                       # Stage all changes
git add -p                      # Interactively stage chunks
git commit -m "message"         # Commit with a message
git commit -am "message"        # Stage tracked files and commit
git commit --amend              # Modify the last commit
```

---

## Branching

```bash
git branch                      # List local branches
git branch -a                   # List all branches (incl. remote)
git branch <name>               # Create a new branch
git branch -d <name>            # Delete a branch (safe)
git branch -D <name>            # Force delete a branch
git branch -m <old> <new>       # Rename a branch
git switch <name>               # Switch to a branch
git switch -c <name>            # Create and switch to a branch
git checkout <name>             # Switch to a branch (legacy)
git checkout -b <name>          # Create and switch (legacy)
```

---

## Merging & Rebasing

```bash
git merge <branch>              # Merge branch into current
git merge --no-ff <branch>      # Merge with a merge commit
git merge --abort               # Abort an in-progress merge
git rebase <branch>             # Rebase current onto branch
git rebase -i HEAD~<n>          # Interactive rebase last n commits
git rebase --abort              # Abort a rebase
git rebase --continue           # Continue after resolving conflicts
git cherry-pick <commit>        # Apply a specific commit
```

---

## Remote Repositories

```bash
git remote -v                           # List remotes
git remote add origin <url>             # Add a remote
git remote remove <name>                # Remove a remote
git remote rename <old> <new>           # Rename a remote
git fetch                               # Fetch all remotes
git fetch <remote>                      # Fetch a specific remote
git pull                                # Fetch and merge
git pull --rebase                       # Fetch and rebase
git push <remote> <branch>              # Push to remote
git push -u origin <branch>             # Push and set upstream
git push --force-with-lease             # Safer force push
git push origin --delete <branch>       # Delete a remote branch
```

---

## Viewing History & Diffs

```bash
git log                         # Show commit history
git log --oneline               # Compact one-line log
git log --oneline --graph       # Visual branch graph
git log -p                      # Show patches (diffs)
git log --author="Name"         # Filter by author
git log --since="2 weeks ago"   # Filter by date
git diff                        # Unstaged changes
git diff --staged               # Staged changes
git diff <branch1>..<branch2>   # Diff between branches
git show <commit>               # Show a specific commit
git blame <file>                # Show who changed each line
```

---

## Undoing Changes

```bash
git restore <file>              # Discard unstaged changes
git restore --staged <file>     # Unstage a file
git revert <commit>             # Create a revert commit
git reset HEAD~1                # Undo last commit (keep changes)
git reset --soft HEAD~1         # Undo commit, keep staged
git reset --hard HEAD~1         # Undo commit, discard changes
git clean -fd                   # Remove untracked files/dirs
```

---

## Stashing

```bash
git stash                       # Stash current changes
git stash push -m "message"     # Stash with a label
git stash list                  # List all stashes
git stash pop                   # Apply and remove latest stash
git stash apply stash@{n}       # Apply a specific stash
git stash drop stash@{n}        # Delete a specific stash
git stash clear                 # Remove all stashes
```

---

## Tags

```bash
git tag                         # List tags
git tag <name>                  # Create a lightweight tag
git tag -a <name> -m "msg"      # Create an annotated tag
git push origin <tag>           # Push a tag to remote
git push origin --tags          # Push all tags
git tag -d <name>               # Delete a local tag
git push origin --delete <tag>  # Delete a remote tag
```

---

## Searching

```bash
git grep "pattern"              # Search working directory
git log -S "string"             # Find commits that added/removed string
git log --all --grep="message"  # Search commit messages
```

---

## Submodules

```bash
git submodule add <url>             # Add a submodule
git submodule update --init         # Init and update submodules
git submodule update --remote       # Pull latest for submodules
git submodule foreach git pull      # Pull in all submodules
```

---

## Useful Shortcuts

|Alias|Command|
|---|---|
|`git st`|`git status`|
|`git co`|`git checkout`|
|`git lg`|`git log --oneline --graph --all`|

Set an alias:

```bash
git config --global alias.lg "log --oneline --graph --all"
```

---

## Common Workflows

### Feature Branch

```bash
git switch -c feature/my-feature    # Create feature branch
# ... make changes ...
git add . && git commit -m "feat"   # Commit
git switch main && git merge feature/my-feature  # Merge back
```

### Undo a Pushed Commit (safely)

```bash
git revert <commit-hash>            # Creates an undo commit
git push                            # Push the revert
```

### Sync a Fork

```bash
git remote add upstream <original-url>
git fetch upstream
git merge upstream/main
```