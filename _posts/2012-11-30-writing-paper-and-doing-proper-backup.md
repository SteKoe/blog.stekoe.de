---
layout: post
title: "[Updated] Writing papers and doing proper backup?!"
tags:
 - Latex
 - Drobox
 - Git
---
This article is about how to save papers and how to back it up properly.
I am currently writing some papers on different topics and as I am studying Information Systems – Wirtschaftsinformatik – I am aware of hardware or software crashes.
Therefore I have found a very simple but effective way in how to get rid of the threat of losing my entire work.

<!--more-->

Since I am writing my papers using LaTeX it is very easy to set up a proper backup method.
LaTeX documents are written in plaintext format.
Nevertheless you can perform my backup method if you are using Microsoft Word or similar software, too.
Anyways: This article is about how to set up Eclipse using EGit for versioning my work and Dropbox as my backup drive.

Let's assume you want to write a paper using LaTeX then you should use Eclipse as LaTeX-Editor since you can integrate Git, SVN or CVS into Eclipse flawlessly.
When you start to work using Eclipse, set your workspace location to a subfolder of your local Dropbox (or whatever you are using) folder.
Everything you are doing in Eclipse now is synchronised into the Cloud.
You will never loose you work anymore – unless Dropbox will collapse ;).
If you are a very paranoid user you might also consider to encrypt your workspace folder, because then even the Dropbox staff won't be able to read your papers and steal them.

This was the first thing to set up.
Second is to use Git! – Oh god, why git, it is just another hyped versioning System!!! Yes, it is, but: You do not have to set up a server, keep it running and so on.
You just have to create a local repository (in a folder synced to Dropbox) and everything is fine.
And since Eclipse provides a plugin for Git, its incredibly easy! Just open "Help" -&gt; "Install new Software..." in Eclipse, select the update site of your current Eclipse distribution and look for "EGit" plugin.
Select it and install.

After you have installed EGit and restarted Eclipse, perform a right click on your project in the Package Explorer, choose "Team" and then "Share Project...".
In the opening window choose "Git", hit next.
In the upcoming window check the box "Use or create repository [...]", hit the button "Create Repository" then hit "Finish".
You have set up your own, local Git Repository now!

At this point you can start to work on your paper.
Every time you have finished work or you want to mark a milestone, you can check in your changes to your repository.
You are able to switch between revisions since you are using a versioning system! To commit your changes, right click on your project, choose "Team" and then "Commit".
Check the files at the bottom which you want to commit, enter a message at the top (e.g.: Rewritten first chapter) and hit commit.
You now have committed your changes and you will be able to switch between different versions of your work.
For instance: When you have deleted a paragraph, saved your changes and can't get it back, you can just switch back to the previous revision of the file! And as the repository is saved in the same folder as your project, it is synchronised to the cloud at the same time.

I think, that this method is very simple and very handy since it automagically backs up your data into a cloud and you have the opportunity to save different versions of your work to a local repository which has been set up in seconds without using a server.
It just works :)
<p style="text-align: left;"><strong>Edit: </strong>In addition to saving your work to a Git-Repository and syncing it with Dropbox, you may also add your Dropbox folder to your TimeMachine if you are using Mac OS.
Every time you plug in your TimeMachine Backup Drive, your work will be backed up one more time.
Or you run a "rsync -acv –delete" manually.
[Thanks to Florian :)]</p>
<p style="text-align: left;"><strong>Edit 2: </strong>I experienced some major issues while using this concept to backup my thesis.
The issues were caused by Eclipse and Dropbox.
Eclipse seems to constantly write into files in workspace while it is running and Dropbox tries to upload the file every time.
As Eclipse writes nearly every second something to the workspace, Drobox.app uses 100% of CPU.
Just disable Dropbox while writing your thesis/paper/text and restart it after you have finished your work.</p>