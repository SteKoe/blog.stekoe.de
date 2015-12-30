---
title: "Status 400 when using AJAX, JSON and Spring MVC"
date: 2014-12-01
tags: 
- java
- spring
- json
- ajax
- JavaScript
---
I recently faced a problem using Spring MVC and AJAX calls.
Whenever I tried to send JSON data via jQuery's `$.ajax()` function, my Spring MVC application returned HTTP status 400 "The request sent by the client was syntactically incorrect".
My first attempt to solve this problem was – of course – browsing the web, especially http://stackoverflow.com/ (SO).
But since finding a proper solution was not that easy, I am going to write down the problems I faced and finally the solution.

<!--more-->

First of all I have implemented a controller like this:

```java
public class MyController {
    @RequestMapping(value = "/member", method = )
    public @ResponseBody String post(@Valid @RequstBody Member member) {
        // ...
        Do whatever you want
    }
}
```

The class Member is defined as follows:

```java
public class Member {
    private String id;
    private String name;

    public Member(String id, String name) {
        this.id = id;
        this.name = name;
    }
}
```

Whenever one executes an AJAX call with jQuery in order to create a new member, the application returns HTTP error 400.
But why does this happen? First let's have a look at the jQuery code:

```javascript
var data = {
    id: "",
    name: "Hans Sarpei"
};

$.ajax({
    url: form.attr('action'),
    data : JSON.stringify(data),
    contentType:"application/json; charset=utf-8",
    dataType:"json",
    type: 'POST'
});
```

When the AJAX call is executed, a POST request with JSON data is sent to the application.
Spring MVC decodes the JSON data and tries to map it to the desired Object–in this case "Member".
But it seems that the framework fails at this point.
So what is the problem?

Most possible solutions found on SO stated that the JSON object definition is malformed and therefore not valid JSON.
Hence Spring MVC won't be able to decode it: <code>{name: "Hans Sarpei"}</code> is not a valid JSON object, but <code>{"name":"Hans Sarpei"}</code> is (mind the double-quoted field definition).

Since no solutions mentioned above solved my problem, I started to investigate my code and found the problem.
In the class Member I have defined a non-standard constructor and a few getters and setters.
Since the class does not have a standard constructor, Spring MVC seems to not be able to decode the JSON @RequestBody.
Hence, I implemented a standard constructor and everything went smooth.
