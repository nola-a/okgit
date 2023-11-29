# Table of contents

- [How To Read this document](#how-to-read-this-document)
  - [Basics items](#basics-items)
    - [Item 1: what's happening: git status](#item-1-whats-happening-git-status)
    - [Item 2: how to commit: git commit](#item-2-how-to-commit-git-commit)
    - [Item 3: references update: git fetch](#item-3-references-update-git-fetch)
    - [Item 4: differences between references](#item-4-differences-between-references)
    - [Item 5: keep safe your contributions](#item-5-keep-safe-your-contributions)
  - [Intermediate items](#intermediate-items)
    - [Item 6: the simplest Git workflow: feature branch](#item-6-the-simplest-git-workflow-feature-branch)
    - [Item 7: my push was rejected, what can I do?](#item-7-my-push-was-rejected-what-can-i-do)
    - [Item 8: a dry run approach to git merge](#item-8-a-dry-run-approach-to-git-merge)
    - [Item 9: how to approach conflicts](#item-9-how-to-approach-conflicts)
    - [Item 10: why sometimes fast-forward is better](#item-10-why-sometimes-fast-forward-is-better)
    - [Item 11: restore a repo to a specific commit: the right way](#item-11-restore-a-repo-to-a-specific-commit-the-right-way)
    - [Item 12: spot a problem: force updated](#item-12-spot-a-problem-force-updated)
  - [Advanced items](#advanced-items)
    - [Item 13: update last commit: git amend](#item-13-update-last-commit-git-amend)
    - [Item 14: rebuild history: rebase](#item-14-rebuild-history-rebase)
    - [Item 15: rebuild history #2: interactive rebase](#item-15-rebuild-history-2-interactive-rebase)
    - [Item 16: Git insights: git reflog](#item-16-git-insights-git-reflog)
    - [Bonus Item: happy git!](#bonus-item-happy-git)

# How To Read this document

This document wants to be a reference for best practices in the daily use of Git, despite the complexity of this tool it is worth mentioning that Git is a tool that tracks the changes in a repository (which is basically a directory containing a .git folder that has all the needed metadata to achieve file tracking).

Some concepts are stressed by the author for a simple reason: the main goal of Git is tracking the changes and years of experience have shown that Git is very good at that. However, this is not for free, it requires some attention by Individual Contributors (the engineers who commit changes, henceforth referred to as IC).

The history of a Git repository shows all the contributions and if ICs don’t follow some simple rules, it can easily turn into a mess, in fact losing the primary goal: tracking changes effectively.

Moreover, having a good history allows for some tasks, like understanding what happened, learning how to implement something and helping the reviewers with Pull Requests.

Git, like other tools, was designed to allow several ICs to work together efficiently.

This document is divided into items, which are designed to explain the pros and cons of every method.

To get the best out of this guide a little knowledge of command line tools is needed (also Git) so the reader, who is not proficient with those is strongly invited to read [https://git-scm.com/book/en/v2](https://git-scm.com/book/en/v2) (especially the chapters 1. Getting Started, 2. Git Basics, and Git Branching).

Three categories are presented: basic, intermediate and advanced, while the first two show safe practices the last one shows how to rewrite the history, which is forbidden when the branch is shared with other ICs but it can be safe when working alone (e. g. local branches).


## Basic items

A very quick introduction to how a Git repo works: code contributions are grouped into commits, but what is a commit? It is just a patch https://en.wikipedia.org/wiki/Patch_(computing) whose hash function (currently sha1sum but with ongoing discussions to upgrade it) is referred to as the commit ID.

The history can be seen as a series of commit IDs, each of them brings changes to the source code: this structure is clear and safe to all its users and guarantees a strict policy against unwanted changes.

A branch is a reference to a specific commit ID and therefore to all its ancestor commits. ICs are free to decide whether to tag or not a specific commit (usually for tracking the release version).
![Scheme](images/simplerepo.png)


### Item 1: what's happening: git status
Git status is the most powerful and simple command that shows what is happening in the repo. But it does more than that; in fact, it suggests what you can do hinting any possible command. Let's look at the examples below to see the most common scenarios:

#### Nothing to do
```bash
~/repos/testrepo$ git status
On branch develop
Your branch is up to date with 'origin/develop'.

nothing to commit, working tree clean
```
#### There is some changes in the local branch
```bash
~/repos/testrepo$ git status
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
       	modified:   new.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)
       	images/simplerepo.png

no changes added to commit (use "git add" and/or "git commit -a")
```
Git is telling us that on branch master:

- Some changes in the file new.md are ready to be staged.
- There is a new file called simplerepo.png that can be added.

#### There are some updates in the remote branch.
```bash
~/repos/testrepo$ git status
On branch develop
Your branch is behind 'origin/develop' by 2 commits, and can be fast-forwarded.
  (use "git pull" to update your local branch)

nothing to commit, working tree clean
```
Git is telling us that the branch develop which is tracking the remote branch origin/develop is behind by 2 commits. That is, some IC has added its own contributions, C3 and C4.

![Scheme](images/ff1.png)

The current reference develop can be fast-forwarded in order to point to C4 with:
```bash
$ git pull
```
or
```bash
$ git merge origin/develop
```
The command git pull is a shorthand for git fetch && git merge origin/develop -> see item 3 for insights on git fetch

### Item 2: how to commit: git commit
When all the changes are steady and the tests have passed (e.g., mvn clean install is OK), It is time to stage the changes, to do that
```bash
$ git add -p
```
for each snippet Git prompts you different choices asking whether the change can be added or not.
```bash
$ git commit
```
A well formatted commit message should contain:
- Title of the commit and if available the ticket ID
- A blank line
- A brief summary of the changes that occurred and meaningful comments for helping future reviews

### Item 3: references update: git fetch
Since Git is a distributed version control this means that more copies of the same repository could exist(ideally one for IC plus one as central). One of them will be considered as the central repository (the one called origin), therefore the repo contains references to both local and remote branches; in order to update local references to the remote branch you need to use git fetch:
```bash
$ git fetch
~/repos/testrepo$ git fetch
remote: Counting objects: 200, done.
remote: Compressing objects: 100% (139/139), done.
remote: Total 200 (delta 76), reused 3 (delta 1)
Receiving objects: 100% (200/200), 34.46 KiB | 1008.00 KiB/s, done.
Resolving deltas: 100% (76/76), completed with 21 local objects.
From https://foobar.com.com/scm/path/testrepo
   1ffa340..71b62d8  feature/feature1-develop -> origin/feature1-develop
   e59d286..e3ffe3c  feature/feature2-develop -> origin/feature2-develop
```
It is worth pointing out that fetch doesn't update nothing, only the references get updated: it is always safe to run git fetch.
In the above example we noticed that two branches received updates in the syntax: oldsha..newsha localbranch -> remote tracked branch

### Item 4: differences between references
Anytime you can check on the differences between the two references using git diff, remember that a Git reference could be a TAG, a commit ID or a branch name

```bash
$ git fetch
$ git diff # show the contributions that are not yet in staged area
$ git diff HEAD 1.0.0 # show differences between local branch and tag 1.0.0
$ git diff develop origin/develop # show differences between local branch and tag 1.0.0
$ git diff develop origin/master # show differences between local branch and remote master
```
The output format is the one used by Patch:
```bash
~/repos/gitcourse$ git diff 008323949d8fe0af977af8980e5f3c0d9d0c6b07 f2de6c316b2ad361ede73461d25c260642e87108
diff --git a/README.md b/README.md
index 5e8fe7e..c101e79 100644
--- a/README.md
+++ b/README.md
@@ -52,11 +52,11 @@
 #### Advanced Commands
 - git reflog
 - git reset --hard origin/master
-- git reset 'commit'
+- git reset <commit>
```
### Item 5: keep safe your contributions
Let's suppose you are working on a long task which could take one/two weeks or even more, there are a bunch of reasons to commit and push very often:
- To prevent loosing your work in case of PC failures
- It is easy sharing contributions between ICs
- The reviewers can get on with the work
- Safely pause a task, switch on another activity to resume it later

## Intermediate items

### Item 6: the simplest Git workflow: feature branch
Supposedly we are asked to work on a feature named find button, so we need to checkout a branch from the mainline
```bash
$ git checkout develop
$ git pull #just to be sure that our branch is updated with remote
$ git checkout -b feature-findbutton # create feature branch
$ git push --set-upstream origin feature-findbutton # push branch to remote
```
Implementing the feature day 1
```bash
$ git fetch && git merge origin/develop # take contributions from mainline and fix conflicts
$ vim file1 #edit file
$ git commit -am 'C1'
$ git push origin feature-findbutton
```

Implementing the feature day N
```bash
$ git fetch && git merge origin/develop # take contributions from mainline and fix conflicts
$ vim file1 #edit file
$ git commit -am 'CN'
$ git push origin feature-findbutton
```
And finally when the feature is ready (optional)
```bash
$ git checkout develop
$ git merge --squash --ff-only feature-findbutton
$ git commit -m 'Merge pull request #feature-findbutton'
$ git push origin develop
```
The last bunch of commands were tagged optional because using Pull Request on bitbucket/github/gitlab needs the approval from reviewers and only when accepted, a merge button will come on to the UI, which replaces all the previous git commands.

### Item 7: my push was rejected, what can I do?

```bash
~/repos/testrepo$ git push
To https://foobar.com.com/scm/user/testrepo.git
 ! [rejected]        master -> master (fetch first)
error: failed to push some refs to 'https://foobar.com/scm/user/testrepo.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```
as usual Git is telling us why the updates were rejected

![Scheme](images/conflict1.png)

Some ICs pushed to remote the commit C3 and now we are trying to push the commit C4. The remote repository is expecting to also have the changes that C3 brought in before.
In this case we have two options available: we can follow what Git suggests us (see above) or we can use the Git rebase command.
The main difference between the two actions is that using Git merge brings to the tree the merge commit which represents how the merge was done (sometimes it is done just for tracking purpose)

#### Pull approach
```bash
$ git pull # or git fetch && git merge origin/develop
$ git push
```
![Scheme](images/conflict2.png)

Looking from CM to left, the history contains all commits including the merge commit

#### Rebasing approach
```bash
$ git rebase origin/develop
$ git push
```
![Scheme](images/conflict3.png)

Unlike the merging strategy, rebasing rewrites the current history putting on top our commit. The previously introduced commit will be rehashed, resulting in a history change.

### Item 8: a dry run approach to git merge
Sometimes you would want to check on the side effects of a merge, without actually making it. This could be achieved by creating a temporary branch and merging it for testing purpose only. Whenever you push this branch remotely, you could share it with other ICs.

Suppose the target branch is develop and the feature branch is feature1

```bash
$ git checkout feature1
$ git checkout develop
$ git checkout -b develop-try-merge
$ git checkout merge feature1
```
Once the merge is done, the resulting outcome will be:

```bash
$ git checkout develop
$ git checkout merge develop-try-merge
```
Even though this approach might appear complex, just consider this use case: you are in the middle of merge (taking something like 2 days) using this strategy, you can pause this task anytime to do something else (e.g. facing a production issue) and you could resume it later on.

### Item 9: how to approach conflicts
Merging and rebasing might cause some issues, such as conflicts. In brief, it means that two commits are changing the same line, and Git was not able to merge them applying the default merge strategy, so it is asking the IC to manually solve the conflict.
Again, we could take advantage of the git status command to list the files affected by conflicts. Once all the conflicts are resolved, we might proceed compiling and testing the resulting source code.
```bash
~/repos/testrepo$ git pull
Auto-merging Minor fix
CONFLICT (content): Merge conflict in Minor fix
Automatic merge failed; fix conflicts and then commit the result.
```
```bash
~/repos/testrepo$ git status
On branch master
Your branch and 'origin/master' have diverged,
and have 1 and 5 different commits each, respectively.
  (use "git pull" to merge the remote branch into yours)

You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Changes to be committed:
       	new file:   file1.txt
       	new file:   file2.txt

Unmerged paths:
(use "git add <file>..." to mark resolution)
     	both modified:   file3.txt
```
In fact, opening file3.txt, you will find this content:

```bash
~/repos/testrepo$ cat file3.txt
<<<<<<< HEAD

this line 1
=======
this line 1
this line 2
this line 3
>>>>>>> ab8106e6b692ff14d5b6fa345021c7e61ceeefda
```
Once file3.txt is edited, just run (as suggested above by git status):
```bash
$ git add file3.txt
$ git commit
```

### Item 10: why sometimes fast-forward is better
The picture below explains why we chose a fast forward merge strategy when a feature branch is going to be merged into mainline develop. If the feature branch is merged with step 6 then all the contributions contained in feature1 were not tested against what c4 has brought. To prevent that one, step 7 is the right approach in fact feature1 can be merged only if no new contributions are committed in develop since the last merge of develop into feature1

![Scheme](images/devsvil.png)

### Item 11: restore a repo to a specific commit: the right way
Let's suppose our repo
![Scheme](images/restore1.png)

And we want to restore the repo to the C2 commit using the following commands:
```bash
$ git rm -rf *
$ git checkout C2 . #note the dot at the end of git checkout
$ git commit -m 'Restore to C2'
```
![Scheme](images/restore2.png)

There are two main reasons to go for 'the right way' :
- The history remains consistent (no rewriting)
- All the changes are stored in the same commit that could easily be reverted by:
```bash
$ git revert CX
```
### Item 12: spot a problem: force updated
```bash
~/repos/pippo11$ git fetch
remote: Counting objects: 9, done.
remote: Compressing objects: 100% (9/9), done.
remote: Total 9 (delta 6), reused 0 (delta 0)
Unpacking objects: 100% (9/9), 962 bytes | 7.00 KiB/s, done.
From https://foobar.com.com/scm/user/testrepo
 + 38b166c...c2b0abe master     -> origin/master  (forced update)
```
The last line indicates that there was an update on the master branch, the big issue is that the update was forced which means the history of that branch was rewritten. Generally origin repo is configured to avoid this kind of situations on important branches such as master/develop, but it is likely to happen on short-lived feature branches. It is important to remember that each time someone rewrites the history it is compromised for the others to commit on the same branch, which cannot be done anymore (unless resetting the branch version from the beginning)

## Advanced items

### Item 13: update last commit: git amend
Let's suppose you just committed (you haven't pushed yet) and then you remember you left something out, it is time to amend:
```bash
$ git commit --amend
```
Keep in mind that its usage only applies to the local commits, in all of the other cases you're going to rewrite the Git history causing serious issues to the other commiters.

### Item 14: rebuild history: rebase
We already saw this function in action on the item 6, now we can add something more to what we already know. Rebasing is the main feature offered by Git for rewriting history:

```bash
$ git checkout existing_feature_branch
$ git rebase develop
```
In a few words, we asked Git to rewind the history of that current branch to the first common ancestor commit and only then all the commits of the current branch will be applied.
In case of unsuccessful rebase Git will ask us to solve the eventual conflicts (as usual Git hints you the resolution commands which are the same as the merge conflicts)

### Item 15: rebuild history #2: interactive rebase
Git gives you more options rebasing, the most important one is the interactive rebase, which consists in a rebase with the opportunity to rewrite history for example melting multiple commits into a single one, deleting or editing specific commits which results in a more compact and clear history.
```bash
$ git rebase -i
```
### Item 16: Git insights: git reflog
Git tracks all commands executed on the local repository. All the operations can be shown by the reflog command
```bash
~/repos/core-services-cmlt-v1$ git reflog
5d80c9b (HEAD -> foobar, origin/develop, develop) HEAD@{0}: checkout: moving from develop to foobar
5d80c9b (HEAD -> foobar, origin/develop, develop) HEAD@{1}: pull: Fast-forward
bac5fe9 (origin/feature/feature22) HEAD@{2}: reset: moving to origin/develop
e80b719 HEAD@{3}: commit: Blabla
7c4bc6c HEAD@{4}: pull: Fast-forward
62b994f (tag: v1.0.0-rc2) HEAD@{5}: pull: Fast-forward
2e90498 HEAD@{6}: commit (amend): Minor fixes
8650e50 HEAD@{7}: rebase (continue) (finish): returning to refs/heads/develop
8650e50 HEAD@{8}: rebase (continue): Minor fixes
6fa2786 HEAD@{9}: rebase (start): checkout origin/develop
96a9246 HEAD@{10}: commit (amend): Minor fixes
0cd6399 HEAD@{11}: commit: Minor fixes
4c72dcf HEAD@{12}: pull: Fast-forward
```
It is also possible restoring the local repo to a specific point for example:

```bash
$ git reset --hard e80b719
```
### Bonus Item: happy git!
Applying all of the previously described rules and testing/compiling correctly your local work might prevent introducing bugs/issues into the main branch. Whenever the code won't compile or the tests will break, it means that there is something wrong on your local branch. Always double check before asking for help or claiming bugs in develop.
