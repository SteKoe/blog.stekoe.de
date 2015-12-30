---
title: "Keeping phpMyAdmin up-to-date"
date: 2012-12-24
tags:
---
I recently got a message from my local phpMyAdmin installation to update to the most current version. I then headed to the web page of <a href="http://www.phpmyadmin.net" target="_blank">phpMyAdmin</a> and started to download the latest stable Release. But then I recognized, that phpMyAdmin has moved its source to <a href="http://www.github.com" target="_blank">GitHub</a>. So why not checking out the code and perform a <code>git pull</code> in order to update the phpMyAdmin installation next time?

<!--more-->

So what I did was: Heading to the folder which contains the current phpMyAdmin installation and backing up the config.inc.php file. This is necessary as this file contains all your configuration data for phpMyAdmin. After you are sure that you have backed-up the config file properly, clear the entire folder. This is necessary as the <code>git clone</code> command needs an empty folder. I achieved this by running <code>rm -rf ./</code>. After you have cleared the folder, you can run the <code>git clone</code> command: <code>git clone https://github.com/phpmyadmin/phpmyadmin.git ./</code>

The cloning takes up to 10 minutes, depending on your connection speed. After you have cloned the repository, you have checked out the most current development version of phpMyAdmin. When you just want to use stable releases, you have to run the following command as well: <code>git checkout -t origin/STABLE</code>.

Now you have checked out the most recent stable version of phpMyAdmin. The last step is to move back your <code>config.inc.php</code> into the phpMyAdmin folder. Voila: Your phpMyAdmin installation works again. If you want to update your installation now, just run a <code>git pull</code> and the most recent sources will be checked out.

Btw.: Happy holidays!