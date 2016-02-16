---
layout: post
title: "FakeSMTP: A local mail server for testing purposes"
date: 2014-06-08
tags:
- SMTP
- local
- unit
- testing
- hamster
---
I recently found a very handy tool which allows one to set up a simple mail server which runs on local host for testing purposes. Since I currently develop a web application which has to send emails on registration or other notification events, I was looking for a simple tool which allows me to simulate a simple mail server running on my local machine. While I was browsing the internet I found different solutions. One was to use the mail server which is shipped with every Mac OS copy: Postfix. But I just wanted to have a nice UI like the Fake Mail App "[Hamster](http://hamster.volker-gringmuth.de)" on Windows. Then I stumbled apon "[FakeSMTP](http://nilhcem.github.io/FakeSMTP/)". This tool does exactly what I was looking for!

<!-- more -->

## Short overview
FakeSMS is written in Java and is shipped free in a single Jar-Archive under the BSD license. In order to start the SMTP server one has different options: either one uses the implemented UI by starting the Jar via a double click or via `java -jar smartSMTP.jar.` Or one uses the command line tool which is implemented as well. The available options are as follows (Version 1.10):

<pre>usage: java -jar fakeSMTP-1.10.jar [OPTION]...
 -a,--bind-address     IP address or hostname to bind to. Binds to
                       all local IP addresses if not specified. Only
                       works together with the -b (--background)
                       argument.
 -b,--background       If specified, does not start the GUI. Must be
                       used with the -s (--start-server) argument
 -m,--memory-mode      Disable the persistence in order to avoid the
                       overhead that it adds
 -o,--output-dir       Emails output directory
 -p,--port             SMTP port number
 -r,--relay-domains    Comma separated email domain(s) for which
                       relay is accepted. If not specified, relays to
                       any domain. If specified, relays only emails
                       matching these domain(s), dropping (not
                       saving) others
 -s,--start-server     Automatically starts the SMTP server at launch
</pre>

The cli offers one a bit more option as the UI but nevertheless this article concentrates on the use of the UI since one can see all important aspects there as well: Received mails, an SMTP log and the last received message including the full email header.

## The UI
In the following paragraphs I will shortly describe the contents of the UI and its functions.

### Mails list
{% include image.html file="2014-06-08-fakesmtp-a-local-mail-server-for-testing-purposes/fakesmtp-mails.png" %}
When one starts the UI one is able to set the listening port of the server. In my case I have set it to port 1234 as the default SMTP port is already used on my Mac. Then I selected a folder where SmartSMTP will save all the received emails to. The emails will be saved in .eml format which is readable by all common mail applications like Apple Mail, Outlook, Thunderbird,... In addition to saving the emails to disc, a list of received emails is shown in the lower half of the UI. The list updates immediately when the SmartSMTP server has received an email (see picture 1). One useful function is to open the received emails by just double clicking on them. In this case the standard mail application will open and show the received message.

### SMTP log
{% include image.html file="2014-06-08-fakesmtp-a-local-mail-server-for-testing-purposes/fakesmtp-log.png" %}

The SMTP log section is very useful if you face problems connecting to FakeSMTP server. I faced the problem, that the mail client I use in my web application did not send a correct hostname to the mail server which ended up in an exception since the SMTP command `EHLO hostname` is invalid and must contain a valid hostname instead of a placeholder like "hostname". I solved this issue by adding the property `mail.smtp.localhost = localhost` to the list of properties in my Bean configuration.

### Last message
This section is useful when one just wants to take a detailed look on the last received email. When one opens up the section "last message", one will see a big textfield containing the full email which the server received last including the full header of the mail.

## Conclusion
FakeSMTP is a very cool tool if one wants to simulate a very simple and rudimentary email server which accepts mails. The application shows the received emails in a very comfortable way and also allows to read the received mails very easy. If one wants to take a detailed look on the SMTP commands which are run this is also possible by using the SMTP log section. FakeSMTP is [hosted at Github](https://github.com/Nilhcem/FakeSMTP) and can be downloaded there as well.
