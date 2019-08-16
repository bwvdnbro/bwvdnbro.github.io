---
layout: post
title: "Resolving painful git merges"
description: >-
  Some personal experiences with resolving git merge conflicts that might be
  useful for other people.
date: 2019-08-16
author: Bert Vandenbroucke
tags: 
  - Code development
  - Tools
---

[git](https://git-scm.com/) is an amazing repository system that greatly 
facilitates the management of large, collaborative code projects. It can 
store a large number of source code files and their history, and allows 
separate developers to work on the same code simultaneously, without 
constantly causing major issues when the same file gets edited by 
separate developers.

The power of git is however limited, and there will always be cases 
where even git is not able to resolve the problems that arise when you 
try to join the work done by different people into a single, coherent 
version of the code. In this post, I will give a short overview of the 
possible ways to resolve these conflicts manually, and show that git has 
some useful tools to help out with this.

# The ideal world

In an ideal world, the online repository for your code project will 
always contain a `master` branch that contains the latest working 
version of your code. All developers that want to make their own changes 
should always make these changes to the `master` branch. However, they 
should not make the changes to the branch directly, but should instead 
create separate branches in which the changes are made. Whenever they 
are happy with the changes to their branch, they should open a *pull 
request* for their branch and merge the changes they made into the 
`master` branch. The `master` branch hence acts as a central root for 
all the individual branches that contain changes that are still under 
development.

When multiple developers make changes to the code, they will hence all 
have a separate branch that branched off from the `master` at some 
point. However, as pull requests get merged back into `master`, the 
actual `master` branch can diverge from the original `master` branch 
that was the root of a specific development branch. When a pull request 
is created for such a branch, this can lead to conflicts.

In order to avoid this situation, developers should always *rebase* 
their development branch against the latest version of the `master` 
branch. During a rebase, git will *rewind* the development branch to the 
original point when the development branch branched off from the 
`master`. It will then change the `master` at that point to the current 
version of the `master`, and *replay* all the changes that were made in 
the development branch on top of that. This way, it will seem as if the 
changes made in the development branch were made directly on top of the 
latest `master`. Once this is done, their will not be any merge 
conflicts any more, and a pull request can be created and successfully 
merged.

# What can go wrong?

There are many things that happen in the real world that are far removed 
from this ideal world. First of all, there is the premise that the 
`master` branch will always contain the latest version of the code. 
Personally, I have some issues with this, as the `master` branch is also 
the default branch that gets used when a user makes a fresh clone of a 
repository. If all changes are to end up in `master`, then it would be 
very easy to break a user's workflow by making changes that affect the 
user experience. Ideally, such changes should not be rolled out as soon 
as they are made, but should be rolled out in a controlled way through 
the release of a new code version. You could of course force users to 
use a specific branch that contains a *frozen* older version of the 
code, or force them to use official releases instead of simply cloning 
the repository. But personally I prefer to use the `master` branch as a 
frozen older version, and make all changes in a specific `development` 
branch.

The second thing that could go wrong is the rebasing step required 
before every pull request with the `master` (or the dedicated 
development branch). git is generally quite good at figuring out what 
the changes are that were introduced by a specific commit, so replaying 
them should be straightforward. There are however cases where a commit 
introduces changes in a file that was also changed in the new `master`. 
To figure out where a change needs to be made, git uses a combination of 
line numbers (those can very easily change if another change is 
introduced), and unchanged code fragments surrounding the change. If an 
external change affects both the line numbers and the code surrounding a 
change, then git cannot figure out how to apply the change from the 
commit, and a conflict arises.

Solving these conflicts is usually easy, but requires manual 
intervention. Suppose for example that you are rebasing against `master` 
using the following command (issues *from* your local development 
branch):

```
git rebase master
```

If a merge conflict arises, git will always tell you about this, and 
cause the branch to end up in an incomplete state, i.e. you will not be 
able to switch branches or make any commits until the conflicts have 
been resolved.

The first step in resolving the conflicts is looking at the output of 
`git status`. This command will tell you that you are actively rebasing 
your branch, and it will also give you a full list of all the changes 
that it managed to automatically replay, and of all files that contain 
unresolved conflicts.

Unresolved conflicts can occur in various ways: both your branch and the 
new master can have added or removed a file with the same name, or (more 
likely) both your branch and the new master can have made changes to the 
same file. In the latter case, git will have made an attempt at merging 
the two files based on the limited information it has available (line 
numbers and surrounding code). It will have added the changes from both 
the branches, and will surround them with `<<<<<`, `======` and `>>>>>>` 
(this will very likely cause compilation errors; you really need to fix 
the conflicts). You will need to open the file in question in a text 
editor, locate these characters and decide what to do: keep both 
changes, choose one or the other, or create some complicated combination 
of both. Once you are done editing the file, you need to tell git that 
file is ready, by running `git add <NAME OF THE FILE>`.

If both branches added or removed a file (and the files that were added 
were the same), you probably want to keep these changes as they are. You 
will also need to tell this to git using `git add <NAME OF THE FILE>`. 
If both branches added a file, but that file is not the same, then 
things can get really complicated. By default, git will make a copy of 
both files, and append the branch name (or commit hash string) as a 
suffix to the file name. This way, you have all the necessary 
information to decide how to proceed. You will need to either choose one 
of the versions, or create your own hybrid version. Once you are done, 
you will again need to tell git about this using `git add <NAME OF THE 
FILE>`.

Note that you can always decide to completely ignore the changes made to 
a file by one of the branches and simply use the version of the other 
branch. To do this, you can run `git checkout --ours/--theirs <NAME OF 
THE FILE>`, where `--ours` means using the version of the branch 
*against which you are rebasing* (the `master` in our example), and 
`--theirs` means using the version contained in the *branch you are 
rebasing*. I personally find this convention quite confusing, so it is 
important to know what `we` and `them` means in this context.

Note that `git checkout --ours/--theirs` will *only* work for files with 
active conflicts. If the rebase did an automatic merge of another file 
and you would rather use a specific version for that file, you will need 
to manually undo the automatic merge. This is not very convenient, but 
it is unfortunately how git works in my experience.

Once all conflicts are resolved (and `git add`ed!) you can continue the 
rebase:

```
git rebase --continue
```

git will replay the commits made in the branch you are rebasing one by 
one, so you might end up having additional conflicts that need to be 
resolved after this. The rebase is complete when all commits have been 
successfully replayed.

Note that you can always opt to skip a specific commit (`git rebase 
--skip`). I have personally never done this, but I expect that this 
could be useful in some cases. And of course, you can also always decide 
to abort the rebase altogether, using `git rebase --abort`. This will 
restore the repository into its original state (but will also remove all 
the changes you made when you were resolving conflicts).

# Merging distant branches

To minimise the pain when doing a rebase, you should try to *often* 
rebase against a central branch. In large projects, a daily rebase is 
common. It is also a very good idea to only branch off from that central 
branch, so that all changes are only made with respect to that branch.

In a non-ideal real world, rebasing does not always happen that often. 
And even worse: branches do not always branch off from a central branch. 
git is so powerful that it will allow you to branch off from any other 
branch, which means that you can have a branch of a branch of a branch 
of the central branch (and even much deeper; you get the gist). And that 
makes things... *interesting*. Suppose for example that you were to 
rebase the first level branch (the one that branched off from the 
central branch) against the new master. This will effectively create a 
new branch that is no longer a direct parent for the second level 
branch. If you then try to rebase that second level branch against the 
first level branch, you might end up resolving the same conflicts you 
already solved again, as the new first level branch will no longer match 
the root of the second level branch.

Even worse things happen with *forks*, i.e. copies of a repository that 
act as an independent repository. These forks do not only contain a copy 
of the master branch of a repository, but also of all its active 
branches. If changes are made in the fork to different branches 
(independently of the changes made to those same branches in the 
original repository), then it can become almost impossible to merge 
those changes back into the original repository, especially if you do 
not know through which intermediate branches the changes were made.

To deal with these issues, there are a number of options. The first one 
is to do an actual *merge*. While a rebase tries to replay the changes 
you made on top of the `master` (rewriting the git history for the 
branch in the process), a merge will simply figure out what the 
differences are between two branches, and will try to apply those 
differences as new changes. In a way, this is a clearer situation for 
you as a developer to deal with, as you will immediately see all 
conflicts, and you will be able to make all the changes required to 
resolve the entire merge. On the other hand, this will probably be a 
very painful step, and it will also lead to a loss of the detailed 
history of the branch that is merged in; all changes made in the branch 
will show up as a single commit in the central branch.

The second option is to try to figure out what the *branch history* of 
the two branches you want to merge is. Unless someone forcefully changed 
the git history, two branches from the same repository or from a 
repository and a fork of that repository can always be traced back to 
some common ancestor. Once you find that ancestor, you can figure out 
how they diverged from that ancestor. You can then do a step-wise rebase 
of one of the branches against that ancestor (subsequently rebasing each 
of the intermediate branches until you have a clean git history from the 
ancestor to your branch), and then do a step-wise rebase of that new 
branch against the ancestors of the other branch. If you do this well, 
then this should be feasible without being too painful.

If the situation is really bad and you are only interested in merging 
*some* changes from one branch into the other one, you can also consider 
using a *patch*. A patch is a textual representation of the changes 
introduced by a commit (or a number of commits), similar to the output 
you get from `git diff`. git has a number of commands that can generate 
such representations (`git format-patch` is the most appropriate one). 
Once you have a patch, you can apply that patch to *any* branch using 
`git apply <NAME OF PATCH FILE>`. git will then try to make the changes 
described in the patch to that branch, independent of the history of the 
branch and the history of the branch that generated the patch. While 
this will likely fail for branches that have drifted too far apart (and 
for patches that contain a lot of changes), it might work to recover 
small changes to files that did not change dramatically between the two 
branches.

# Conclusion

git is very powerful, and so it also contains all the tools you would 
ever need to resolve even the most desperate merge conflicts. Some of 
these tools can be very tricky to use, but if used correctly, they will 
help you out.

So if you end up in a seemingly impossible merge chaos, remember:
 - don't panic! You can always abort a merge or rebase.
 - a git commit is simply a list of changes to a specific state of the 
repository. If you know what the original state of the repository was to 
which those changes were applied, you can always replay those changes on 
top of another state that was based on that same state. The worst merge 
conflicts arise when you try to merge git histories that do not derive 
from a common ancestor; in this case you should try to find a common 
ancestor and make sure your git histories agree on that.
 - you can always recover specific changes using a patch, which could be 
a lot less painful and equally useful as doing a full merge.

And of course, as long as all your changes are stored in a repository, 
you can always manually copy and paste them across branches. But if you 
need to revert to that, you are probably not using git correctly.
