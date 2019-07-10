# Advanced GIT Tips and Tricks

## Removing commits from a branch

There are times when changes are put into a branch and then later on it is decided that a change should be removed.

```sh
# The branch to remove commits from is features/branching
# and the parent of this branch is master

git checkout features/branching
git rebase --interactive master
```

This will put you in your default `$EDITOR`, by default `vi` editing a `git-rebase-todo` file. Here is an example of such a file.

```text
pick 3ab36d5 Starting tutorial for git
pick cac3f6f Add branching tutorial
pick 7a2f375 PLEASE REMOVE THIS COMMIT
pick 5ae6870 More work on the tutorial

# Rebase a3cde16..5ae6870 onto a3cde16 (4 commands)
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

In order to remove the commit `7a2f375`, remove the word `pick` and instead insert either a `d` or `drop` so that the file looks like:

> For navigating inside of `vi`, use your arrow keys. To delete the word `pick`, use the keys `dw` which will _delete the word_ that the cursor is currently in. Then type `i` to go into _insert mode_ and then type `drop ` and hit the `escape` key to exit _insert mode_. Then type `wq` to write the current buffer to the file and then quit vi.

```text
pick 3ab36d5 Starting tutorial for git
pick cac3f6f Add branching tutorial
drop 7a2f375 PLEASE REMOVE THIS COMMIT
pick 5ae6870 More work on the tutorial
```

Now if you type `git log --oneline` you will see that this commit has been removed.

```text
3134e4e (HEAD -> features/branching) More work on the tutorial
cac3f6f Add branching tutorial
3ab36d5 Starting tutorial for git
a3cde16 (master) First commit
```

But what if you didn't want to lose that commit? Don't worry, it isn't lost, you just need to put it somewhere.

```sh
git checkout master
git checkout -b wip/lost-commits
git cherry-pick 7a2f375

# error: could not apply 7a2f375... PLEASE REMOVE THIS COMMIT
# hint: after resolving the conflicts, mark the corrected paths
# hint: with 'git add <paths>' or 'git rm <paths>'
# hint: and commit the result with 'git commit'
```

You will receive an merge conflict when you do this. So let's walk through the resolution of this.

```sh
git status
# On branch wip/lost-commits
# You are currently cherry-picking commit 7a2f375.
#  (fix conflicts and run "git cherry-pick --continue")
#  (use "git cherry-pick --abort" to cancel the cherry-pick operation)
#
# Unmerged paths:
#   (use "git add <file>..." to mark resolution)

#         both modified:   README.md
#
# no changes added to commit (use "git add" and/or "git commit -a")


# This will open up your default merge tool. It is outside the scope
# of this tutorial to cover merge resolution.
git mergetool
git add README.md

# This will resume the cherry-pick and complete restoration of changes
git cherry-pick --continue
```

If at anytime you need to review what commits may exist, you can use `git reflog` to view the local copy of reference logs contained inside of git. Here is an example view for this repository.

```plain
0272b64 (HEAD -> wip/lost-commits) HEAD@{0}: commit (cherry-pick): PLEASE REMOVE THIS COMMIT
a3cde16 (master) HEAD@{1}: checkout: moving from master to wip/lost-commits
a3cde16 (master) HEAD@{2}: checkout: moving from features/branching to master
3134e4e (features/branching) HEAD@{3}: rebase -i (finish): returning to refs/heads/features/branching
3134e4e (features/branching) HEAD@{4}: rebase -i (pick): More work on the tutorial
cac3f6f HEAD@{5}: rebase -i (start): checkout master
5ae6870 HEAD@{6}: commit: More work on the tutorial
7a2f375 HEAD@{7}: commit: PLEASE REMOVE THIS COMMIT
cac3f6f HEAD@{8}: commit: Add branching tutorial
3ab36d5 HEAD@{9}: commit: Starting tutorial for git
a3cde16 (master) HEAD@{10}: checkout: moving from master to features/branching
a3cde16 (master) HEAD@{11}: commit (initial): First commit
```

## Squashing Commits

Squashing a commit may be done while preparing a branch for a Pull Request so that the commit history is clean and proper commit messages are used.

Squashing a commit is very similar to dropping commits from history, instead of dropping though a `squash` is performed.

Using the scenario of preparing the branch `features/branching` for a Pull Request, I want to clean up the commit messages and also squash everything down into a single commit. Doing this is as simple as performing an interactive rebase against master.

```sh
git rebase --interactive master
```

This will bring up the `git-rebase-todo` file in your preferred editor. Make the following changes to sqash everything down into one commit.

```plain
pick 3ab36d5 Starting tutorial for git
squash cac3f6f Add branching tutorial
squash 3134e4e More work on the tutorial
```

After this file is written, a new file is opened that will let you modify the commit message that is used for the squashed commit.

```plain
Create a tutorial for working with branches.

This tutorial additionally includes frequently used techniques for
keeping branches clean:
  - Rebase
  - Squash
  - Branch Bankruptcy
```

This will now leave two total commits in this line of the graph.

```plain
* b398824 (HEAD -> features/branching) Create a tutorial for working with branches.
* a3cde16 (master) First commit
```

Please note that this will require a force push in order to update the remote which will then require everyone with a local copy of the branch to delete and re-fetch.

## Branch Bankruptcy

Sometimes the history of a branch is so dirty that the only recourse is to start over. While on the surface it sounds hard to get into this situation, but a common reason for this happening is using the GitHub web interface to update a PR against the target when it becomes out of date.

The first few times you declare branch bankruptcy you will want to make a copy of your branch. The good news is you can do that simply by pushing your changes to the remote.

```sh
git push origin features/bankruptcy
```

Or if you are really worried, let's work inside of a new branch. In this case let's create a new `features/life-is-good` branch and then do a soft reset that will make git think our parent is the `master` branch.

```sh
git checkout -b features/life-is-good
git reset --soft master
```

A soft reset does not change the local files, instead it just makes the current branch a straight line to the parent and then flag all of the local files as being modified.

At this point you get to add whatever changes you like and create commits. This is your clean start to make any history you want inside of the branch.
