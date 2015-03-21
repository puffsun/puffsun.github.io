---
layout: post
title: "Solutions to Scala for the Impatient - Chapter3"
date: 2014-03-07 23:01:32 +0800
comments: true
categories: Scala
keywords: Scala for the Impatient solutions
description: Scala for the Impatient solutions
---

In this blog post, I will continue to post the solutions to [Scala for the Impatient](http://www.amazon.com/Scala-Impatient-Cay-S-Horstmann/dp/0321774094), Chapter 3 as book notes, hopes it could help if you're reading the book too. Without further ado, let's see the solutions in Scala.

1\. Write a code snippet that sets a to an array of n random integers between 0 (inclusive) and n (exclusive).
```scala

import scala.util.Random

def random_arr(n: Int) = {
	Array.fill(n) {Random.nextInt(n)}
}
```
<!--more-->

2\. Write a loop that swaps adjacent elements of an array of integers. For example, <code>Array(1, 2, 3, 4, 5)</code> becomes <code>Array(2, 1, 4, 3, 5)</code>.
```scala
	val arr = Array(1, 2, 3, 4, 5)
	arr.grouped(2).flatMap {
		case Array(x, y) => Array(y, x)
		case Array(x) => Array(x)
	}.toArray
```

3\. Repeat the preceding assignment, but produce a new array with the swapped values. Use <code>for/yield</code>[^1].
```scala
	val arr = Array(1, 2, 3, 4, 5)
	val new_arr = for (i <- 0 until arr.length) yield {
  	if (i % 2 == 0 && i < arr.length - 1)
  	arr(i + 1)
  	else if (i % 2 != 0 && i > 0)
   		arr(i - 1)
  	else
   		arr(i)
   }
```
	
4\. Given an array of integers, produce a new array that contains all positive values of the original array, in their original order, followed by all values that are zero or negative, in their original order.
	
There's a function called <code>partition</code> built-in into Scala by [ArrayOps](http://www.scala-lang.org/api/2.10.3/index.html#scala.collection.mutable.ArrayOps) class, below is the implementation via <code>partition</code>:
```scala
	val arr = Array(-1, 2, -3, -3.2, -9, 9, 8, 5, -10)
	val p = arr.partition {_ > 0}
	p._1 ++	p._2  // produce Array(2.0, 9.0, 8.0, 5.0, -1.0, -3.0, -3.2, -9.0, -10.0)
```
Of course, we can also achieve without using the partition function, let's do it:

```scala
	val ta = for (e <- a if e > 0) yield e
	val tb = for (e <- a if e <= 0) yield e
	ta ++ tb // produce: Array(2.0, 9.0, 8.0, 5.0, -1.0, -3.0, -3.2, -9.0, -10.0)
```
	
5\. How do you compute the average of an <code>Array[Double]</code>?
```scala
	val arr = Array(-1, 2, -3, -3.2, -9, 9, 8, 5, -10)
	arr.sum / arr.size
```
6\. How do you rearrange the elements of an <code>Array[Int]</code> so that they appear in reverse sorted order? How do you do the same with an <code>ArrayBuffer[Int]</code>?

With array, we can sort it in reverse order by below code:
```scala
	import scala.util.Sorting
	val arr = Array(-1, 2, -3, -3.2, -9, 9, 8, 5, -10)
	Sorting.quickSort(are)	// Sort the array in place
	arr.reverse
```
	
With ArrayBuffer:
```scala
	import collection.mutable.ArrayBuffer
	val a = ArrayBuffer(3, 2, 5, 8, 9, -1, -7)
	a.sortWith(_>_)
```
	
7\. Write a code snippet that produces all values from an array with duplicates removed. (Hint: Look at Scaladoc.)
```scala
	val arr = Array(1, 2, -1, 1, 2, 1)
	arr.distinct // produce Array(1, 2, -1)
```
8\. Rewrite the example at the end of Section 3.4, “Transforming Arrays,” on page 32. Collect indexes of the negative elements, reverse the sequence, drop the last index, and call <code>a.remove(i)</code> for each index. Compare the efficiency of this approach with the two approaches in Section 3.4.

We can get the same result with below code:
```scala
	import collection.mutable.ArrayBuffer
	
	val a = ArrayBuffer(1, 3, 9, -1, -6, 4, -2)
	var indexes = for (i <- 0 until a.length if a(i) < 0) yield i
	indexes = indexes.reverse
	indexes.dropRight(1)
	for (i <- 0 until indexes.length) {
		a.remove(indexes(i))
	}
```
Now let see the time/space efficiency of above code. First, we use indexes to store the indexes of negative numbers, so it will take O(n) extra space. Then we traverse the original array for two times, reverse the indexes of negative numbers once, so the total time complexity will be 3 * O(n) ~ O(n)
	
9\. Make a collection of all time zones returned by <code>java.util.TimeZone.getAvailableIDs</code> that are in America. Strip off the "America/" prefix and sort the result.
```scala
	val zones = java.util.TimeZone.getAvailableIDs()
	val zones_no_prefix = for (zone <- zones if zone.startsWith("America")) yield zone.stripPrefix("America/")
	util.Sorting.quickSort(zones_no_prefix)
```
Another solution more in Scala way:
```scala
val zones = java.util.TimeZone.getAvailableIDs
val zones_no_prefix = zones.filter(_.startsWith("America/")).map(_.drop("America/".size))
util.Sorting.quickSort(zones_no_prefix)
```

10\. Import <code>java.awt.datatransfer._</code> and make an object of type <code>SystemFlavorMap</code> with the call <code>val flavors = SystemFlavorMap.getDefaultFlavorMap().asInstanceOf[SystemFlavorMap]</code>, Then call the <code>getNativesForFlavor</code> method with parameter <code>DataFlavor.imageFlavor</code> and get the return value as a Scala buffer. (Why this obscure class? It’s hard to find uses of <code>java.util.List</code> in the standard Java library.)
```scala
	val flavMap = SystemFlavorMap.getDefaultFlavorMap().asInstanceOf[SystemFlavorMap]
	val natives = flavMap.getNativesForFlavor(DataFlavor.imageFlavor)
```

[^1]: [Scala for the Impatient solutions, by McDamon](https://bitbucket.org/McDamon/scalaimpatient/src/4a11167459b2/ch03/answers.txt)
