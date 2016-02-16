---
title: "Setting up a VirtualHost"
date: 2012-05-08
tags:
- apache
- vhost
- hosts
- localhost
---
It's been a while since I started to learn website engineering on my own, but when I tried to set up multiple hosts on my local computer I ran into problems. Although it is very easy to set up as many hosts on your local computer as you want by using virtual hosts, I have never found a good tutorial describing it in a short but clear way. Therefore I've written this one.

<!--more-->

## Introduction
Whenever you start to develop multiple websites at once it might be usefull to have own, custom local domains to access the websites directly. Let's assume you have three websites: You own personal homepage, one for a friend and the third one for a customer. Then you should have three folders in your document root of your apache server containing the different sources of the websites:

* DOCUMENT ROOT
	* my-tiny-little-website
	* my-friends-website
	* customers-website

In order to access your websites you typed an URL like this into your browser bar: "*http://localhost/my-tiny-little-website*" But woultn't it be much nicer to have an URL like this: "*http://my-tiny-little-wegsite.dev*"? Well, of course, since you do not have to navigate all the way long to your subfolders! And if you are using some difficult techniques like *mod_rewrite* you may run into problems when you do not use an own URL for developing your website.

I hope that you are spoiling to set up an own URL on you local machine at this point. So let's start with the procedure now.

## Editing your hosts file
![Fig. 1: hosts file in terminal](sudonano.png)

Every computer regardless of wether it runs Mac OS, Windows or Linux, you will have to find the ``hosts`` file. On Mac OS and other unix based OS it is located at ``/etc/hosts``. When you are using Windows you have to look for it in this folder: 
``%SystemRoot%\system32\drivers\etc\hosts.``

In order to change the file you have to gain admin previliges. Using unix bases systems,  just to open up the terminal and type in ``sudo nano /etc/hosts`` and you should see an output in your terminal as shown on Figure 1. Or try to use the command ``sudo vi /etc/hosts``.

At this point you have to decide how you want to name your local website. I found out, that it is not good to call them "*whatever.local*", since the domain ending *.local leads to a multicast DNS lookup in your network and may result in longer loading times than expected. Therefore I call my local development pages "*whatever.dev*".

Now, when you have decided how to call your local page, you have to add the URL to your previously opened hosts file. Type in

``127.0.0.1     my-tiny-little-website.dev``

and you are done here. But wait! What did you just do?! Well, simple question, simple answer: Whenever your PC tries to access a network address – no matter if its an IP or URL – it will (depending on your local settings) read in the hosts file and if there is an entry for the IP/URL your PC tries to connect to, it will replace (or route) the connection attempt to the IP address which has been entered in front of the entry.  For instance: When you save your hosts file now and open your browser and you type in "*my-tiny-little-website.dev*" and you hit enter, your PC knows, that it has to route the request to your localhost located at 127.0.0.1. And since your browser does an HTTP-request using port 80, your apache will answer the request.

## Adding virtual host to httpd.conf
![Fig. 2: Apache&#39;s httpd.conf](httpd.conf_.png)

Adding the development URL to your hosts file was the first task to perform. But you have to tell your apache server how to deal with the new URL. Otherwise apache will just deliver you the default webpage in your document root directory. To change this, find the file ``httpd.conf`` on your hard disc. When you are using XAMPP it should be at ``c:/xampp/apache/conf/httpd.conf``. MAMP holds this file at the following location: ``/Applications/MAMP/conf/apache/httpd.conf``. If you are running another unix based system try out this location ``/etc/httpd/conf/httpd.conf`` or search your PC.

Once you have found the file, open it up and add these lines at the end of it:

~~~text
NameVirtualHost *:80

<VirtualHost *:80>
    ServerAlias my-tiny-little-website.dev
    DocumentRoot "/PATH/TO/YOUR/HTDOCS/FOLDER/my-tiny-little-website"
</VirtualHost>
~~~

Add a new file into the folder "my-tiny-little-website" called index.php and add some PHP code. Then restart apache as it needs to re-read the httpd.conf file. When you open a browser now and type in the url "my-tiny-little-website.dev" your PHP file should be parsed and the result should be send to the browser. You have successfully set up a virtual host!