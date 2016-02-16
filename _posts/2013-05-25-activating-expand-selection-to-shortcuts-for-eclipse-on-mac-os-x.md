---
layout: post
title: "Activating \"Expand selection to\" shortcuts for Eclipse on Mac OS X"
date: 2013-05-25
categories: 
- IDE
tags:
- Eclipse RCP
- Shortcuts
- Mac OS
---
When using Eclipse on Windows the IDE offers you a very useful shortcut to select lines and code blocks at once. This feature is called "Expand selection to..." and you can trigger it by using the keys <kbd>Alt</kbd> <kbd>Shift</kbd> <kbd>Arrow Keys</kbd>.

{% raw %}
<table>
    <tr>
        <td rowspan="4">Expand Selection to</td>
        <td>Enclosing Element: Selects the enclosing expression, block, method in the code. This action is aware of the Java syntax. It may not function properly when the code has syntax errors. (<kbd>Arrow Up</kbd>).</td>
        <td rowspan="4"><kbd>Alt</kbd> <kbd>Shift</kbd> <kbd>Arrow Keys</kbd></td>
    </tr>
    <tr>
        <td height="34">Next Element: Selects the current and next element. (<kbd>Arrow Right</kbd>)</td>
    </tr>
    <tr>
        <td height="34">Previous Element: Selects the current and the previous element (<kbd>Arrow Left</kbd>)</td>
    </tr>
    <tr>
        <td height="34">Restore Last Selection: After an invocation of Expand Selection to restore the previous selection. (<kbd>Arrow Down</kbd>)</td>
    </tr>
</table>
{% endraw %}


![Key binding settings](1140_disable_system_shortcuts.png)
The corresponding key bindings on Mac OS are as follows: <kbd>Ctrl</kbd> <kbd>Shift</kbd> <kbd>Arrow Keys</kbd>

But there is one problem with this key binding: It is usually bound to a Mac OS feature to switch Desktops or to open Mission Control (aka Exposé). You have to deactivate this features in the settings of Mac OS as shown in the figure on the right.

After disabling the marked key bindings you will be able to use the "Expand Selection to..." feature in Eclipse on Mac OS. Enjoy!