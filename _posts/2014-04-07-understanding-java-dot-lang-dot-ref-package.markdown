---
layout: post
title: "understanding java.lang.ref package"
date: 2014-04-07 15:03:55 +0800
comments: true
categories: Java
keywords: WeakHashMap WeakReference SoftReference PhantomReference HashMap Cache Java Reference Collection
description: WeakHashMap WeakReference SoftReference PhantomReference HashMap Cache Java Reference Collection
---

There're several useful classes under package <code>java.lang.ref.*</code>, which seems to be a little-known feature. That's what I'm going to talk about in this blog post.

###A Program with Memory Leak
As Java developer, most of time, you should not pay attention to memory management, JVM GC do that job for you automatically. It can easily lead to the impression that you don’t have to think about memory management, but this isn’t quite true. Consider the following simple cache implementation:
<!--more-->
```java
import java.util.HashMap;
import java.util.Map;
/*ATTENTION, BUGS AHEAD.*/
public class UserDataCache {
    private Map<String, Object> cache = new HashMap<>();
    
    public Object getData(String key) {
        if (key == null) {
            throw new NullPointerException("Key should not be null.");
        }
        return cache.get(key);
    }
    
    public void putData(String key, Object data) {
        if (key == null) {
            throw new NullPointerException("Key should not be null.");
        }
        cache.put(key, data);
    }
}
```
If you check above code carefully, you may find that there's a memory leak in side the <code>getData()</code> method. If a cache grows and then shrinks, the objects that were cached will not be garbage collected, even if the program using the cache has no more references to them. This is because the cache maintains obsolete references to these objects. An obsolete reference is simply a reference that will never be dereferenced again. In this case, any references outside of the “active portion” of the element map are obsolete. The active portion consists of the elements whose index is less than size.[^1]

###Solutions to the Memory Leak
Now comes the solutions to this memory leak issue. Here I will present several solutions to memory leak in Java program briefly. Since the point of this blog post is about Reference type, I'll talk more on that.

Solution 1: use <code>java.util.LinkedHashMap</code> instead, code shown as below:
```java
import java.util.LinkedHashMap;
import java.util.Map;

public class LinkedHashMapCache {

    private final static int CACHE_MAX_SIZE = 100;
    private LinkedHashMap<String, Object> cache;

    public LinkedHashMapCache() {
        this.cache = new LinkedHashMap<String, Object>(CACHE_MAX_SIZE, 0.75f, true) {
            protected boolean removeEldestEntry(
                    Map.Entry<String, Object> eldest) {
                // Remove the eldest entry if the size of the cache exceeds the maximum size
                return size() > CACHE_MAX_SIZE;
            }
        };
    }

    public Object getData(String key) {
        if (key == null)
            throw new NullPointerException("Key should not be null.");
        return cache.get(key);
    }

    public void putData(String key, Object data) {
        if (key == null)
            throw new NullPointerException("Key should not be null.");
        cache.put(key, data);
    }
}
```
In above code, whenever the total entries exceed the maximum size of the map, the eldest entry will be overwritten as the latest one, so the memory leak issue is eliminated. More commonly, the useful lifetime of a cache entry is less well defined, with entries becoming less valuable over time. Under these circumstances, the cache should occasionally be cleaned of entries that have fallen into disuse. This can be done by a background thread (perhaps a <code>Timer</code> or <code>ScheduledThreadPoolExecutor</code>) or as a side effect of adding new entries to the cache. The <code>LinkedHashMap</code> class facilitates the latter approach with its <code>removeEldestEntry</code> method.

Solution 2: this is the real meat of this blog post, use <code>java.lang.ref.WeakHashMap</code> instead of <code>java.util.HashMap</code>
```java
import java.util.Map;
import java.util.WeakHashMap;

public class WeakHashMapCache {
    private Map<String, Object> map = new WeakHashMap<>();

    /* This put() will return any data associated with key. */
    public void put(String key, Object data) {
        if (key == null)
            throw new NullPointerException("Key cannot be null.");
        map.put(key, data);
    }

    /* This get() will return the data specified by key, or null if no
     data was put there, or if the object was garbage collected. */
    public Object get(String key) {
        if (key == null)
            throw new NullPointerException("Key cannot be null.");
        return map.get(key);
    }
}
```
Why could the <code>java.lang.ref.WeakHashMap</code> come to rescue? Let's dig deeper into what is weak references, how to use them, and when to use them.

###Description in [Javadoc package summary](http://docs.oracle.com/javase/7/docs/api/java/lang/ref/package-summary.html) of java.lang.ref
> Provides reference-object classes, which support a limited degree of interaction with the garbage collector. A program may use a reference object to maintain a reference to some other object in such a way that the latter object may still be reclaimed by the collector. A program may also arrange to be notified some time after the collector has determined that the reachability of a given object has changed.

So what is reachability, here is the explanation from that document too:
>Going from strongest to weakest, the different levels of reachability reflect the life cycle of an object. They are operationally defined as follows:

* An object is strongly reachable if it can be reached by some thread without traversing any reference objects. A newly-created * object is strongly reachable by the thread that created it.
* An object is softly reachable if it is not strongly reachable but can be reached by traversing a soft reference.
* An object is weakly reachable if it is neither strongly nor softly reachable but can be reached by traversing a weak reference. When the weak references to a weakly-reachable object are cleared, the object becomes eligible for finalization.
* An object is phantom reachable if it is neither strongly, softly, nor weakly reachable, it has been finalized, and some phantom reference refers to it.
* Finally, an object is unreachable, and therefore eligible for reclamation, when it is not reachable in any of the above ways.

