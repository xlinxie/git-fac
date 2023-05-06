# git-fac

Frequently asked use cases when using Git

## Recreate or reinit a git repository

Since Git keeps all the metadata in the hidden `.git` folder, which lies in the repo root folder,\
so we can just delete that folder and recreate a Git repo.

Remove the `.git` folder won't affect any of our repo files,
we just lose all the revision logs of files

```shell
# inside your repo folder, delete the metadata folder
# Linux/Unix
rm -rf .git
# Windows PowerShell
rm -R -Force .git

# init again
git init
```

## Clone into a specific folder other than folder named after the repository

You may wanna a short catchy folder name or maybe there is a folder with the same name as the repo to be cloned.

```shell
# syntax: git clone <repo-url> <folder-name>
git clone https://github.com/git/git-reference.git git-ref
```

## Let git remember my username and password

You may cloned a repo via `HTTPS` protocol, every time you push new commits to remote repository,\
Git will ask you for username and password, it tedious and annoying.

```shell
# let git cache your usrename and password, cache will be purged after 900 seconds by default
git config --global credential.helper cache

# set expire time in seconds
git config --global credential.helper 'cache --timeout=7200'

# or stores the credentials to a plain-text file on disk, they never expire
git config --global credential.helper store
```

## Get name or other info of current branch

```shell
# show all local branches, with current branch highlighted
git branch

# show current branch name only
git branch --show-current

# reverse parsing current branch name from the HEAD object
git rev-parse --abbrev-ref HEAD

# show branch name, latest short commit hash, upstream tracking branch, latest commit message
git branch -vv
```

## Clone only a specific branch of a repo

Some repos may be very large(accumulates long history or contains huge binary assets) in size and have many branches,\
to perform a fast clone, we can clone a repo only with the branch we are interested in.

```shell
# syntax: git clone <repo-url> -b <branch-name> --single-branch
git clone https://github.com/facebook/react.git -b main --single-branch

# without --single-branch argument
remote: Enumerating objects: 229257, done.

# with --single-branch argument
remote: Enumerating objects: 164242, done.

# if branch not specified, the default branch(main or master) will be cloned
git clone https://github.com/facebook/react.git --single-branch
```

to be even faster, do a shallow clone by just getting only a specific amount of commits

```shell
# clone with only the most recent one revision
# syntax: git clone --depth <number> <repo-url>
git clone --depth 1 https://github.com/facebook/react

# later, convert to a full clone if necessary
git fetch --unshallow
```

## Create a new branch from another branch

```shell
# syntax: git checkout -b <branch-name> <source-branch-name>
git checkout -b dev main

# checkout from current branch if <source-branch-name> not given
git checkout -b dev
```

## Rename a local branch

Rename a local branch works the same  way as renaming file in Linux/Unix system, you just move it into a new one

```shell
# syntax: git branch -m|--move <old-branch-name> <new-branch-name>
git branch -m bugfix hotfix

# if <old-branch-name> not present, current branch will get renamed
git branch -m hotfix
```

## Rename a branch both locally and remotely

You checked out a branch from remote and renamed it later. You wanna all your collaborators to use the new branch.

```shell
# rename a local branch is easy
# syntax: git branch -m <old-branch-name> <new-branch-name>
git branch -m bugfix hotfix

# you'll find out that your hotfix branch is still tracking the origin/bugifx upstream branch
# others won't see the newly renamed branch until you pushed it up
git branch -vv

# switch to the renamed branch and push it to remote repo
# push with the --upstream/-u argument to make it the new upstream tracking branch
# syntax: git push -u <remote-name> <new-branch-name>
git push -u origin hotfix

# add --delete/-d argument to remove the old tracking branch from remote
# syntax: git push -d <remote-name> <old-branch-name>
git push -d origin bugfix
```

