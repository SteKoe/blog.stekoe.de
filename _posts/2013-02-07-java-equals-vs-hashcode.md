---
layout: post
title: "Java equals() vs. hashCode() methods"
tags:
 - Java
 - equals()
 - hashCode()
---
I recently stumbled upon the two methods `equals()` and `hashCode()` in Java while I was implementing a class which should be able to be comparable.
My first try was just to use the common `equals()` method in `java.lang.Object` which lead to the problem that the method always returned false–but why? 
Lets have a look at the implementation of the `equals()` method in `java.lang.Object`:

{% gist d9d6c83f421a1c6ea41d %}

What do these three (or one in particular) lines do? Well, each Object is located in main memory as long as any part of the software has an active reference to the object, otherwise it will be removed by the garbage collector.
So each variable points to the location of the object in memory.
Hence, the line above compares wether the location of the compared objects are equal or not and if they are equal the compared objects are equal.
You may think that this is fine, but we will see: It's not!

<!--more-->

## equals()
In order to explain the behaviour of `equals() and `hashCode()` we need to introduce a example class as follows:

{% gist 5127c56ef0df9e8890dd %}

The class represents a simple type of a Employee who has a number, name and a birthday.–At this point we ignore the fact that type String is not the best choice to be used to store a date (instead of using Calendar), but thats another story.

What we want to do now is to run some tests using the class above and the methods `equals()` and hashCode().
Therefore we create a new class which will implement some JUnit tests like this one:

{% gist 60842a79057ad60edfdb %}

What is done in the code above? We create a new instance of type Employee and fill it with some example data.
Then we compare the newly generated Object named "burns" with itself.
Referring to the description in the Java API how `equals()` works, the result of the comparison will be true.
As we use JUnit, we run an asserTrue() in order to get notified if the expression which is passed to the asserTrue() method returns false.
As we compare the very same object here which is located at the same point in main memory, `equals()` will return true.
But lets go on and have a look at the following situation:

{% gist 3c37107bc114df8483b1 %}

We have two objects, burns and burns_double which do have exactly the same "content"–in case of objects we speak of "states".
BUT: if we now compare these two objects by using the `equals()` method, the JUnit test will fail, why? Again, let's go one step back and think again of how `equals()` works: It just compares wether the passed in references are pointing to the same location in main memory.
As we do have two different objects which have their own separated locations in memory, the common implementation of `equals()` will return false, although the object states are equal.
The current implementation of Employee needs to be extended in order to achieve the desired result.
We have to override the `equals()` method on our own, so that it checks wether we are dealing with the very same object or wether the objects have equal states.
In order to start to implement our own `equals()` method, we have to have a look on the source code of java.lang.Object as you have to fulfill some requirements when overriding the `equals()` method:


> For any non-null reference value
> x, x.equals(x) should return true.
> 
> For any non-null reference value x,
> x.equals(null) should return false.

Lets start to implement the basic requirements mentioned above:

{% gist f49d3a31fec2c8301b68 %}

What have we done here? Lines three and four implement the same operation as the java.lang.Object#equals() method.
We have discussed them above.
Lines five and six fulfills the requirement that "x.equals(null) should return false".
Lines seven and eight compare wether the current object has the same type as the object which has been passed into the `equals()` method.–Wait! Why don't we use "instanceof" here?

Let's assume that we have a specialised version of class Employee called Manager and we create an object of each of the classes with the same content.
If we now perform a "instanceof" call on the Manager-Object like obj instanceof Employee[/cci] it will return true.
But in our case we do not want to check wether the object is an instance of a type but if its the same type (no specialised one) and state.
Hence, "instanceof" would not fulfill this and is not sufficient.

By now we have implemented some generic comparison in the `equals()` method.
But we still do not check for the state of the object.
Therefore we start to implement this now:

{% gist aecbedc3beb9f8eefa7a %}

Now we have implemented state checking as well.
For each field in class Employee that we want to be part of the comparison, we have written an if block in our `equals()` method.
As the code contains just some basic logic, just read and try to understand it–it is really straight forward.

Now, if we rerun our JUnit test "iShouldBeEqual()", the JUnit test will pass which means that we are done, or? Sorry for asking a leading question as the answer is obvious: No, we are not done yet! But why?! Well, we have to refer to the documentation of java.lang.Object again.
At the end of the documentation block of the `equals()` method you will find the following sentence:

> Note that it is generally necessary to override the {@code hashCode} method whenever this method is overridden, so as to maintain the general contract for the {@code hashCode} method, which states that equal objects must have equal hash codes.

Hence, I will introduce the `hashCode()` method in short in the following chapter and describe where it is used and why it is important to implement the `hashCode()` method as well.

## hashCode()

When you think of hash codes some other terms might get into your mind like "hash map", "hash set" or "hash table".
This are basically the fields where `hashCode()` method is used and why it is necessary to override the implementation of it.
The current implementation of our example just contains the `equals()` method which is currently comparing objects and their states.
As soon as we want to use the type Employee in combination with a HashMap or HashSet, we run into problems which are not obvious at first sight.

Although I will not introduce HashMap and HashSet in detail, I will give you the most important information about these two Collections in Java and in general.
HashMaps and HashSets are special types of Collections which are using hash codes in order to address contained elements in the collection in a fast and efficient way.
HashMaps can contain any amount of objects, even equal objects more than one time.
HashSets may only contain objects once(!).
If you pass an object to a HashSet which is already part of the HashSet, the old value will be overwritten with the new one (cf.
java.util.HashMap(sic!) ll.
468).
That's the major difference between HashMaps and HashSets, but what about the term "Hash"?

If you take a deeper view into the source code of HashMaps and HashSets you will see that both use some kind of table which contains a key and a value.
The key corresponds to the hash of an stored object and the value is the stored object itself.
But why do we use hashes instead of just looking up the objects directly? To keep the answer as short as possible, ask yourself: What is faster, comparing computers by their part numbers (model identifiers or whatever) or by comparing each part inside the computer? Of course it is much faster to compare the part number of the whole PC instead of each part itself.
And thats why HashTables have been invented.
In case of cost reduction (in this context costs do not have any economical aspect but express the time an algorithm lasts), hashes have evolved and are much easier and faster to use than anything else.
In order to keep the advantages of HashTables up, it is necessary that hashes for objects are as unique as possible.
If we have more than one object with the same hash we have to deal with collisions of hash codes.
If multiple objects which are passed into a HashCollection have the same hash code in Java, they will be added to a single chained list of table entries:

~~~
Key      Value
====     =======================================================
0001     "object 1" --> "object 2" --> "object 3" --> null
0002     null
0003     "object 4" --> null
....
~~~

If each object returns the same hash code, all objects would be added to one key containing all objects.
This would slow down the hash table dramatically.

One other thing you may have noticed is that you can check wether objects are not equal by comparing their hash codes but you can not be sure wether objects are equal if you just compare their hash code, since two totally different objects may return the very same hash code.

Let's implement the `hashCode()` method now.
One good rule for implementing the `hashCode()` method is to use any field you have used in the `equals()` method in `hashCode()` again.
In case of our example this would be the fields "itsName", "itsBirthday" and "itsPersonnalNumber".
Our `hashCode()` method will look as follows:

{% gist d77636b4cb6167fab4bc %}

What do we do here? We are calculating a hash code as integer value by adding and multiplying different values.
Line four initialises the result by setting the variable result to 1 but this value is arbitrary.
The next lines (five and six) calculate the hash codes of the fields which are of type String by using the `hashCode()` method implemented in java.lang.String or zero if the field values are null.
Line seven just adds the value of the integer of field "itsPersonnalNumber" to the result variable.
Finally in line eight the computed result is returned as the hash code of the current object.

What about line three? Why do we use the prime 31 here? As Josuha Bloch (p.48) says, 31 is used because it is an odd number and in case of overflowing multiplication, "information would be lost, as multiplication by 2 is equivalent to shifting".
Though, the usage of a prime is not clear, but traditional.
This means, we are allowed to use any number we want to use if it is not even.

## Conclusion

In this blog post you have learned why it might be necessary to override the `equals()` method in your implementation and why it is necessary to override `hashCode()` method as well.
I have tried to explain the behaviour of both methods as simple as possible and in a way I got used to the meanings of both methods myself.
If you want to get more into detail, I recommend you to read the source code and documentation of the classes touched in this blog or to get your hands on a book which is focused on this topic (i.e.
Effective Java, Second Edition by Joshua Bloch.
Addison-Wesly Pearson Education.
ISBN-13: 978-0-321-35668-0).
