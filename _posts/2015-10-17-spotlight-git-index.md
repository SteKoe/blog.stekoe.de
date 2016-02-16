---
title: "Spotlight: Git index"
date: 2015-10-17
tags:
- git
- index
---
I cannot remember when I started to use version control systems (VCS), but I know that I started by using Apache's SVN. Nowadays the tool Git has gained more attention and is used in many companies as well as in software engineering in general. But even though I am using Git on a daily basis, I have not understood the main concepts of Git in detail yet. Therefore I will have a closer look on Git's index and its behavior in this blogpost. I also plan to add more blogposts focussing on Git objects and the objects store.

<!-- more -->

Since Git is a very complex tool -- it is a distributed system --, I will just focus on one local Git repository at the moment. Hence, this blogpost will not handle things like bare repositories or remote repositories at all. In order to understand the concept of Git's index I write down all steps I have done to understand the way Git is working. The following list shows each step I have performed to see what kind of information is stored in the index.

## File handling

1. **Create a new empty Git repository**
First of all I need to create an empty Git repository anywhere on my hard disc drive. In order to do so, I use the well known git command as follows:

~~~ bash
$ git init .
Initialized empty Git repository in ~/Development/spotlight-git-index
~~~

1. **Create new file: test.txt**
Then I create a new file with some example content in it:

~~~bash
$ echo "Test" > test.txt
~~~

1. **Check Git's index**
Event though the new file has been created in the workspace, Git does not know the file yet. In order to link the file to Git, we have to add it as described in the following step.

~~~bash
$ git ls-files --stage
~~~

1. **Add file to Git's index**
In order to add the file to the Git index, one has to use the following command. After the command has been executed Git now knows the file and tracks it.

~~~bash
$ git add .
~~~

1. **Check Git's index**
When one checks Git's index once again, one can see that Git knows the file and tracks it now. This is done by calculating an hash value of the file's content which is added to the index and can be seen in the output of the index listing command.
 
~~~bash
$ git ls-files --stage
100644 345e6aef713208c8d50cdea23b85e6ad831f0449 0	test.txt
~~~

1. **Add another line to test.txt**
In this step we are going to update the existing file by adding a new line of text.

~~~bash
$ echo "Test" >> test.txt
~~~

1. **Check Git' index**
When checking the index again, one can see that the index has not recognized the changes yet, since we have to tell Git to do so as described in the following step. Notice that the hash value has not yet changed in the output of the index listing command:

~~~bash
$ git ls-files --stage
100644 345e6aef713208c8d50cdea23b85e6ad831f0449 0	test.txt
~~~

1. **Add modified test.txt file to index again**
We now add the updated test.txt file to Git's index again.

~~~bash
$ git add .
~~~

1. **Check Git's index**
The file test.txt has changed due to content modificiations and hence the hash value has changed as well. You can now see the newly generated hash value based on the new content of the file below.

~~~bash
$ git ls-files --stage  
100644 95306f63471907a92e280462cdc96272bdd7b11b 0	test.txt
~~~

## Folder handling

1. **Add a new folder**
Let's have a look how Git handles folders in the index. So let's create an empty folder and add the changes to the index.

~~~bash
$ mkdir folder
$ git add .
~~~

1. **Check Git's index**
When one now checks the index again, one can see that the empty folder is not recognized at all. This is due to Git's focus on files only. Hence it is not even possible to commit empty folders. (*Note: One workaround to commit empty folders is to add a file named "empty" to it which indicates the users that the folder is somehow required in the project, and that the empty file is necessary to Git for commiting an empty folder*).

~~~bash
$ git ls-files --stage  
100644 95306f63471907a92e280462cdc96272bdd7b11b 0	test.txt
~~~

1. **Create new file in empty folder and add it to index**
Let's create a new empty file in the empty folder and add it to the index.

~~~
$ cd folder
$ echo "test" >> 2.txt 
$ cd .. 
$ git add .
~~~

1. **Check Git's index**
The newly created file 2.txt is tracked by Git now by using the relative path to the file.
The hash of test.txt has not changed since the content hasn't changed at all.

~~~bash
$ git ls-files --stage  
100644 9daeafb9864cf43055ae93beb0afd6c7d144bfa4 0	folder/2.txt
100644 95306f63471907a92e280462cdc96272bdd7b11b 0	test.txt
~~~

1. **Create another new file**
Let's create another new file in our repository. For simplicity reasons we will just copy an existing file and add it to the index since we already know that the file will not be tracked otherwise.

~~~bash
$ cp test.txt newText.txt
$ git add .
~~~

1. **Check Git's index**
As we can see know the file has been added to the index. But have a closer look at the hash values of the files. They are equal! But why is that? You might have wondered why Git creates hashes of the files and how Git tracks changes of the files. This question will be answered in another blogpost later and is a very important concept. But for short: if two tracked files contain the very same content, the hash values will always be equal. 

~~~bash
$ git ls-files --stage  
100644 9daeafb9864cf43055ae93beb0afd6c7d144bfa4 0	folder/2.txt
100644 95306f63471907a92e280462cdc96272bdd7b11b 0	newText.txt
100644 95306f63471907a92e280462cdc96272bdd7b11b 0	test.txt
~~~


