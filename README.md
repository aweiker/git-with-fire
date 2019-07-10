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

## Lorem Ipsum

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Curabitur sapien lectus, convallis et rutrum sit amet, porta et ligula. Integer neque sem, ornare et tellus a, pellentesque posuere nulla. Lorem ipsum dolor sit amet, consectetur adipiscing elit. Mauris id libero sit amet tellus venenatis congue. In nibh erat, congue at enim sed, iaculis pharetra nibh. Donec eu felis turpis. Quisque ut ex eget nibh consequat accumsan vel sed nibh. Aenean ante ex, vehicula ut tincidunt ac, volutpat in lorem. Quisque eu euismod neque. Vivamus ullamcorper turpis vel blandit accumsan. Sed placerat elit nec justo lobortis, quis aliquet sem feugiat. Aenean gravida lacinia volutpat. Fusce iaculis ipsum et mattis pellentesque. Mauris augue libero, ullamcorper in luctus at, ultrices quis sapien. Sed in est vitae arcu suscipit mattis a interdum nisl. Nunc mattis vestibulum enim id mattis.

Vivamus nec scelerisque lorem. Proin metus tortor, tempus ut diam ac, bibendum aliquam purus. Ut luctus nisl condimentum sodales accumsan. Nam id tellus quis orci laoreet lobortis. Fusce ultrices ornare lectus, ac facilisis nisl. Mauris eleifend felis urna, at consequat purus faucibus nec. Praesent vulputate, lectus quis euismod cursus, erat lectus iaculis urna, eu fermentum lectus tellus ut erat. Integer in nulla vel neque sagittis lacinia in in ligula. Morbi semper diam sed eleifend efficitur. Proin suscipit ultrices ipsum ac faucibus. Orci varius natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. Vestibulum aliquam massa at mi dignissim, ut consectetur risus dignissim. In elit leo, lacinia nec enim eget, congue commodo nisi. Aliquam pharetra suscipit finibus. Integer convallis urna vitae justo sodales, et pretium tellus tincidunt. Nam porttitor tellus non nibh lobortis sodales.

Cras maximus luctus sapien, ut sollicitudin nulla facilisis ullamcorper. Maecenas tristique varius erat a egestas. Curabitur dictum odio vitae neque pellentesque condimentum. Integer ut volutpat massa, venenatis volutpat quam. Integer vestibulum feugiat quam, at placerat nunc sagittis eu. Phasellus nunc dolor, ullamcorper eu sodales ut, pretium id tortor. Aliquam imperdiet mi in odio cursus consequat. Pellentesque mattis porta sem tristique rhoncus. Pellentesque ut ipsum sed ex ornare placerat. Morbi imperdiet nec risus tincidunt laoreet. Sed pellentesque accumsan dolor sed maximus.

Nam aliquam tincidunt arcu, vel bibendum metus. Duis efficitur lobortis felis sit amet mollis. Nunc nec ultrices magna. Etiam ac elit enim. Curabitur in diam ipsum. Praesent in lacus cursus, consectetur diam vitae, commodo massa. Donec vulputate velit id leo tempus consequat. Ut eu erat nisl. Pellentesque enim tortor, dignissim vel arcu a, gravida fermentum eros. Phasellus ut viverra nisi.

Donec ut pharetra augue, sit amet efficitur ex. Integer convallis diam sit amet mauris mollis placerat ut at mauris. Cras quis mi dolor. Nam ac mauris non dolor efficitur porta. Cras eget lorem in metus efficitur placerat. Nunc nisl metus, bibendum in tortor ultricies, faucibus euismod mi. Curabitur elementum mollis augue at facilisis. Curabitur semper magna ut purus aliquam posuere. Vestibulum eu sem vitae lorem sagittis sodales.
