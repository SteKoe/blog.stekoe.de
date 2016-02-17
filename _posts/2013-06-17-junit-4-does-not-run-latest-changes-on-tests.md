---
layout: post
title: "JUnit 4 does not run latest changes on tests"
tags:
 - JUnit4
 - Java
 - Eclipse
---
Today I faced the problem that newly created JUnit test cases were not run by JUnit in Eclipse.

As we are using Eclipse in combination with Maven we figured out that this was caused by a configuration problem of the project settings in Eclipse.

<!--more-->

Maven's default test output directory is <code>target/test-classes</code> whereas Eclipse puts its output to <code>target/classes</code> which is the default configuration.
In order to get my JUnit 4 tests run again, I deleted the <code>target/classes</code> folder and reconfigured my project configuration by selecting the project, hitting <kbd>Alt</kbd> + <kbd>Return</kbd>.
In the properties window navigate to "Java Build Path" and open up the "Source" tab.
Now select the entry for your tests <code>whatever/src/test/java</code> and have a look at the textfield at the bottom.
The content has to look like <code>whatever/target/test-classes</code> and not as in many cases <code>whatever/target/classes</code>.
Change it to <code>whatever/target/test-classes</code>, hit ok and you should be ready to go.