---
layout: post
title: "Solutions to Scala for the Impatient - chapter1 and chapter2"
date: 2014-02-24 20:09:58 +0800
comments: true
categories: Scala
keywords: Scala impatient solutions
description: Scala impatient solutions
---

Recently I'm reading the book [Scala for the Impatient](http://www.amazon.com/Scala-Impatient-Cay-S-Horstmann/dp/0321774094), the book is pretty brilliant, you can see that from the comments. Since I'm a newbie to Scala, I'm try to complete the exercises after every chapter, I post the solutions in a serious of blog posts as book notes.

Since both chapter 1 and chapter 2 are very simple, I'll post the solutions together here with this blog post.

<!--more-->
###Solutions to exercises of Chapter 1
8\. One way to create random file or directory names is to produce a random BigInt and convert it to base 36, yielding a string such as "qsnvbevtomcj38o06kul". Poke around Scaladoc to find a way of doing this in Scala.

```scala
	val tmp = BigInt(100, util.Random).toString(36)
```
According to [BigInt's document](http://www.scala-lang.org/api/2.10.2/index.html#scala.math.BigInt$), there's a method called apply, which will construct a randomly generated BigInt, uniformly distributed over the range 0 to (2 ^ numBits - 1), its method signature is <code> def apply(numbits: Int, rnd: Random): BigInt </code>.

9\. How do you get the first character of a string in Scala? The last character?

```scala
	val str = "abcd"
	str.head // get the first
	str.dropRight(1)
```
There're several way to achieve this goal, what I have shown is one of them.

10\. What do the take, drop, takeRight, and dropRight string functions do? What advantage or disadvantage do they have over using substring?
To get an idea what they can do, we need to check Scala document.

The difference between then and java.lang.String is that, before Java 7, String#substring method will operate on the original string, it will generate new String instance at all, so it takes constant time to complete the substring operation. But this behaviour changed after Java 7 due to security reasons, for more details, please go to [Changes to String.substring in Java 7](http://www.javaadvent.com/2012/12/changes-to-stringsubstring-in-java-7.html). After this change, substring will always produce new String instances, which will take linear time to complete the operation.

As far as I know, in Scala, the take\* and drop\* method will always generate new String instances(please correct me if I'm wrong), I think this is the difference.


###Solutions to exercises of Chapter 2
1\. The signum of a number is 1 if the number is positive, –1 if it is negative, and 0 if it is zero. Write a function that computes this value.
```scala
	def signum(num: Int) = { if (num > 0) 1 else if (num < 0) -1 else 0 }
```
Notice that I didn't use return statement in above code, it a best practice to avoid the return statement in Scala.

2\. What is the value of an empty block expression {}? What is its type?
It's of type Unit, you can verify that with <code> {}.getClass </code> inside Scala interactive shell.

3\. Come up with one situation where the assignment x = y = 1 is valid in Scala. (Hint: Pick a suitable type for x.)
```scala
	var x: Unit = Unit
	var y = 0
	x = y = 1
```

4\.Write a Scala equivalent for the Java loop <code>for (int i = 10; i >= 0; i--) System.out.println(i); </code>
```scala
	for (i <- 10 to 0 by -1) println(i)
```

5\. Write a procedure countdown(n: Int) that prints the numbers from n to 0.
```scala
	def countdown(num: Int) {
		var step = if (num >= 0) -1 else 1
		for (i <- num to 0 by step) println(i)
	}
```

6\. Write a for loop for computing the product of the Unicode codes of all letters in a string. For example, the product of the characters in "Hello" is 9415087488L.
```scala
	var prod = 1
	for (i <- "Hello") println(prod * i.toInt)
```

7\. Solve the preceding exercise without writing a loop. (Hint: Look at the StringOps Scaladoc.)
```scala
	"Hello".foldLeft(1L) (_ * _.toInt)
```
Note that here I use 1L instead of 1, because java.lang.Integer.MAX_VALUE is 2147483647, which is too small to hold the produce of products of "Hello", so I need to specify Long as the products type.

8\. Write a function product(s : String) that computes the product, as described in the preceding exercises.
```scala
	def product(str: String) = {
		str.foldLeft(1L) (_ * _.toInt)
	}
```
9\. Make the function of the preceding exercise a recursive function.
```scala
	def product_r(str: String): Long = {
		str match {
			case "" => 1; 
			case _ => str.head.toInt * product_r(str.tail)
		}
	}
```
Note that we still specify the return value as Long for the same reason above.



