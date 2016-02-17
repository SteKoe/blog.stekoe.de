---
layout: post
title: "Nested category list in WordPress"
tags:
 - WordPress
 - PHP
---
I have just added a nested category item to our category list (on the right side of our page) and wondered why the nested item was not shown as child entry in the sidebar.
After hours of looking around in the WordPress documentation, I figured out how to get a hierarchical (nested) category list.

<!--more-->

The method ``wp_list_categories()`` can receive various arguments to control the output.
When you are using widgets in your templates and especially the category list widget, you can not easily open a PHP file, change the arguments as desired and upload the file again.
In order to manipulate/change the arguments you have to add filters in your function.php file of the current template.

As you can see in the code example at the end of this post, you may change ANY of the arguments passed to the function ``wp_list_categories()`` by just merging them to the ``$args`` array inside a special filter.

In the case of SystemOutPrintln.de we wanted to have a hierarchical (nested) list, instead of a plain one (shortened) as show below:
	
* Software development
* Miscellaneous
* OS X
* Common
	
We wanted the result to look like this (shortened):

* Software development
* Miscellaneous
	* OS X
* Common
	
You see: OS X is now displayed as a child entry of the category "Miscellaneous".
This makes the list much clearer and readable to you, the user of our website.
When you set the argument ``hierarchical`` to ``true``, the list will look like this (shortened):
	
* Categories
	* Software development
	* Miscellaneous
		* OS X
	* Common
		
A new level holding just one entry (Categories) appears.
But as we do not want this, you can just unset the title by setting ``title_li`` to ``''`` (an empty string).
The result looks like this:
	
* (the empty string)
	* Software development
	* Miscellaneous
		* OS X
	* Common
		
Now  you can simply style the whole list using CSS.
If you want to see a CSS example, look at the code example below.
We provide you with our CSS styles for our nested category list and with an expert of our functions.php file.

As you can see, it is very easy to manipulate the output of the category widget by just adding a filter to the ``widgets_category_args`` (as shown below).

We hope that this article helps you to understand filters in WordPress and how to change the output of the widgets on the front page.

{% gist b31eeca2d672e3947992 %}

{% gist 456c358f40ea3e2fc109 %}

