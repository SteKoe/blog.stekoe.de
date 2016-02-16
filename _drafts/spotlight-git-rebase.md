---
title: "Spotlight: Git rebase"
tags:
- git
- rebase
- merge
- feature
- branching
---
In this article I write about the common problem one faces when working with feature branches.
At one point in time one wants to reintegrate a feature (and the corresponding branch) back into the current development branch (mostly called *master* or *development*).
Git offers two different approaches for merging branches:
The simplest one is a `git merge` which creates a new commit containing the complete diff of the two branches.
The other but more complex one is a combination of `git merge` and `git rebase`.
These two commands in combination allows a user to merge not just only the file diffs but also preserves the complete history of the feature branch which is helpful in most cases since the commit messages contain relevant information as well.

<!-- more -->

In order to set up a Git repository having two branches and some commits I have created a small bash script which creates some files and commits to the different branches.
Using this setup I have played around using `git rebase` command to illustrate how this command actually works. Figure 1 shows the commit graph which is the result when one executes the bash script.

{% include image.html file="2016-02-07-spotlight-git-rebase/git-demo-repo-initial.png" %}

At this point we assume that the feature is fully implemented and is about to be reintegrated into the master branch.
The big question now is: How does one have to call the `git rebase` command.
Figure 2 and Figure 3 show the different results depending on how `rebase` is applied.
When the `rebase` command is called, Git looks for a common ancestor in the commit graph and uses this as basis for merging all later commits.
In this example *M1* is the last common commit of both branches and will be used as starting point.
Now one has to know how `rebase` actually works: `rebase` always appends a branch to the end of commits which is currently checked out.
Hence, when one wants to reintegrate a feature branch, one has to checkout the feature branch first and then call `git rebase master`.
When I call the `rebase` command I add an imaginary "*to*" to the call: `git rebase (to) master` which &mdash; in my opinion &mdash; makes the way `rebase` is working clearer. The command tells Git onto which branch the current branch shall be applied (cf. Figure 3).

The feature branch now has all preceding commits from master branch reinte

| Fig. 2: Rebase feature branch         | Fig. 3: Rebase master branch   |
| ------------------------------------- | ------------------------------ |
| <img src="/images/2016-02-07-spotlight-git-rebase/git-demo-repo-rebase-feature.png"> | <img src="/images/2016-02-07-spotlight-git-rebase/git-demo-repo-rebase-master.png"> |

## Conclusion
Git's `rebase` command is a nice way to preserve commits of a feature branch and allows to add all commits to the development branch without loosing them. This helps to reduce the amount of branches as rebased branches can be safely removed.

## Appendix: Bash script
```bash
#!/bin/bash
function fakeCommit {
	echo $1 >> 1.txt
	git add .
	git commit -m "$1"
}

## init git repo
git init git-demo-repo
cd git-demo-repo

## do commits and switch branches
fakeCommit "master 1"
fakeCommit "master 2"

git checkout -b feature_branch
fakeCommit "feature 1"

git checkout master
fakeCommit "master 3"

git checkout feature_branch
fakeCommit "feature 2"
fakeCommit "feature 3"

git checkout master
fakeCommit "master 4"
```