[Definition of "downstream" and "upstream"](https://stackoverflow.com/questions/2739376/definition-of-downstream-and-upstream)
[What is a git upstream](https://stackoverflow.com/questions/17122245/what-is-a-git-upstream)

## Delete a branch both locally and remotely

```shell
# remove local branch
# syntax: git branch -d <branch-name>
git branch -d hotfix

# remove remote tracking branch
# syntax: git push -d <remote-name> <branch-name>
git push -d origin hotfix
```

There is also a tricky way to remove a remote branch

```shell
git push origin :hotfix
```

[git-push - Update remote refs along with associated objects](https://git-scm.com/docs/git-push)

> Pushing an empty `<src>` allows you to delete the `<dst>` ref from the remote repository.
> Deletions are always accepted without a leading + in the refspec (or --force),
> except when forbidden by configuration or hooks.

## Recover a deleted branch

```shell
# check local git reference log to find the last commit hash of the deleted branch
git reflog

# recreate branch from that point
# syntax: git checkout -b <branch-name> <last-commit-hash>
git checkout -b recovered 0de29ad
```

> Reference logs, or "reflogs", record when the tips of branches and other references were updated in the local repository.

[https://git-scm.com/docs/git-reflog](https://git-scm.com/docs/git-reflog)

## Specify the remote tracking branch for a local branch

You created a local branch `feat/zenmode` and wanna push this branch to remote repository.\
You need to tell git which branch in the remote repo is the corresponding branch of your local branch, this is called branch tracking.

```shell
# when perform git push on a newly created local branch
# git will prompt for setting a tracking upstream, since a corresponding branch does not exist in the remote repo
# syntax: git push -u|--set-upstream <remote-name>/<branch-name>
git push --set-upstream origin/feat/zenmode

# if the feat/zenmode branch does not exist in the remote repo, git will create one
# next time just push without specifing remote and branch
git push

# you may push without setting a upstream tracking branch
git push origin feat/zenmode
```

## Checkout a remote branch

```shell
# since git is distributed, sync refs and remote-tracking branches to keep your local repo up-to-date
git fetch

# if you have multiple remote reposistories, add --all argument to sync with all remote repos
git fetch --all

# checkout a remote branch and make implicit upstream branch tracking
# syntax: git checkout <remote-branch-name>
git checkout dev

# git will create a new local branch with the same name as the remote branch
# and set it as the upstream tracking branch for the local branch
git branch -vv

# or if you prefer ex plicit upstream tracking
# works the same as `git checkout dev`
# syntax: git checkout -b <local-branch-name> <remote-name>/<tracking-branch-name>
git checkout -b dev origin/dev
git checkout -b dev fork/dev

# checkout the remote dev branch to local with a different name
git checkout -b beta origin/dev
```

Your local branch will be pushed to a namesake branch, but git won't set the namesake branch as the upstream tracking branch.\
Every time you wanna push to that remote branch you will need to specify the remote and branch.

## Show only a few commits

```shell
# syntax: git log -<number>
# show only last 7 commits
git log -7
# from a given point
git log fd1ec82547d6139e66556ad3f694ffdc22858670 -3
```

## Find a commit by message

```shell
# syntax: git log --grep="<pattern>"
git log --grep="keywordhere"
```

## Show commit log in tree view

```shell
git log --all --decorate --oneline --graph

# config a global alias for later quick usage
git config --global alias.tree "log --graph --pretty=oneline --abbrev-commit"
git tree
```

## View specific user's commit log

Often we need to check our collaborator's revisions

```shell
# syntax: git log --author=<regex-pattern>
git log --author="John Doe"
git log --author="johndoe@example.com"
```

## View the revision history of a file

```shell
# syntax: git log -p -- <file-path>
git log -p -- README.md

# or visually with gitk
# syntax: gitk --follow <file-path>
# without the `--follow` option, only history up to the point where the file was renamed will be displayed
gitk --follow README.md
```

## Show all authors

```shell
# get username, email and commit count info from current branch
git shortlog --summary --numbered --email
# short version
git -sne

# get data from all branches
git -sne --all
```

## Recover a deleted file from commit history

```bash
# show all commits that have deleted files from git index
git log --diff-filter=D --summary

# show all deleted files
# set --pretty to empty string will omit info such as commit hash, author
# set --diff-filter D --name-only to show deleted files paths
git log --pretty="" --diff-filter D --name-only | sed '/^$/d'

# if you know the path of deleted file, to find in which commit you deleted it
git log --diff-filter=D --name-only -- <file-path>

# or find the deletion commit by following the parent links from the HEAD commit in reverse chronological order
git rev-list -n 1 HEAD -- <file-path>

# then recover the deleted file
# syntax: git checkout <commit-hash>~1 -- <file-path>
git checkout 8f9a1a345c5cc050d2aa03cdcf1278270ba0215e~1 -- LICENSE.md
```

## Change the URI of a remote

```bash
# show remote info, find the remote you wanna change
git remote -v

origin  https://github.com/git/git.git (fetch)
origin  https://github.com/git/git.git (push)

# update the remote url
# syntax: git remote set-url <remote> <url>
git remote set-url origin https://gitlab.com/git/git.git
```

## Add all changed files and folder to the stage area

Sometimes you may changed quite a lot files that need multiple `git add` to

```shell
# stage all(new, modified, deleted) changes
# **git add -A** is prefered over **git add .**
# git add . only stage changes in current folder, it works the same only when current folder is root folder of your repo. and for Git 1.x, it will not add deleted files to the stage area
git add -A

# if you haven't introduce any new file, add --update or -u to stage modified and deleted files
git add -u
```

## Add and commit empty folder

create an empty placeholder file called .gitkeep | .placeholder | .gitignore in the folder you wanna keep, `git add` it and commit

## Commit only part of changes made to a file

```shell
# add --patch or -p option to select hunks interactively
# git will ask you to pick changes hunk by hunk
# syntax: git add -p <file-path>
git add -p README.md
```

## Make git detects file rename

Git is case sensitive, but macOS and Windows is case insensitive by default.\
So when you change file name from uppercase to lowercase or vice versa, git will not detect the change.

```shell
# git mv will rename file and update git index
# syntax: git mv <OLD_FILE_NAME> <new_file_name>
git mv readme.md README.md

# both macOS and Windows are case insensitive
# it's recommended to config git to be case sensitive to avoid this problem
git config core.ignorecase false
```

[Why does Git not "track" renames?](https://archive.kernel.org/oldwiki/git.wiki.kernel.org/index.php/Git_FAQ.html#Why_does_Git_not_.22track.22_renames.3F)

## Undo `git add` before commit

Sometimes you may added some files that you don't want to commit

```shell
# syntax: git reset <file-path>
git reset .DS_Store

# if <file-path> not given, all changes staged will be reset
git reset
```

## Show what added to the stage area

```shell
# diff or show verbose status
git diff --cached
git status -v

# show unstaged changes too
git diff
git status -vv
```

## Show changes made in a commit

```shell
# syntax: git show <commit-hash>
git show 54e5202642f7b6a2c850d2e0f3a2069ea4c16ffe
# if you don't care about file content
git show --compact-summary 54e5202642f7b6a2c850d2e0f3a2069ea4c16ffe
# or use a short equivalent version
git show --stat 54e5202642f7b6a2c850d2e0f3a2069ea4c16ffe

# or diff against the previous commit
# syntax: git diff <commit-hash>^
# put a ^ after the <commit-hash> means the parent of that <commit-hash>
git diff 54e5202642f7b6a2c850d2e0f3a2069ea4c16ffe^
```

## Show local commits that has not yet pushed

```shell
git log --branches --not --remotes

# a terse version, @{u} means the upstream tracking branch of current branch
git log @{u}..
```

## Undo a `merge` or `pull`

according to the official documentaion

`https://git-scm.com/docs/gitrevisions`

> ORIG_HEAD is created by commands that move your HEAD in a drastic way (git am, git merge, git rebase, git reset), to record the position of the HEAD before their operation, so that you can easily change the tip of the branch back to the state before you ran them.

so we can rollback by resetting to the `ORIG_HEAD`

```shell
git reset --hard ORIG_HEAD
```

## Undo the most recent local commit

Sometimes you may committed some files unexpcted and you haven't do a push yet

```shell
# soft reset, keep changes of last commit
git reset HEAD~1
# hard reset, which drops all the uncommitted changes of last commit
git reset --hard HEAD~1
```

if you wanna go back to a specific point

```shell
# or reset to a specific commit
# syntax: git reset <commit-hash>
git reset 9d3c7ef116a7aaa19675e3506e4a8a655cc49c0f
```

## Rollback a file to a specific revision

```shell
# first let's check all revisions of that file
# syntax: git log <file-path>
git log README.md

# checkout from previous commit
# syntax: git checkout <commit-hash> <file-path>
git checkout df7375d77287f3665867b99ae18fc3cbe3289a67 README.md
```

## Get one file from another branch

You wanna get a copy of some file that committed to another branch, or you wanna get the comitted version of file from another branch to override the file on your current branch

```bash
git checkout <branch-name> -- <file-path>

git restore --source <branch-name> -- <file-path>
```

## Overwrite local files with remote branch

You messed up your local branch and wanna restore it to the same state as the remote tracking branch

```shell
# syntax: git reset --hard <remote>/<branch-name>
git reset --hard origin/dev

# or use a handy version, @{u} means current branch's upstream tracking branch
git reset --hard @{u}
```

## Remove untracked files

```shell
# remove untracked files and directories
git clean -fd
# it's recommended to have a dry-run to check what would be done before actually do this, cause git does not track these files, they can't be recovered with git
git clean -fnd
```

## Remove tracked files from history

sometimes you may need to remove files committed, such as add some committed file to `.gitignore`

```shell
# git rm --cached <file-path>
# the `--cached` option prevents file from getting deleted from file system
git rm --cached pnpm-lock.yaml

# or have a dry-run by add `-n` or `--dry-run` option
git rm -n pnpm-lock.yaml
```

## Stop tracking and ignore changes made to a file

If you do not need this file anymore, you can just remove it from git index

```shell
# syntax: git rm <file-path>
git rm pnpm-lock.yaml
```

some file need be committed, such as some kind of config files. You need to change it locally, but you do not want your changes be detected or tracked, We can let git bypass revision checking.

```shell
# syntax: git update-index --assume-unchanged <file-path>
git update-index --assume-unchanged nuxt.config.ts

# --assume-unchanged is not good enough, when you perform a git pull, your local changes will be overwritten, to keep your own local dependent version of some file, add --skip-worktree
# syntax: git update-index --skip-worktree <file-path>
git update-index --skip-worktree .env.development
```

## Abort a commit in the middle of editing message

When you are in midway of editing commit, you suddenly firgured out some files need to be exclude, you wanna quit and start again

```shell
git commit -m "feat: add some feature"
# if you use vim as your editor, just type :cq to quit
# a general way for any editor is to leave the commit message empty
# in vim you can type ggdG to clear file content and then type :wq to do an empty commit
:cq

# another approach is to commit and reset last commit
git reset HEAD~1
```

## Modify existing, unpushed commit messages

**The general rule to modify commit history is that you should not rewrite history that you have pushed, because your collaborators might have done their work based on it.**

I always found guys use the same commit message for a bunch of continuous commits. If two commits share the same commit message, they should be combined into one. We can append current commit into last one by amend the last commit

```shell
# syntax: git commit --amend
git add changed-file-that-leave-behind.txt
git commit --amend
# if you don't need to edit the commit message
git commit --amend --no-edit
```

If you need to

to modify a specific commit

```shell
git rebase -i <commit-hash>^
```

## Copy or move recent commits to another branch

```shell
# syntax: git cherry-pick <commit-hash>
git cherry-pick 79f2338b3746d23454308648b2491e5beba4beff
git cherry-pick HEAD~3
git cherry-pick 79f2338b3746d23454308648b2491e5beba4beff..dd3f6c4cae7e3b15ce984dce8593ff7569650e24
```

```shell
git reset HEAD~3
git stash
git checkout hotfix
git stash pop
```

## Pick some commits from another branch

```shell
# diff current branch with target branch
# syntax: git cherry -v <target-branch-name>
git cherry -v hotfix

# pick the commits you would like to copy to current branch
# syntax: git cherry-pick <commit-hash>
git cherry-pick 0652e507fab6aedb4515d85adc0fe0c0a204e6e2

# or pick multiple commits with commit range
git cherry-pick 0652e507fab6aedb4515d85adc0fe0c0a204e6e2..4298b5b04ffdbacd77f11427e0b53f113cd17b18
```

## Combine multiple commits into one

```shell
git reset --soft HEAD~N
git commit

git rebase -i HEAD~N
git rebase -i <hash-of-last-nth-commit>
```

## Break a commit into multiple new commits

You may have a large commit that you wanna split into smaller ones.

```shell
# reset that commit, all changes of that commit rollback to unstaged status, then you can pick files to commit again
git reset <commit-hash>
```

## Delete a commit from history

```shell
# find the commit hash just before the commit you wanna delete
# syntax: git rebase -i <commit-hash>~1

git rebase -i dd48aa1b2db843fa3050c41994a1edb03165d71d~1

# change the `command` from pick to drop or d, save and quit
pick dd48aa1 message of the commit to delete

drop dd48aa1 message of the commit to delete
```

## Undo changes made in a commit

you made many changes in a commit, later you wanna to undo these changes without much effort

```shell
# syntax: git revert <hash-of-previous-commit>
# revert and commit
git revert fe3939bc2a91ec126313392cabc5777030d14d7b
```

## Hide local changes temporary

```shell
git stash
git stash pop
# keep it in list, if you need to use it multipe times
git stash apply
git stash list
```

## Stash untracked files

```shell
git stash --include-untracked
# short version
git stash -u
# stash ignored files alongside with untracked files
git stash --all
```

## Stash only one file out of multiple changed files

```shell
# for git 2.13+, the new patch option -p let us choose hunks to stash
git stash push -p -m "message"
```

## Preview revisions stashed without unstash

```shell
# check the stash list
git stash list

# check the content of a stash by index
# syntax: git stash show stash@{<index>}
git stash show stash@{0}

# add -p option to preview detail
git stash show -p stash@{0}

# or use diff to do that job
# syntax: git diff stash@{<index>}
git diff stash@{0}
```

## Clear the stash stack

```shell
# remove all entries
git stash clear

# to drop a specific one by reference
# syntax: git stash drop stash@{<index>}
git stash drop stash@{0}
# drop by index number works too
git stash drop 0
```

## Recover a dropped stash entry *

```shell
# use git fsck(file system check) to retrieve file from git database
# since our changes are stashed, so it's unreachable from any commit
git fsck --unreachable | grep commit

# for Windows, use ripgrep(rg) as an alternative to grep
# ripgrep(rg) is recommended if you work across platforms
git fsck --unreachable | rg commit

# find the commit hash to recover
# syntax: git stash apply <commit-hash>
git stash apply 01cac0dadc21a8c5331355c1b56445f1a6f2cb84
```

## Discard local changes that haven't staged

```shell
# restore changes made to a specific file
# syntax: git restore <file-path>
git restore README.md

# discard all unstaged changes
git restore .
```

## List files in conflict state

```shell
# the diff-filter value U means Unmerged
git diff --name-only --diff-filter=U --relative
git diff --check
```

## Resolve merge conflicts *

```shell
# both modified some file, but our revisions should be used
git checkout --ours <file-path>

# oth modified some file, but their revisions should be used
git checkout --theirs <file-path>

# after resolving, add to the stage area
git add <file-path>

# continue after conflicts resolved
git merge --continue

# if something is wrong and need restart resolve, abort the ongoing merge
git merge --abort
```

```shell
git config merge.tool

# idea's builtin merge tool is sophisticated and easy to use
https://www.jetbrains.com/help/idea/command-line-merge-tool.html

# choose a better conflictstyle, default is merge
git config merge.conflictstyle diff3
```

## Undo a resolved merge that hasn't been pushed yet

```shell
# git will generate a ORIG_HEAD file when do merge
git reset --hard ORIG_HEAD

# or in a safier way, unlike the --hard option, --merge won't reset uncommitted changes
git reset --merge ORIG_HEAD

# or dop the latest merged commit
git reset --hard HEAD^
```

## Check if a branch has been merged into another branch

```shell
# syntax: git branch --merged <branch-name>
git branch --merged main
```

## Merge multiple commits onto another branch as a single commit

```shell
# syntax: git merge --squash <branch-name>
git merge --squash main
```

## Sync a forked repo

You forked a repo, later new commits were added to the original repo, you may need to sync these changes to your forked repo.

```shell
# add a remote for the original repo
git remote add root https://github.com/git/git.git

# check your remote info
git remote -v

# there should be two remotes, origin & fork
origin  git@github.com:johndoe/git.git (fetch)
origin  git@github.com:johndoe/git.git (push)
root    https://github.com/git/git.git (fetch)
root    https://github.com/git/git.git (push)

# sync the original repo
git fetch root

# pull from a branch of the root remote
# syntax: git pull root/<branch-name>
git pull root/main
```

## Delete a tag both locally and remotely

```shell
# delete local tag
# syntax: git tag -d|--delete <tag-name>
git tag --delete v1.0

# delete remote tag
# syntax: git push -d|--delete <remote-name> <tag-name>
git push --delete origin v1.0
```

## Remove a submodule

```shell
# option 1
git rm <submodule-path>
git add <submodule-path>
git commit -m "your commit message"

# option 2
git submodule deinit -f <submodule-path>
# Linux/Unix
rm -rf .git/modules/<submodule-path>
# windows powershell
rm -R -Force .git/modules/<submodule-path>
git rm -f <submodule-path>
```
