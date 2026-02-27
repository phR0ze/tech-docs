# Git
<img align="left" width="40" height="40" src="../../../data/images/logo_256x256.png">

### Quick links
* [Config](#config)
* [Branches](#branch)
  * [Delete Branch](#delete-branch)
  * [Push Branch](#push-branch)
* [Clone](#clone)
  * [Clone a specific branch](#clone-a-specific-branch)
* [Cherry Pick](#cherry-pick)
* [Diff](#diff)
  * [Diff Branches](#diff-branches)
  * [Diff Name Only](#diff-name-only)
  * [Diff Specific Dir](#diff-specific-dir)
  * [Diff Two Commits](#diff-two-commits)
* [Hooks](#hooks)
  * [Set Local](#set-local)
* [Reauthor](#reauthor)
  * [Reauthor Range](#reauthor-range)
* [Rebase](#rebase)
  * [Squash](#squash)
* [Remotes](#remotes)
  * [Add Remote](#add-remote)
  * [Rename Remote](#rename-remote)
* [Resolve Conflicts](#resolve-conflicts)
* [Tags](#tags)
  * [Checkout a tag](#checkout-a-tag)
  * [Delete All Tags](#delete-all-tags)
  * [Delete Matching Tags](#delete-matching-tags)

## Config
Git supports conditional configuration includes:

1. Create your `~/.gitconfig` with your regular configuration. At the bottom add the conditional 
   configuration for your `company_a` which will override the previously set values.
   ```
   [user]
     name = John Smith
     email = js@gmail.com
   [push]
     default = simple
   
   [includeIf "gitdir:~/Projecdts/work/company_a/"]
     path = ~/.config/.gitconfig-company_a
   ```

2. Now create your company specific values to override with at `~/.config/.gitconfig-company_`
   previously set that match. Using a github token explicitly for your company allows for simple ssh 
   keys management in that your default will handle your github account while the tokens handle your 
   company account. Alternatively you can do some `~/.ssh/config` foo but its more complicated.
   ```
   [url "https://user:git-hub-token@github.com/company_a"]
     insteadof = company_a:
   [user]
     name = smithj
     email = smithj@companya.com
   ```

## Branches

### Delete Branch

* Delete Local Branch
  ```bash
  $ git branch -d <branch>
  ```

* Delete Remote Branch
  ```bash
  $ git push origin -d <branch>
  ```

### Push Branch

* Push to Different Name
Call out the remote first then the branch names `<local-name>:<remote-name>`

```bash
# Push local working to upstream/master
$ git push upstream working:master

# Push local master to upstream/fix
$ git push upstream master:fix
```

## Clone

* Clone a specific branch
  ```bash
  $ git clone --single-branch -b <branchname> <remote-repo>
  ```

## Cherry Pick
Git cherry-pick allows you to merg in a single commit from one branch to another.

```bash
$ git cherry-pick <HASH>
```

## Diff

### Diff Branches
```bash
$ git diff v1.0.0..v1.0.1
```

### Diff Name Only
```bash
$ git diff HEAD^ --name-only
```

* Diff Name Only with Excludes
You can exclude files from a diff with the `:(exclude)<pattern>` syntax.

```bash
# git diff HEAD~20 --name-only -- . ':(exclude)<pattern>'
$ git diff HEAD~20 --name-only -- . ':(exclude)vendor/*'
.gitignore
.vscode/launch.json
Gopkg.lock
Gopkg.toml
Makefile
README.md
VERSION
go.mod
go.sum
```

### Diff Specific Dir
```bash
$ git diff HEAD^ --name-only /dir/to/filter/on
```

### Diff Two Commits
```bash
$ git diff 176aaaaf fecb1299 --name-only
```

## Hooks

### Set Local
Often we'll want custom hooks to run after commits to auto incrementa a version number or something.
To do this create the hooks in your repo at `.githooks` then configure your repo to use the hooks.

```bash
# Change to your project root
cd ~/Projects/cyberlinux

# Set hooks to be used locally from your repo
git config core.hooksPath .githooks
```

## Reauthor

### Reauthor Range
1. Identify the last good commit:
   ```bash
   $ git log -5 --oneline
   31ce85f31 (HEAD -> staging) Patch nil reference bug
   6d74d628e Adding new Dockerfile and cleaning up README and Makefile
   d490c85c4 Versioning vendored packages
   038bad502 Simplifying root dir
   89bd14c15 (tag: v2.16.5, origin/release-2.16) Fixing validation issue
   ```
2. Invoke an interactive rebase and change all entries to `edit`:
   ```bash
   $ git rebase -i -p 89bd14c15
   ```
3. Each time rebase stops:
   ```bash
   $ git commit --amend --author="user <user@example.com>" --no-edit
   $ git rebase --continue
   ```

## Rebase
### Squash
```bash
# Squash the last 78 commits into a single commit
$ git rebase -i HEAD~78
```

## Remotes

### Add Remote
```bash
# syntax: git remote add <name> <url>
$ git remote add origin ssh://user@ip-address/root/path/to/project
```

### Rename Remote
```bash
# syntax: git remote rename <old> <new>
$ git remote rename origin upstream
```

## Resolve Conflicts
This can happen when your merging, rebasing or cherry-picking commits

```bash
# Run status to see conflicts
$ git status
On branch istio
Your branch is up to date with 'upstream/master'.

You are currently cherry-picking commit 3dc0eb85.
  (fix conflicts and run "git cherry-pick --continue")
  (use "git cherry-pick --abort" to cancel the cherry-pick operation)

Changes to be committed:
	modified:   project1/templates/deployment.yaml

Unmerged paths:
  (use "git add/rm <file>..." as appropriate to mark resolution)
	both modified:   project1/templates/cronjobs.yaml
	both modified:   project1/templates/db-migrate.yaml
	deleted by them: project1/templates/foo.yaml

# Now deal with them
# Manually: alias gf='vim `git diff --name-only --diff-filter=M | uniq`'
# Note in this context HEAD the new stuff your merging in
$ gf

# Mark them resolved
$ git add .

# Continue
$ git cherry-pick --continue
```

## Tags

### Checkout a Tag
```bash
$ git checkout -b <my branch> tags/<my tag>
```

### Delete All Tags

Delete all local tags
```bash
$ git tag -d $(git tag -l)
```

Fetch all remote tags
```bash
$ git fetch
```

Delete all remove tags
```bash
$ git push origin --delete $(git tag -l)
```

Delete all local tags
```bash
$ git tag -d $(git tag -l)
```

### Delete Matching Tags

Delete all matching local tags
```bash
# Deletes all local tags starting with 'v1'
$ git tag -d $(git tag -l "v1*")
```

Delete all matching remote tags
```bash
$ git push origin --delete $(git tag -l)
```