At this point, you must be wondering about the difference, and the need, to have three different levels of weaker referencing mechanisms. In the order of strength, the references can be arranged as:
**Strong References > Soft References > Weak References > Phantom References**


###Strong References [^2]
The references that we create using the assignment operator are known as strong references, because the instance is strongly referred by the application, making it ineligible for garbage collection.
```java
Object o = new Object();
```
Soft, Weak and Phantom references are the weaker counterparts of referencing, where the garbage collection algorithm is allowed to mark an instance to be garbage collected, even though such a reference exists. What this means is that, even though you hold a weak reference to a particular instance, the JVM can sweep it out of the memory if it needs to. This works out great for the problem we discussed before, since instances in our cache will be automatically released if the JVM thinks it needs more memory for other parts.

###Soft References
According to the document of [SoftReference](http://docs.oracle.com/javase/7/docs/api/java/lang/ref/SoftReference.html), the JVM implementations are encouraged not to clear out a soft reference if the JVM has enough memory. That is, if free heap space is available, chances are that a soft reference will not be freed during a garbage collection cycle (so it survives from GC).  However, before throwing an <code>OutOfMemoryError</code>, JVM will attempt to reclaim memory by releasing instances that are softly reachable.  This makes Soft References ideal for implementing memory sensitive caches (as in our example problem).
```java
public class SoftReferenceSample {
    public static void main(String[] args) {

        // Initial Strong Ref
        Object obj = new Object();
        System.out.println("Instance : " + obj);

        // Make a Soft Reference on obj
        SoftReference<Object> softReference = new SoftReference<>(obj);

        // Make obj eligible for GC !
        obj = null;
        System.gc();    // Run GC
        // should be null if GC collected
        System.out.println("Instance : " + softReference.get());
    }
}
```

###Weak References
Unlike Soft References, Weak References can be reclaimed by the JVM during a GC cycle, even though there’s enough free memory available. As long as GC does not occur, we can retrieve a strong reference out of a weak reference by calling the <code>WeakReference#get()</code> method.

Below is an example code of WeakReference, two of other References can be found under <code>java.lang.ref.*</code> package:
```java
import java.lang.ref.WeakReference;

public class WeakReferenceSample {
    public static void main(String[] args) {
        // Initial Strong Ref
        Object obj = new Object();
        System.out.println("Instance : " + obj);

        // Create a Weak Ref on obj
        WeakReference<Object> weakRef = new WeakReference<>(obj);
        // Make obj eligible for GC !
        obj = null;
        // Get a strong reference again. Now its not eligible for GC
        Object strongRef = weakRef.get();
        System.out.println("Instance : " + strongRef);
        // Make the instance eligible for GC again
        strongRef = null;
        // Keep your fingers crossed
        System.gc();
        // should be null if GC collected
        System.out.println("Instance : " + weakRef.get());
    }
}
```
From the little toy program you can see how to create WeakReference instances and how to store/get data into/from it either.

###Phantom References
Phantom references are the weakest form of referencing. Instances that are referred via a phantom reference cannot be accessed directly using a get() method (it always returns null), as in case of Soft / Weak references. Instead, we need to rely on Reference Queues to make use of Phantom References. One use case of Phantom references is to keep track of active references within an application, and to know when those instances will be garbage collected. If we use strong references, then the instance will not be eligible for GC due to the strong reference we maintain. Instead, we could rely on a phantom reference with the support of a reference queue to handle the situation.

###Reference Queues
ReferenceQueue is the mechanism provided by the JVM to be notified when a referenced instance is about to be garbage collected. Reference Queues can be used with all of the reference types by passing it to the constructor. When creating a PhantomReference, it is a must to provide a Reference Queue.

Let's see an example of reference queue:
```java
import java.lang.ref.PhantomReference;
import java.lang.ref.ReferenceQueue;

public class PhantomRefQueueSample {

    public static void main(String[] args) throws InterruptedException {

        Object obj = new Object();
        ReferenceQueue<Object> queue = new ReferenceQueue<>();
        PhantomReference<Object> pRef = new PhantomReference<>(obj, queue);
        obj = null;

        new Thread(new Runnable() {
            public void run() {
                try {
                    System.out.println("Awaiting for GC");
                    // This will block till it is reclaimed by JVM
                    PhantomReference pRef = (PhantomReference) queue.remove();
                    System.out.println("Referenced reclaimed");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();

        // Wait for 2nd thread to start
        Thread.sleep(2000);

        System.out.println("Invoking GC");
        System.gc();
    }
}
```

###WeakHashMap
<code>java.util.WeakHashMap</code> is a special version of the HashMap, which uses weak references as the key. Therefore, when a particular key is not in use anymore, and it is eligible for garbage collection, the corresponding entry in the WeakHashMap will magically disappear from the map. And the magic relies on ReferenceQueue mechanism explained before to identify when a particular weak reference is to be garbage collected. This is useful when you want to build a cache based on weak references. In more sophisticated requirements, it is better to write your own cache implementation.



[^1]: [Effective Java, 2nd edition](http://www.amazon.com/Effective-Java-Edition-Joshua-Bloch/dp/0321356683)
[^2]: [Know the JVM Series -3- When Weaker is Better: Understanding Soft, Weak and Phantom References](http://blog.yohanliyanage.com/2010/10/ktjs-3-soft-weak-phantom-references/)
