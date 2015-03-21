---
layout: post
title: "solutions to scala for the impatient - chapter4"
date: 2014-03-08 20:49:44 +0800
comments: true
categories: Scala
keywords: Scala for the impatient solution
description: Scala for the impatient solutions of chapter 4
---

1\. Set up a map of prices for a number of gizmos that you covet. Then produce a second map with the same keys and the prices at a 10 percent discount.

```scala
val books = Map("Scala for the Impatient" -> 30, "Ruby under a Microscope" -> 40, "Ruby Cookbook" -> 35)
val discounted = for((b, p) <- books) yield (b, p * 0.9)
```
<!--more-->

2\. Write a program that reads words from a file. Use a mutable map to count how often each word appears. To read the words, simply use a <code>java.util.Scanner</code>:
```scala
val in = new java.util.Scanner(new java.io.File("myfile.txt"))
while (in.hasNext()) 
	process in.next()
```
Or look at Chapter 9 for a Scalaesque way.
At the end, print out all words and their counts.

```scala
def count_words(file: String): Unit = {
	val container = collection.mutable.Map[String, Int]()
	val in = new java.util.Scanner(new java.io.File(file))
	while (in.hasNext()) {
		val word =  in.next()
		if (container.contains(word))
			container(word) += 1
		else
			container(word) = 1
	}
	for ((k, v) <- container)
		println(k + ": " + v)
}
```

3\. Repeat the preceding exercise with an immutable map.
```scala
def count_words(file: String): Unit = {
	var container = Map[String, Int]()
	val in = new java.util.Scanner(new java.io.File(file))
	while (in.hasNext()) {
		val word = in.next()
		container = container + (word -> (container.getOrElse(word, 0) + 1))
	}
	println(container)
}
```
Or we can achieve nearly the same more Scalaesque(the result is little different, but you get the idea)[^1]:
```scala
import scala.io.Source

def count_words(file: String): Unit = {
	val source = Source.fromFile(file)
	val words = for (word <- source.getLines.toArray) yield word
	val wordCounts = for (word <- words.distinct) yield {
  		(word, words.count(_ == word))
	}
	val wordCountsMap = wordCounts.toMap
	println(wordCountsMap)
}
```

4\. Repeat the preceding exercise with a sorted map, so that the words are printed in sorted order.
```scala
def count_words(file: String): Unit = {
	var container = collection.immutable.SortedMap[String, Int]()
	val in = new java.util.Scanner(new java.io.File(file))
	while (in.hasNext()) {
		val word = in.next()
		container = container + (word -> (container.getOrElse(word, 0) + 1))
	}
	println(container)
}
```

5\. Repeat the preceding exercise with a <code>java.util.TreeMap</code> that you adapt to the Scala API.
```scala
def count_words(file: String): Unit = {
	val container = new java.util.TreeMap[String, Int]
	val in = new java.util.Scanner(new java.io.File(file))
	while (in.hasNext()) {
		val word =  in.next()
		if (container.containsKey(word))
			container.put(word, container.get(word) + 1)
		else
			container.put(word, 1)
	}
	println(container)
}
```

6\. Define a linked hash map that maps "Monday" to <code>java.util.Calendar.MONDAY</code>, and similarly for the other weekdays. Demonstrate that the elements are visited in insertion order.
```scala
val daysOfTheWeek = LinkedHashMap("Monday" -> java.util.Calendar.MONDAY,
    "Tuesday" -> java.util.Calendar.TUESDAY,
    "Wednesday" -> java.util.Calendar.WEDNESDAY,
    "Thursday" -> java.util.Calendar.THURSDAY,
    "Friday" -> java.util.Calendar.FRIDAY,
    "Saturday" -> java.util.Calendar.SATURDAY,
    "Sunday" -> java.util.Calendar.SUNDAY)
    
println(daysOfTheWeek)
```

7\. Print a table of all Java properties, like this:
```
java.runtime.name             | Java(TM) SE Runtime Environment
sun.boot.library.path         | /home/apps/jdk1.6.0_21/jre/lib/i386
java.vm.version               | 17.0-b16
java.vm.vendor                | Sun Microsystems Inc.
java.vendor.url               | http://java.sun.com/
path.separator                | :
java.vm.name                  | Java HotSpot(TM) Server VM
```
You need to find the length of the longest key before you can print the table.
```scala
import scala.collection.JavaConversions.propertiesAsScalaMap
val props: scala.collection.Map[String, String] = System.getProperties
val len = props.keys.maxBy(_.length).length // java.vm.specification.version
for ((k, v) <- props) printf("%-"+ len + "s|%s\n", k, v)
```

8\. Write a function <code>min_max(values: Array[Int])</code> that returns a pair containing the smallest and largest values in the array.
```scala
def min_max(values: Array[Int]): Tuple2[Int, Int] = {
	(values.min, values.max)
}
```

9\. Write a function lt_eq_gt(values: Array[Int], v: Int) that returns a triple containing the counts of values less than v, equal to v, and greater than v.
```scala
def lt_eg_gt(values: Array[Int], v: Int): Tuple3[Int, Int, Int] = {
	(values.count(_ < v), values.count(_ == v), values.count(_> v))
}
```

10\. What happens when you zip together two strings, such as <code>"Hello".zip("World")</code>? Come up with a plausible use case.
```scala
// produce: scala.collection.immutable.IndexedSeq[(Char, Char)] = Vector((H,W), (e,o), (l,r), (l,l), (o,d))
"Hello".zip("World") 
```
From above code we can see that the zip of two String will produce a immutable Vector, which contains Tuples. The Tuples contains a pair of characters in the same position. A possible use case is to produce a immutable map with left characters as keys and right side characters as values, you can transfer the zipped characters to a immutable map via:
```scala
val map = "Hello".zip("World").toMap
```

[^1]: [Scala for the Impatient solutions, by McDamon](https://bitbucket.org/McDamon/scalaimpatient/src/4a11167459b2/ch04/answers.txt)
