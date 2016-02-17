---
layout: post
title: "Enable User Management in Jenkins"
date: 2013-07-08
tags:
 - Jenkins
 - CI 
---
Just a short memo to all users who are looking for the "Manage Users" section in Jenkins... Here is how to activate it in ten easy steps:

1. Klick on "Manage Jenkins"
1. Then on "Configure Global Security".
1. Then activate "Jenkin's own user database" beyond "Security Realm"
1. Now scroll down to "Authorization" and activate "Matrix-based security". You will find one entry for the user "Anonymous".
1. Activate all checkboxes for this user (hint: use the tiny litte icon at the far right of the table row).
1. Hit "Save".
1. Got back to "Manage Jenkins" and you will find a new entry named "Manage Users".
1. Create a new user and copy it's username
1. Add the user (via it's username) to the list mentioned in step 4.
1. Turn off all checkboxes for the new user and remove all checkboxes for the user "Anonymous". Done.