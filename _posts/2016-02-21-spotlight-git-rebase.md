---
title: "Spotlight: Git rebase"
layout: post
tags:
 - Git
keywords:
 - git-rebase
 - git-merge
---
This article is about the common problem one faces, when working with feature branches.
At one point in time one wants to reintegrate a feature (branch) back into the current development branch (often called *master* or *development*).
Git offers two different approaches for this:
The simplest one is a `git merge` which creates a new commit containing the complete diff of the two branches.
The other but more complex one is a combination of `git rebase` and `git merge`.
These two commands in combination allows a user to merge not just only the file diffs but also preserves the complete history of the feature branch which may be helpful in most cases since the commit messages contain relevant information as well.

<!-- more -->

In order to set up a Git repository having two branches and some commits I have created a small bash script which creates some files and commits to the different branches.
Using this setup I have played around using `git rebase` command to illustrate how this command actually works. Figure 1 shows the commit graph which is the result when one executes the bash script.

{% include image.html file="initial-repo-structure.svg" caption="Fig. 1: Initial commit history of demo repository" %}

At this point we assume that the feature is fully implemented and is about to be reintegrated into the master branch.
The question now is: How does one have to call the `git rebase` command.
Figure 2 and Figure 3 show the different results depending on which branch the `rebase` command is applied.
When the `rebase` command is called, Git looks for a common ancestor in the commit graph and uses this as basis for merging all later commits.
In this example *M2* is the last common commit of both branches and will be used as starting point.
Now one has to know how `rebase` actually works: 
`git rebase` always appends all commits of a branch to the end of commits of the branch which is currently checked out.
Hence, when one wants to reintegrate a feature branch, one has to checkout the feature branch first and then call `git rebase master`.
When I call the `rebase` command I add an imaginary "*to*" to the call: `git rebase (to master)` which makes the way `rebase` is working clearer. 
The command tells Git onto which branch the current branch shall be applied (cf. Figure 3).

## git rebase feature
{% include image.html file="git-rebase-feature.svg" caption="Fig. 2: Rebase feature branch (to master)" %}
This example assumes, that the master branch is checked out. 
Hence, the commits after *M2* are cut off and appended one by one to the very last commit of the feature branch.

## git rebase master
{% include image.html file="git-rebase-master.svg" caption="Fig. 3: Rebase master branch (to feature)" %}
In this example we assume, that the feature branch is checked out.
Thus, the commits of the feature branch made after *M2* are cut off and appended one after the other to the last commit of the master branch.

## Conclusion
Git's `rebase` command is a nice way to preserve commits of a feature branch and allows to add all commits to the development branch without loosing them. 
This helps to reduce the amount of branches as rebased branches can be safely removed.
But using `rebase` might also mess up your history, since the whole history (at least the timestamps) are reset.
Thus, `rebase` is dangerous as well when used the wrong way.

Nevertheless, using `rebase` over `merge` (espacially a fast-forward merge) always depends on the current use-case.
In some cases it might be useful to preserve the branch commit history, then one should rely on `rebase`.
When the history is irrelevant or contains messages which should not be part of the later result, using `git merge` might be a better choice.

## Appendix: Bash script

{% gist 26e291331eb5eabb145b %}
