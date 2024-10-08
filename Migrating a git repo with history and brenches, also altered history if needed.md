
# Migrating a git repo with history and brenches, also altered history if needed.

I will use source and destination naming for the 2 repositories, one remote from which the code is migrated to tthe destination remote, via local machine.
    
Create in the destination repo a new repository empty with no readme, license, .gitignore  etc.

On the local machine clone the source repo with the `–mirror` option. This will keep the source repo history and its old branches
    
        `git clone --mirror https://<source-repo-domain>/<the-repo>.git`

Navigate in the folder just cloned and:

        `git remote add <destination> https://<destination-repo-domain>/<new-repo-name>.git`
        `git push --mirror <destination>`

If there are issues with files that are too big and the push fails, there are a few options, but for some of these options (using `git lfs track`) one needs admin rights on the source repo. Should that not be the case, one can remove the fat file stil, alas this means altering the history of the repo. Here's how to do it:
    
        `git filter-repo --strip-blobs-bigger-than <desired-limit>`

If `filter-repo` is not installed, it can be installed with python (or brew for mac). Or use java:  `java -jar bfg.jar --strip-blobs-bigger-than <desired-limit> --no-blob-protection`, but you need the jar file.

After this little clean up the history is altered and this version will be pushed to the new repo.

        `git remote add <destination> https://<destination-repo-domain>/<new-repo-name>.git`
        `git push --mirror <destination>`

Remove the old remote origin:

        `git remote remove origin`

Set the origin remote:

        `git remote add origin https://<destination-repo-domain>/<new-repo-name>.git`
        `git fetch`
        `git branch --set-upstream-to=origin/master`
Here `master` is the main branch. It can have another name, depending on the repo is setup.
And done. The local machine files can be removed. Working with the repo involves cloning the destination repo.


# Keeping the 2 repos synchronized

If for any reason keeping two repos synchronized is needed, given that the 2 repos have different histories, it is a bit complicated. Not impossible.

Clone locally the repo from your desired remote repo (in this case from 'destination' repo):

        `git clone https://<destination-repo-domain>/<new-repo-name>.git`


Set remote origin to the desired/destination repo and the remote <desired-name> (here 'source')to the secondary repo:

        `git remote add origin https://<destination-repo-domain>/<new-repo-name>.git`
         
        `git remote add <source-name> https://<source-repo-domain>/<the-repo>.git`
     
        # check the remotes:
        `git remote -v`
         
        # switch to a desired branch
        `git pull origin master master`

In order to make commits with only recent changes - that do not affect the whole history there are 2 options:


### Option 1: `git fetch` and `git cherry-pick`:

Fetching from the remote without merging or rebasing, then manually cherry-picking specific commits.

        `git fetch <source>`

Then, list the commits from the remote branch:

        `git log <source>/branch-name`

Check which commits to add to the local branch and cherry-pick them in the desired order:

        `git cherry-pick <commit-hash>`

If applying a range of commits from `commit1` to `commit2` is desired, one can use:

        `git cherry-pick commit1^..commit2`

         
### Option 2: `git fetch` and `git merge` with `--no-ff`

Fetch the changes from the remote but don't merge them:

        `git fetch <source>`

Find the desired commits, say the last 5 commits:

        `git log <source>/branch-name`

Find the hash of the 5th commit from the top of the remote branch, then merge:

        `git merge --no-ff commit-hash`

`--no-ff` means "no fast forward". It creates a merge commit even if the merge could be resolved as a fast forward, which is useful when wanting to preserve the exact history of the commits.
There are other options also, this info is not meant to be exhaustive. Share if you like :-)

### Option 3: Create a Temporary Branch

Create a new temporary branch at the remote's target commit and then pull in that branch:
   
    `# Fetch the remote changes
    git fetch bitbucket`
 
    `# Create a temp branch from the desired commit
    git branch temp-branch commit-hash` 
 
    `# Checkout to the temp branch
    git checkout temp-branch`
 
    `# Merge the temp branch into your current branch
    git merge temp-branch`
 
    `# Delete the temp branch
    git branch -d temp-branch`
    
