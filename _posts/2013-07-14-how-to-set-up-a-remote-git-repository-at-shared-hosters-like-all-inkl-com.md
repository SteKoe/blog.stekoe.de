---
layout: post
title: "How to set up a remote git repository at shared hosters like All-Inkl.com"
date: 2013-07-14
tags:
- git
- init
- bare
- ssh
---
Github has some great features and I do not want to miss them, but I am not willing to pay $7 per month for a "micro" account in order to set up a private repository.
Well, the $7 do not contain just the private repository but also a wiki, issue tracker and so on, but this is not what I wanted to have.
My plan was to have a hidden repository which is not visible to the public but usable by me – and only by me.

Hence, I started to investigate wether my (already existing) webhoster allows to create git repositories on the shared servers.
As All-Inkl.com (this is my current hoster) already offers SSH access in the so called "Private Plus" package I own, I just typed the common command <code>git</code> into the shell.
And guess what? It just worked.
So I started to create my own private git repository via SSH on my own:

<!--more-->

The first step was to set up the server properly.
As my hosting package mentioned above is a webhosting package, all data in my home folder located at <code>/www/htdocs/stekoe</code> is accessible via a browser and url.
In order to avoid others to access the git repository, I moved all my homepage related files to a new subdirectory <code>www</code> and configured the urls to point at this directory instead of the home directory.
This was the first step to do.

Now I started to build a directory structure for the git repository (or repositories – always think of upcoming requirements).
I quickly decided to create another subdirectory just aside of the new <code>www</code> directory named <code>git-repos</code>.
This directory will contain all my private repos from now on!

The directory structure looks like this now:

~~~text
/www/htdocs/stekoe/www        <-- Containing my websites
/www/htdocs/stekoe/git-repos  <-- Containing private git repos
~~~

After changing into the <code>git-repos</code> directory, I created a new subdirectory for my first project.
Let us call it <code>myPrivateRepo</code> in this article.
Inside the newly created folder I had to create a new git repository.
As this repository is just about to store and versioning my code without having an own working copy and therefore no work tree (unversioned or files with changes).
In order to create such a "bare" repository just fire the command <code>git --bare init</code>.
Git will now initialize a new and empty git repository in the directory <code>myProvateRepo</code>.
That's it.
We are done here on our server.

## Using a new local repository
The next step is to create a local(!) git repository on your Computer – you may also use an existing one (that's what I did).
But let us start from scratch: Create a new folder anywhere on your hard disc where you want to have your working directory for your awesome project.
I will do this on my desktop: <code>mkdir awesomeProjekt</code>.
Now we want to check out the repository on our server.
Let us do it: <code>git clone ssh://<ssh-user>@stekoe.de/www/htdocs/stekoe/git-repos/myPrivateRepo</code>.
You see one important thing: the path after the url hast to be the absolute path on filesystem to your git repository! It took me several tries to figure this out...

Git clones the remote repository to your computer now and will finish with a warning:

> warning: You appear to have cloned an empty repository.</blockquote>

That's right.
Our git repository on our server is empty, but we will now do our first commit.
Change into the directory <code>myPrivateRepo</code> which has been created by the clone command of git and create an new file.
I have added a "readme.txt" with a content of"Hello World!".
Now type <code>git status</code> and you will see that git has found the unversioned (untracked) file.
Just add it to the list of files to be committed via a <code>git add "your file"</code>.
After adding the file to the commit list, commit the file either via <code>git commit -m "Added a file"</code> or <code>git commit</code>.
You have to add a commit message.
This is mandatory.

After committing all files you are done, right? No! But why? You have committed the file only to your local repository! In order to push it to the remote repository on the server you have to do one step more: <code>git push</code>.
Whoops! It fails! But again: why? The answer is pretty easy: Remember we have created an <em>emtpy</em> repository on the server? It has no branches or any structure...
This means that you have to tell the remote repository where to put the first commit you are doing...
Let us try again: <code>git push origin master</code>.
"origin" is the (standard) name of our remote repository and "master" the branch we want to commit our changes to.
Everything should run fine now and we have made our first commit to local and remote repository!

## Using an existing project or local repository
What happens if we have already some sources or a local git repository? If we already have some source files which we want to add to a git repository just change into the root folder of the project you want to commit and create a new git repo via <code>git init</code>.
Then commit all the files to your local repository via <code>git add .</code> and <code>git commit -m "Init commit"</code>.

The next step is to connect your local repo to the remote repo.
This is fairly easy:
``git remote add origin ssh://<ssh-user>@stekoe.de/www/htdocs/stekoe/git-repos/myPrivateRepo``.
This command tells your local repository that a remote repository is located at the given URL and that you want to name it "origin" for readability.
Now you can use the <code>git push</code> command to push changes to the repository and <code>git pull</code> to receive changes committed (by another user or whatever) from the repository.
That's it :)