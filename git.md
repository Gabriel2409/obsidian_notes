#git
### Config
```bash
# shows values of configs as well as origin (config file, repo...) 
git config --list --show-origin

# set username and email
git config --global user.name <myname>
git config --global user.email <myemail>

# check global config in ~/.gitconfig
[user]
	name = <name>
	email = <email>

# can also check local config in current_repo/.git/config

# configure core editor
git config --global core.editor 'NVIM_APPNAME="nvim/nvim-lazyvim" nvim'

# change default branch
git config --global init.defaultBranch main

# alias
git config --global alias.hist 'log --oneline --graph --decorate --all'
git config --global alias.lazy '!lazygit' # now git lazy opens lazygit

# rebase instead of merge on pull
git config --global pull.rebase true

# force each commit to be signed
git config --local commit.gpgsign true
```

### add
```bash
git add myfile # stages myfile
git add -i # enter interactive mode
git add --patch # enter patch mode (stage only some hunks)


```

### status, logs, resets

```bash
# short status
git status -s

# Restores to state in working directory
git restore file # same as 
git checkout -- file 
# Remove file from staging area
git restore --staged <file> # same as
git reset HEAD <file> # HEAD is optional as it is the default

# More on resets
# Resets moves what HEAD points to, this is not the same as changing HEAD like in checkout
# --soft only reset HEAD=> go back in history and modified files since then are staged, can not be used with files
git reset --soft HEAD~1

#--mixed: default, also resets the index => unstages modifications
git reset <myfile>

# --hard, also resets the working directory => unstages and deletes modifications, can not be done with file paths
git reset --hard HEAD~2

# remove file from git but not from working dir
git rm --cached file

# changes not yet staged
git diff 

# compare staged changed
git diff --staged

# see changes introduced by each commit
git log -p
# see stats 
git log --stat
# specify own format
git log --pretty=format:"%h - %an, %ar : %s"

# Amend commit. Stage new files and run
git commit --amend

```
Reset recap:
1. Move the branch HEAD points to (stop here if --soft).
2. Make the index look like HEAD (stop here unless --hard).
3. Make the working directory look like the index.


### remote
```bash

# Add remote
git remote add <shortname> <url>
# Modify url
git remote set-url <shortname> <url>

# extra info on a remote
git remote show <shortname>
git ls-remote

# rename and remove
git remote rename pb paul
git remote remove paul

```
### Tags
```bash

# standard tags are lightweight, just a pointer to a specific commit
git tag v1.5-lw
git show v1.5-lw # we see as if we just showed the commit

# usually better to create annotated tabs which are stored as full objects in the git db
git tag -a v1.5 -m "my version 1.5"
git show v1.5 # see extra info about the tag in addition

# tag later
git tag -a v1.2 9fceb02

# share tag
git push origin <tagname>
# share all tags
git push origin --tags

# delete tag
git tag -d v1.5-lw

# delete tag on remote
git push origin --delete v1.5-lw # or
git push origin :refs/tags/v1.5-lw # interpreted as null value before : pushed to remote tag name which deletes it
```

