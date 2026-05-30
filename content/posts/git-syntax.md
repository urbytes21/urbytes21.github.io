---
author: "Phong Nguyen"
title: "Git Notes"
date: "2026-05-30"
description: "Git Notes"
tags: ["git"]   #tags search
FAcategories: ["syntax"]    #The category of the post, similar to tags but usually for broader classification.
FAseries: ["Themes Guide"]    #indicates that this post is part of a series of related posts
aliases: ["migrate-from-jekyl"]    #Alternative URLs or paths that can be used to access this post, useful for redirects from old posts or similar content.
ShowToc: true    # Determines whether to display the Table of Contents (TOC) for the post.
TocOpen: true    # Controls whether the TOC is expanded when the post is loaded. 
weight: 10    # The order in which the post appears in a list of posts. Lower numbers make the post appear earlier.
---

## Git Cheat Sheet

**`<commit>`** can be any Git reference that points to a commit

Local git config: `.git/config`
(See all possible config options man git-config)
Global git config: `~/.gitconfig`
Git ignore file : `.gitignore`
Local ignore rules (not committed): `.git/info/exclude`

```bash
# Start a new repo
$ git init 

# clone an existing repo
$ git clone <url>

# Add untracked file or unstaged changes
$ git add <file>

# Add all untracked files and unstaged changes
$ git add .

### Choose which parts of a file to stage
$ git add -p

# Delete file
$ git rm <file>

# Tell Git to forget about a file without deleting it
$ git rm --cached <file>

# Unstage one file
git reset <file>

# Unstage everything
git reset

# Check what you added
git status

# Make a commit (and open text editor to write message)
git commit

# Make a commit
git commit -m 'message'

# Commit all unstaged changes
git commit -am 'message'

# Make an empty commit for specific purpose
git commit --allow-empty -m 'message'

# Switch branches
$ git switch <name> 
$ git checkout <name>

# Create a new branch from the current branch
$ git switch -c <new-branch>
$ git checkout -b <new-branch>

# Create a new branch from another local branch
$ git switch -c <new-branch> <base-branch>    # prefer
$ git checkout -b <new-branch> <base-branch>

# Create a new branch from a remote branch
$ git switch -c <new-branch> origin/<remote-branch>   # prefer
$ git checkout -b <new-branch> origin/<remote-branch>

# List branches
$ git branch
$ git branch -r # remote branches

# Delete a branch
$ git branch -d <name>
$ git branch -D <name> # Force

# Diff all staged and unstaged changes
$ git diff HEAD

# Diff just staged changes
$ git diff --staged

# Diff just unstaged changes
$ git diff

# Save to file
$ git diff >> file

# Compare two refs and list changed files
git diff --name-only <branch> <commit> > files.txt

# Show diff between a commit and its parent
git show <commit>

# Diff two commits
git diff <commit> <commit>

# Diff one file since a commit
git diff <commit> <file>

# Show a summary of a diff
git diff <commit> --stat git show <commit> --stat

# Discard unstaged changes in one file
git restore <file>
git checkout -- <file>

# Discard all changes (staged and unstaged) in one file
git restore --staged --worktree <file>
git checkout HEAD -- <file>

# Delete all staged and unstaged changes
git reset --hard

# Force Delete untracked files/director
git clean -fd

# 'Stash' all staged and unstaged changes
git stash

# "Undo" the most recent commit
git reset HEAD^

# Squash the last 5 commits into one
git rebase -i HEAD~6

# Open the log history
git reflog
git reflog BRANCHNAME

# Move current branch back to <commit>
git reset --hard <commit>

# Recreate the branch using hash
git checkout -b <new-branch> <commit>

# Change a commit message / add a file
git commit --amend

# Show commit history
git log main

# Show compact commit history
git log --oneline main

# Show commit history as a branch graph
git log --graph main

# Show a compact graph of all branches
git log --oneline --graph --all --decorate

# Show every commit that modified a file
git log <file>

# Show every commit that modified a file, including before it was renamed
git log --follow <file>

# Find every commit that added or removed some text
git log -G "<text>"

# Show who last changed each line of a file
git blame <file>

# Cherry pick to copy one commit onto the current branch
git cherry-pick <commit>

# Replace the current version of a file with the version from another commit    
git restore --source <commit> <file>

# Add a Remote
git remote add <name> <url>

# Push the main branch to the remote origin
git push origin main

# Push the current branch to its remote "tracking branch"
git push

# Push a branch that you've never pushed before
git push -u origin <name>

# Force push
git push --force-with-lease # the remote has not changed 
git push --force

# Create a tag
$ git tag <tag>
$ git tag -a <tag> -m "<des>"

# Push tags
$ git push origin <tag>
$ git push --tags

# Fetch changes
git fetch origin main

# Fetch changes and then rebase your current branch
git pull --rebase

# Set a config option
git config user.name <user_name>
git config user.email <user_email>

# Set option globally
git config --global ...

# Mark a repository as safe (useful with Docker, WSL, shared folders, etc.)
$ git config --global --add safe.directory <path>

# Initialize and update submodules
git submodule update --init

# Run a single Git command without SSL certificate verification
git -c http.sslVerify=false <git-command>

# Set upstream branch
git branch --set-upstream-to=origin/<branch_name>

# Copy a file from another branch without switching branches
git checkout <branch> -- <file>
```

---

## Useful VSCode Extensions
- [Git Graph](https://marketplace.visualstudio.com/items?itemName=mhutchie.git-graph)
- [Git History](https://marketplace.visualstudio.com/items?itemName=donjayamanne.githistory)


---
## Useful Eclipse Plugins
- [EGit](https://projects.eclipse.org/projects/technology.egit)

---
TBD