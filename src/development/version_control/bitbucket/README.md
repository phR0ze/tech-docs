# Bitbucket

### Quick links
* [Bitbucket Static Pages](#bitbucket-static-pages)
* [Bitbucket Publish Pages](#bitbucket-publish-pages)
* [Bitbucket SSH Keys](#bitbucket-ssh-keys)

## Bitbucket Static Pages
* [Repo and file size limits](https://confluence.atlassian.com/bbkb/what-are-the-repository-and-file-size-limits-1167700604.html)
* [Repo size limits](https://bitbucket.org/blog/repository-size-limits)
* [Configure static website](https://support.atlassian.com/bitbucket-cloud/docs/publishing-a-website-on-bitbucket-cloud/)

* Limitations
  * 1 GB repo soft limit
  * 2 GB repo hard limit
  * 5000 requests per hour
  * 2 GB archive.zip files
  * Upload file size limit of 1GB?
    * I was able to push 115MB
    * Failed with 200+MB file :(
  * Push limit of 3.5 GB/hour
* Features
  * Unlimited private repos
  * Git LFS limit of 1GB
* Workspace affects the clone URL
  * e.g. `git clone https://bitbucket.org/cyberlinux/aur.git`

## Bitbucket Publish Pages
Note: because of 200+MB limit I'm going to have to host my own packages

[Reference](https://support.atlassian.com/bitbucket-cloud/docs/publishing-a-website-on-bitbucket-cloud/)
1. Create a new repository
   1. Set the Project name to `cyberlinux.bitbucket.io`
   2. Clone your repo with git protocol `git@bitbucket.org:cyberlinux/cyberlinux.bitbucket.io.git`
2. Add a `index.html` to the root of your project and commit and push the changes
3. Navigate to your new site [https://cyberlinux.bitbucket.io/](https://cyberlinux.bitbucket.io)
4. Directories are treated as their own sites
   1. Create a new directory `packages`
   2. Add nested directories to that dir `packages/cyberlinux/x86_64`
   3. This will then be accessible from arch linux with `Server = https://cyberlinux.bitbucket.io/packages/$repo/$arch`
5. Adding a new package
   1. Navigate to `~/Projects/cyberlinux.bitbucket.io/packages/cyberlinux/x86_64`
   2. Copy the target package here
   3. Remove stale index files `rm cyberlinux.*`
   4. Rebuild the index files `repo-add cyberlinux.db.tar.gz *.pkg.tar.*`
   5. Replace soft links with hard links for bitbucket to play nice
      ```
      $ ln cyberlinux.db.tar.gz cyberlinux.db -f
      $ ln cyberlinux.files.tar.gz cyberlinux.files -f
      ```

## Bitbucket SSH Keys
Use SSH to avoid password prompts when you push code to Bitbucket

1. Edit your ssh config `~/.ssh/config`
2. Add a match block for bitbucket
   ```
   HostName bitbucket.org
   IdentityFile ~/.ssh/id_rsa
   ```
3. Now Navigate to your workspace root in bitbucket
4. Click the `SSH keys` option on the left
5. Click `Add key`
6. Give the key a label then paste in your public key
