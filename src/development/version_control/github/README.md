# Github

### Quick links
* [CLI](#cli)
* [Container Registry](#container-registry)
* [Github Pages](#github-pages)
* [Github Large File Storage](#github-large-file-storage)
* [Security](#security)
  * [Public key](#public-key)
  * [Codeowners](#codeowners)

## CLI
In more complicated projects you'll want to automate github operations like posting PRs, assigning
reviewers or adding labels. These types of operations fall into the github automation category and
are served by a few different tools.

The [Github CLI](https://cli.github.com/) project brings Github to your terminal as a free an open
source project using terminology consistent with github. The [Github CLI is also written in GO](https://github.com/cli/cli)

## Container Registry
Github container registry uses a new domain `ghcr.io/OWNER/IMAGE_NAME` e.g.
`ghcr.io/phR0ze/alpine-net`.

**References**:
* [Container Registry About](https://docs.github.com/en/packages/guides/about-github-container-registry)

ITS A JOKE - DATA TRANSFER LIMITS ARE RIDICULOUS

## Github Pages
Github pages are pretty slick as they provide a simple way to create a static website for your 
project. Originally I used this as a nice way to host package files for my Arch Linux distro. However 
Github Pages has a strict file size limit of 100MB which makes this use case difficult as more and 
more packages are exceeding the limit these days.

## Github Large File Storage
**WARNING** the git-lfs feature for Github is worthless as it has a 1Gb upload limit for opensource 
free accounts. Additionally it seems to interfere with powerline causing it to hang for a minute 
repeatedly. By turning off powerline in your shell before navigating to that project you can avoid 
this.

Git Large File Storage (LFS) replaces large files such as audio, videos, datasets and graphics with 
text pointers inside Git, while storing the file contents on a remote server like GitHub.com

1. Install the git-lfs extension for Arch Linux
   ```bash
   $ sudo pacman -S git-lfs
   ```
2. Navigate to the repo you'd like to use git-lfs for
   ```bash
   $ cd ~/Projects/cyberlinux-repo
   ```
3. Enable git-lfs for that repo which adds the configs `post-checkout`, `post-commit`, `post-merge`, 
   and `pre-push` to your local `.githooks` directory
   ```bash
   $ git lfs install
   ```
4. Configure the target extensions to track large file types
   ```bash
   $ git lfs track "*.zst"
   $ git lfs track "*.xz"
   ```
5. Now add all changes and commit them out and push them
   ```bash
   $ git add .
   $ git commit -m "Adding git lfs support"
   $ git lfs migrate import --everything --above=55Mb
   $ git push -f
   ```

## Security

### Public key
Github requires a configured public key for your account before you can clone repos using the 
`git@github.com` syntax or that you need to specifically call out a user token in the URL.

1. You can check if you have a public key already configured on you local system
   ```bash
   $ ssh -T git@github.com
   Hi USERNAME! You've successfully authenticated...
   ```
2. If instead you get the `Permission denied (publickey)` you then need:
   1. [Generate a new RSA key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
   2. Login to your Github Account
   3. Navigate to `Settings`
   4. In the `Access` section of the sidebar click `SSH and GPG keys`
   5. Click `New SSH key`
   6. Paste in your public key
   7. Authorize it if necessary with your organization

### Codeowners
[CODEOWNERS](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners) 
are automatically requested for review when someone opens a pull request that modifies code that they 
own. Code owners are not automatically requested to review draft pull requests; however once the PR 
is marked as ready for review the codeowners are automatically notified. This feature when combined 
with the branch protection rule to `Require review from Code Owners` effectively blocks merges 
without proper eyes seeing the changes.

* the last matching entry has the highest level of precedence

* File location

* File syntax
[Codeowners file syntax](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners#codeowners-syntax)
Consists of patterns similar to a gitignore file with owners associated with the file pattern 
matches.

* gitignore type file pattern match followed by one or more users or groups using the `@org/team-name` format

<!-- 
vim: ts=2:sw=2:sts=2
-->
