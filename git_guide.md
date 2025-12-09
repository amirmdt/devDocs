### Git Guide — From Basics to Advanced
A clear, corrected, improved, and expanded version of your original notes.
___________
## Table of Contents
- [Introduction](#introduction)
- [Installing Git](#installing-git)
- [Initializing a Repository](#initializing-a-repository)
- [Checking Status](#checking-status)
- [Staging and Committing](#staging-and-committing)
- [Viewing History](#viewing-history)
- [Comparing Changes (git diff)](#comparing-changes-git-diff)
- [Undoing Changes](#undoing-changes)
- [Branches](#branches)
- [Branches Diagram](#branches-diagram)
- [Merging Branches](#merging-branches)
- [Deleting Branches](#deleting-branches)
- [Removing Files (`git rm`)](#removing-files-git-rm)
- [Working With Remote Repositories](#working-with-remote-repositories)
- [Using SSH Keys With GitHub/GitLab](#using-ssh-keys-with-githubgitlab)
- [Handling Merge Conflicts](#handling-merge-conflicts)
- [Tags](#tags)
- [GPG Signing](#gpg-signing)
- [Git Blame](#git-blame)
- [Git Bisect](#git-bisect)
___________

# Introduction
Git is a distributed version control system that helps you track changes in your project, collaborate with others, and keep a full history of all updates.
___________

# Installing Git
Example for Rocky Linux:
```bash
sudo dnf install git
```
___________
# Initializing a Repository
```bash
cd /tmp
mkdir project
cd project
git init
```
This creates a hidden .git folder inside /tmp/project which allows Git to track everything inside this directory.
___________
# Checking Status
```bash
git status
```
Shows tracked/untracked files, staged changes, and your current branch.
___________
# Staging and Committing
Create a file:
```bash
echo "Hello" > index.html
git status

```
Git shows it as untracked.\
To stage it:
```bash
git add index.html
```
To commit:
```bash
git commit -m "Add index.html"
```
**First-time commit username/email error**\
If Git complains:
```bash
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
```
Adding multiple files
```bash
git add -A
git commit -m "Added page1 and edited index"
```
Or one-by-one:
```bash
git add index.html
git commit -m "Changed index"

git add page1.html
git commit -m "Added page1"
```
___________
# Viewing History
```bash
git log
```
___________
# Comparing Changes (git diff)
Compare working directory with last commit:
```bash
git diff HEAD
```
Compare staged changes:
```bash
git diff --staged
```
___________
# Undoing Changes
Unstage a file:
```bash
git reset page3.html
```
Discard local changes (return file to last commit):
```bash
git checkout -- page3.html
```
⚠️ This deletes your local modifications permanently.
___________
# Branches
Branches let multiple people work on different features without affecting `master` (or `main`).\
See all branches:
```bash
git branch
```
Create a branch:
```bash
git branch fixpages
```
Switch to a branch:
```bash
git checkout fixpages
```
Create + switch (recommended):
```
git checkout -b fixpages
```
___________
# Branches Diagram
```mathematica
master:   A ── B ── C
                  \
fixpages:          D ── E
```
___________
# Merging Branches
Switch to master:
```bash
git checkout master
```
Merge the branch:
```bash
git merge fixpages
```
Now master contains all commits from `fixpages`.\
**Important Tip**\
You will not see uncommitted work from another branch.\
You will see committed work only after merging.

___________
# Deleting Branches
```bash
git branch -d fixpages
```
___________
# Removing Files (`git rm`)
This removes the file from the working directory and Git history (after commit):
```bash
git rm filename
git commit -m "Remove filename"
```
___________
# Working With Remote Repositories
Clone an existing repository:
```bash
git clone <URL>
```
Add a remote to an existing local project:
```bash
git remote add origin <URL>
```
Check remotes:
```bash
git remote -v
```
Push your local branch:
```bash
git push origin master
```
Pull updates:
```bash
git pull origin master
```
Best practice:\
Always pull before working.
___________
# Using SSH Keys With GitHub/GitLab
GitHub removed password authentication (2021+).\
Use SSH keys:

Generate keys:
```bash
ssh-keygen -t rsa
```
Copy the public key and add it to:\
**GitHub → Settings → SSH and GPG Keys → New SSH Key**

Then update your remote URL:
```bash
git remote set-url origin <SSH_URL>
```
Now you can push:
```bash
git push origin master
```
___________
# Handling Merge Conflicts
If you modify a file locally and someone else modifies it on GitHub, Git will stop your pull and tell you there is a conflict.

You have 3 strategies:
___________
**Option 1: Merge (default)**
```bash
git config pull.rebase false
git pull origin master
```
Keeps both histories.
___________
**Option 2: Rebase**
```bash
git config pull.rebase true
git pull origin master
```
Rewrites your commits on top of remote history.\
Creates a clean history.
___________
**Option 3: Fast-forward only**
```bash
git config pull.ff only
git pull origin master
```
Fails if both sides changed.\
Good for strict workflows.
___________
# Tags
Tags are used to mark versions (e.g., `v1.0`, `v2.0`).\
Create an annotated tag:
```bash
git tag -a v1.0 -m "First stable release"
```
Tag an older commit:
```bash
git tag -a v0.5 f9cd629 -m "Old version"
```
Checkout a tag:
```bash
git checkout v1.0
```
(You enter a detached HEAD state.)\
Show tag info:
```bash
git show v1.0
```

___________
# GPG Signing
Install GPG, then create keys:
```bash
gpg --gen-key
```
List keys:
```bash
gpg --list-secret-keys --keyid-format LONG
```
Set Git signing key:
```bash
git config --global user.signingKey <KEYID>
```
Sign a tag:
```bash
git tag -s v1.0 -m "Signed version"
```
Sign a commit:
```bash
git commit -S -m "Signed commit"
```
Verify:
```bash
git tag -v v1.0
```
___________

# Git Blame
Find who last modified a line:
```bash
git blame index.php -L4,5
```
___________
# Git Bisect
Bisect helps you find which commit introduced a bug.\
Start bisect:
```bash
git bisect start
git bisect bad        # current commit has bug
git bisect good <hash>  # known good commit
```
Git will now checkout different commits.\
At each step:
```bash
git bisect good
# or
git bisect bad
```
Git automatically narrows down the exact faulty commit.
___________