Note: standard tags are stored in `.git/refs/tags` and contain pointer to a commit
Annotated tag contain instead pointers to a tag object in `.git/refs/tags` which contain a pointer to a commit as well as additional info
### Branch
```bash
git branch # shows the branches
--merged and --no-merged to filter
--all to see remotes as well
# rename branch locally
git branch --move bad-branch-name corrected-branch-name
# upstream
git push --set-upstream origin corrected-branch-name
git push origin --delete bad-branch-name

# change branch
git switch -c branch # same as
git checkout -b branch 

## push
git push origin mybranch
# shortcut for 
git push origin mybranch:mybranch
# shortcut for
git push origin refs/heads/mybranch:refs/heads/mybranch
# to push local branch to another name
git push origin localbranch:serverbranch

## track 
git checkout -b <mybranch> <remote>/<mybranch> # same as
git checkout --track <remote>/<mybranch> # same as
# if the branch does not exist locally but exists on the remote 
git checkout <mybranch> 


# merge
git merge other-branch
# cancel merge
git merge --abort
# choose current branch version
git checkout --ours myfile
# choose other branch version
git checkout --theirs myfile
# Alternatively, modify the file directly and stage it


# Rebase
git rebase main mybranch # mybranch is optionnal if you are checked out on mybranch

# rebase workflow
git checkout exp
git rebase master # rebases master on exp
git checkout master
git merge exp

# this rebases s2 without any changes from s1
git rebase --onto main s1 s2

# example
git checkout main
git checkout -b s1
touch f1
git commit -m "f1"
git checkout -b s2
touch f2
git commit -m "f2"
git rebase --onto main s1 s2
# after this, on s2, there is only f2, not f1 and the previous commit is from main

# rebase on pull
git pull --rebase
```

###  Diff @ ^ ~

@ for reflog
^ which parent (useful for merge commit)
~ nb of first parents 
```bash
git reflog # shows history of HEAD position

git show HEAD@{5} # 5 position of HEAD ago
git show master@{yesterday} # where was HEAD yesterday
git show HEAD@{2.months.ago} # 2 months ago

git log -g # shows reflog info

git show HEAD^2 # second parent in a merge commit
git show HEAD~2 # first parent of first parent (grandparent)

# find all commits reachable by experiment but not master
git log master..experiment # same as
git log experiment --not master # same as
git log ^master experiment

# find all commits reachable by experiment or master but not both 
git log --left-right master...experiment 

git rev-parse mybranch # shows sha of given branch
```

### stash

```bash
git stash # stashes current changes
git stash list # see all stashes
git stash apply stash@{2} # applies stash number 2

# try to restage files that were staged before stash
git stash apply --index 

git stash drop stash@{2} # drops the stash

git stash pop # applies and drop 

# stashes everything but at the same time keep staged files 
git stash --keep-index 

git stash -u # include untracked

git stash --patch # interactive stash

# applies the stash on another branch, starting point is 
# useful if we need to handle large conflicts
git stash branch mybranch 

git clean # DANGER removes everything untracked and modified
git clean --dry-run # or -n, see what would be removed
git stash --all # safer option

```

### Signing 

https://docs.github.com/en/authentication/managing-commit-signature-verification/generating-a-new-gpg-key
```bash
# list gpg keys
gpg --list-keys
gpg --gen-key
git config --global user.signingkey 0A46826A!

# sign tag
git tag -s v1.5 -m 'my signed 1.5 tag'
# verify tag
git tag -v v1.5

# sign commit
git commit -S -m 'Signed commit'
# see signature
git log --show-signature

# make merge fail if it contains unverified commits
git merge --verify-signatures mybranch
git merge --verify-signatures -S mybranch # also signs merge commit


# if everyone must sign, good config option
git config --local commit.gpgsign true

# if you forgot to sign previous commit
git commit --amend --no-edit -S
```
Signing old commits: https://superuser.com/questions/397149/can-you-gpg-sign-old-commits

### Searching

```bash
git grep pattern #searches patterns in git tree
git log -S ZLIB_BUF_MAX --oneline # find commit where ZLIB_BUF_MAX was introduced

# all changes to function myfunc in utils.py
git log -L :myfunc:utils.py

```

### Find commit that introduced bug
```bash
git bisect start
git bisect bad # tells current commit is problematic
git bisect good <sha> # gives a commit that works
# then for each commit you get, run git bisect good / bad
git bisect reset # resets HEAD at where we were at the beginning
```
Or run a script that has a non zero exit status on failure
```bash
git bisect start HEAD <old_commit>
git bisect run test-error.sh
```

### Rewrite history
```bash
# last commit
git commit --amend

# interactive rebasing
git rebase -i HEAD~3
```
Note: the argument of interactive rebasing is the parent of the last commit we want to rebase. Which means it is not included. 

We can easily edit commits and squash them. To drop them, add drop or delete the line.

To split a commit, choose edit in interactive rebasing, then
```bash
git reset HEAD^
git add myfile
git commit -m "A"
got add myfile2
git commit -m "B"
git rebase --continue
```

## .git folder

### files

`.git/COMMIT_EDITMSG`contains the commit message of a commit in progress. If `git commit` exits due to an error before creating a commit, any commit message that has been provided by the user (e.g., in an editor session) will be available in this file, but will be overwritten by the next invocation of `git commit`

`.git/HEAD` contains pointer to current commit
```bash
git checkout main
cat .git/HEAD # ref: refs/heads/main
git checkout d3230bab359561b4c93061ca7b4015fcc86e5bc0
cat .git/HEAD # d3230bab359561b4c93061ca7b4015fcc86e5bc0
```

`.git/config` contains specific config for the repo. Full config consists in global config and this config


`.git/description` is just a text file?

`.git/index` stores staged filed: it is a binary file containing a sorted list of path names, each with permissions and the SHA1 of a blob object: https://mincong.io/2018/04/28/git-index/
```bash
git ls-files --stage
100644 802992c4220de19a90767f3000a79a31b98d0df7 0	README.md
```

The flag `--stage` shows staged contents’ mode bits, object name and stage number in the output. So in our case, the output means:

- The readme file mode bits are 100644, a regular non-executable file.
- The SHA-1 value of blob README.md
- The stage number (slots), useful during merge conflict handling.
- The object name.



### objects

`.git/objects` contains all the objects of the repo. The hash is divided in the folder (2 chars) and the rest of the hash. For ex an object of hash `f7b1c8d0f73fa7326cda7e38d4e81ad82a2d1b76` is stored in `.git/objects/f7/b1c8d0f73fa7326cda7e38d4e81ad82a2d1b76`

The content of each object is compressed with zlib (default compression level, i believe 6), but you can read it with `git cat-file`

```bash
git cat-file -p <hash> # prints content
git cat-file -t <hash> # prints type (commit, tree, blob, tag...)
git cat-file -p master^{tree} # last tree pointed by master branch
```

Each time we commit, it adds git objects:
- the commit itself which contains a reference to parent commit and the root tree object
- tree objects which contain references to blob and other trees
- each blob contains the full content of the file (compressed). git does not store the changes but the full file, which is why it is very fast to change ref.


For blobs, first we calculate the `<size>` of the raw `<content>`.
Then we calculate the hash (sha1) of `blob <size>\0<content>`. What is actually stored is the zlib compresion of `blob <size>\0<content>`

For trees, we use `tree <size>\0<content>` instead. Content is several lines (not separated by `\n`) of `<mode> <name>\0<20_bytes_sha1>` where `<name>` is the file name and `<mode>` is `40000` for directories, `100644` for regular files, etc: more info on `<mode>` => `man -s 7 inode`

For commits, we use `commit <size>\0<content>`. Here the lines are separated by `\n`
```bash
tree <tree_hash>
parent <hash> # actually there is one line per parent
author <author_name> <author_email> <author_timestamp> <timezone>
committer <committer_name> <committer_email> <committer_timestamp> <timezone>

<commit_msg>
```
Note that emails are enclosed in `<>`


`git hash-object <file>` allows to get the sha1 hash of a blob. Passing the `-w` flag optionnaly writes it to the .git/objects folder (low level command)

`git update-index` allows to write files to the index (staging area).

`git write-tree` writes a tree object and all corresponding blobs based on the staging area

`git commit-tree` takes a tree sha and parent commits to write a commit object


Note: About pack files
https://app.codecrafters.io/courses/git
https://www.git-scm.com/docs/http-protocol
https://github.com/git/git/blob/795ea8776befc95ea2becd8020c7a284677b4161/Documentation/gitformat-pack.txt
https://codewords.recurse.com/issues/three/unpacking-git-packfiles