---
layout: post
title: "Singleton pattern in Java"
date: 2014-04-07 06:54:11 +0800
comments: true
categories: Java
keywords: Singleton Design Pattern AccessibleObject Java MultiThreading Classloader
description: Singleton Design Pattern AccessibleObject Java MultiThreading Classloader
---

The Singleton pattern is deceptively simple, even and especially for Java developers, but it presents a number of pitfalls for the unwary Java developer which make it hard to implement properly. In this article, I'll talk about several ways to implement the Singleton pattern in Java, and leave it to you to decide which one is best suited for your circumstance depending on your requirement.
<!--more-->
With the Singleton design pattern you can:

* Ensure that only one instance of a class is created
* Provide a global point of access to the object
* Allow multiple instances in the future without affecting a singleton class's clients

### The classic Singleton pattern[^1]
In Design Patterns: [Elements of Reusable Object-Oriented Software](http://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612), the GoF describe the Singleton pattern like this:
> Ensure a class has only one instance, and provide a global point of access to it.

```java
public class ClassicSingleton {
    private static ClassicSingleton instance = null;

    private ClassicSingleton() {
        // Exists only to defeat instantiation.
    }

    public static ClassicSingleton getInstance() {
        if (instance == null) {
            instance = new ClassicSingleton();
        }
        return instance;
    }
}
```

Above code is easy to understand, the ClassicSingleton hold a static reference to the single instance and returns that reference from the static <code>getInstance()</code> method.

There are several interesting points concerning the ClassicSingleton class. 
1. ClassicSingleton employs a technique known as lazy instantiation to create the singleton; as a result, the singleton instance is not created until the <code>getInstance()</code> method is called for the first time. This technique ensures that singleton instances are created only when needed.
2. It's possible to have multiple singleton instances if classes loaded by different Classloaders access a singleton. That scenario is not so far-fetched; for example, some servlet containers use distinct Classloaders for each servlet, so if two servlets access a singleton, they will each have their own instance.
3. If ClassicSingleton implements the <code>java.io.Serializable</code> interface, the class's instances can be serialized and deserialized. However, if you serialize a singleton object and subsequently deserialize that object more than once, you will have multiple singleton instances.
4. ClassicSingleton class is not thread-safe. If two threads—we'll call them Thread 1 and Thread 2—call <code>ClassicSingleton.getInstance()</code> at the same time, two ClassicSingleton instances can be created if Thread 1 is preempted just after it enters the if block and control is subsequently given to Thread 2.
5. A privileged client can invoke the private constructor reflectively with the aid of the <code>AccessibleObject.setAccessible</code> method. If you need to defend against this attack, modify the constructor to make it throw an exception if it’s asked to create a second instance.

As you can see from the preceding discussion, although the Singleton pattern is one of the simplest design patterns, implementing it in Java is anything but simple. The rest of this article addresses Java-specific considerations for the Singleton pattern.

###Synchronization for multithreading considerations
Making Singleton thread-safe is easy-just synchronize the <code>getInstance()</code> method:
```java
public class Singleton {
    private static Singleton instance = null;

    private Singleton() {
        // Exists only to defeat instantiation.
    }

    public synchronized static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```
However, the astute reader may realize that the getInstance() method only needs to be synchronized the first time it is called. Because synchronization is very expensive performance-wise, perhaps we can introduce a performance enhancement that only synchronizes the singleton assignment in <code>getInstance()</code>.

###A performance enhancement
In search of a performance enhancement, you might choose to rewrite the <code>getInstance()</code> method like this:
```java
// CAUTION, BUGS AHEAD
public static Singleton getInstance() {
   if(singleton == null) {
      synchronized(Singleton.class) { 
         singleton = new Singleton();
      }
   }
   return singleton;
}
```
Instead of synchronizing the entire method, the preceding code fragment only synchronizes the critical code. However, the preceding code fragment is not thread-safe. Consider the following scenario: Thread 1 enters the synchronized block, and, before it can assign the singleton member variable, the thread is preempted. Subsequently, another thread can enter the if block. The second thread will wait for the first thread to finish, but we will still wind up with two distinct singleton instances. Is there a way to fix this problem? Read on.

###Double-checked locking
Double-checked locking is a technique that, at first glance, appears to make lazy instantiation thread-safe. That technique is illustrated in the following code fragment:
```java
// CAUTION, BUGS AHEAD
public static Singleton getInstance() {
  if(singleton == null) {
     synchronized(Singleton.class) {
       if(singleton == null) {
         singleton = new Singleton();
       }
    }
  }
  return singleton;
}
```
What happens if two threads simultaneously access <code>getInstance()</code>? Imagine Thread 1 enters the synchronized block and is preempted. Subsequently, a second thread enters the if block. When Thread 1 exits the synchronized block, Thread 2 makes a second check to see if the singleton instance is still null. Since Thread 1 set the singleton member variable, Thread 2's second check will fail, and a second singleton will not be created. Or so it seems.

Unfortunately, double-checked locking is not guaranteed to work because the compiler is free to assign a value to the singleton member variable before the singleton's constructor is called. If that happens, Thread 1 can be preempted after the singleton reference has been assigned, but before the singleton is initialized, so Thread 2 can return a reference to an uninitialized singleton instance.

Since double-checked locking is not guaranteed to work, you must synchronize the entire getInstance() method. However, another alternative is simple, fast, and thread-safe.

###An alternative thread-safe singleton implementation
```java
public class Singleton {
   public final static Singleton INSTANCE = new Singleton();
   private Singleton() {
       // Exists only to defeat instantiation.
   }
}
```
The preceding singleton implementation is thread-safe because static member variables created when declared are guaranteed to be created the first time they are accessed. You get a thread-safe implementation that automatically employs lazy instantiation, until the class file get loaded into memory, their is no instance instanciated at all.

Of course, like nearly everything else, the preceding singleton is a compromise; if you use that implementation, you can't change your mind and allow multiple singleton instances later on. With a more conservative singleton implementation, instances are obtained through a <code>getInstance()</code> method, and you can change those methods to return a unique instance or one of hundreds. You can't do the same with a public static member variable.
```java
public class Singleton {
   private final static Singleton INSTANCE = new Singleton();
   private Singleton() {
       // Exists only to defeat instantiation.
   }
   
   public static Singleton getInstance() {
       return INSTANCE;
   }
}
```
###Leveraging volatile to change Java Memory Model
We can recheck the instance again in synchronized block to ensure that only one instance of the Singleton object be instantiated, shown as below code:
```java
public class LazySingleton {
    private static volatile LazySingleton instance;

    // private constructor
    private LazySingleton() {
    }

    public static LazySingleton getInstance() {
        if (instance == null) {
            synchronized (LazySingleton.class) {
                // Double check
                if (instance == null) {
                    instance = new LazySingleton();
                }
            }
        }
        return instance;
    }
}
```
Please ensure to use <code>volatile</code> keyword with instance variable otherwise you can run into out of order write error scenario, where reference of instance is returned before actually the object is constructed i.e. JVM has only allocated the memory and constructor code is still not executed. In this case, your other thread, which refer to uninitialized object may throw null pointer exception and can even crash the whole application.

Although above code is the correct way to implement Singleton pattern, this is not recommended since it introduce extra complexity of code, which may introduce subtle bugs.

###Classloaders
Because multiple classloaders are commonly used in many situations—including servlet containers—you can wind up with multiple singleton instances no matter how carefully you've implemented your singleton classes. If you want to make sure the same classloader loads your singletons, you must specify the classloader yourself; for example:
```java
private static Class getClass(String classname) throws ClassNotFoundException {
    ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
    if(classLoader == null)
        classLoader = Singleton.class.getClassLoader();
        return (classLoader.loadClass(classname));
    }
}
```
The preceding method tries to associate the classloader with the current thread; if that classloader is null, the method uses the same classloader that loaded a singleton base class. The preceding method can be used instead of Class.forName().

###Serialization[^2]
If you serialize a singleton and then deserialize it twice, you will have two instances of your singleton, unless you implement the <code>readResolve()</code> method, like this:
```java
public class Singleton implements java.io.Serializable {
    private static final long serialVersionUID = 1L;
    public static Singleton INSTANCE = new Singleton();
    
    private Singleton() {
        // Exists only to thwart instantiation.
    }
   
    private Object readResolve() {
        return INSTANCE;
    }
}
```
The previous singleton implementation returns the lone singleton instance from the readResolve() method; therefore, whenever the Singleton class is deserialized, it will return the same singleton instance. **Don't forget to add serial version id in this case, or you will get an exception during de-serialise process**.

###Bill Pugh solution[^3]
Bill Pugh was main force behind java memory model changes. His principle “Initialization-on-demand holder idiom” also uses static block but in different way. It suggest to use static inner class:
```java
public class BillPughSingleton {
    private BillPughSingleton() {
    }

    private static class LazyHolder {
        private static final BillPughSingleton INSTANCE = new BillPughSingleton();
    }

    public static BillPughSingleton getInstance() {
        return LazyHolder.INSTANCE;
    }
}
```
As you can see, until we need an instance, the LazyHolder class will not be initialized until required and you can still use other static members of BillPughSingleton class. This is the solution, i will recommend to use. I also use it in my all projects.

###Using Enum
As of release 1.5, there is a another approach to implementing singletons. which provide implicit support for thread safety and only one instance is guaranteed. This is also a good way to have singleton with minimum effort. Simply make an enum type with one element:
```java
// Enum singleton - the preferred approach
public enum LazyEnumSingleton {
    INSTANCE;
    public void otherMethods() { System.out.println("Other methods go"); }
}
```

This approach is functionally equivalent to the public field approach, except that it is more concise, provides the serialization machinery for free, and provides an ironclad guarantee against multiple instantiation, even in the face of sophisticated serialization or reflection attacks. While this approach has yet to be widely adopted, **a single-element enum type is the best way to implement a singleton**. JVM does the lazy-loading in a thread-safe manner. That's why using this kind of lazy class loading is the preferred method—the JVM handles all the synchronization. We can see that with below little example code, try run it your self:
```java
public class LazyEnumSingletonExample {
    public static void main(String[] args) throws InterruptedException {
        System.out.println("Accessing enum for the first time.");
        LazyEnumSingleton lazy = LazyEnumSingleton.INSTANCE;
        System.out.println("Done.");
    }
}

enum LazyEnumSingleton {
    INSTANCE;

    static {
        System.out.println("Initialize in process...");
    }
}
```
Output of the code:
```
Accessing enum for the first time.
Initialize in process...
Done.
```
In Java, enum can hold state, which makes the single-element enum type the perfect condidate for Singleton pattern, see below code:
```java
enum LazyEnumSingletonWithState {
    INSTANCE("State") {

        @Override
        public String toOverwrittenOps(String str) {
            return str + this.state;
        }
    };

    private String state;

    private LazyEnumSingletonWithState(String state) {
        this.state = state;
    }

    public String otherOps(String arg) {
        return arg + state;
    }

    @Override
    public String toString() {
        return this.state;
    }

    public abstract String toOverwrittenOps(String str);
}
```

###Preventing privileged clients
As we mentioned before, single pattern can be broken with reflection, as below code shown:
```java
public static void main(String[] args) throws Exception {
    Singleton s1 = Singleton.getInstance();
    Class clazz = Singleton.class;
    Constructor cons = clazz.getDeclaredConstructor();
    cons.setAccessible(true);
    // Another instance can be instantiated now.
    Singleton s2 = (Singleton) cons.newInstance();
}
```
It's easy to get over this issue:
```java
public class PrivilegedSingleton {
    private static final PrivilegedSingleton INSTANCE = new PrivilegedSingleton();

    private PrivilegedSingleton() {

        // Check if we already have an instance
        if (INSTANCE != null) {
            throw new IllegalStateException("Singleton instance already created.");
        }
    }

    public static PrivilegedSingleton getInstance() {
        return INSTANCE;
    }
}
```
Sometimes, your IDE may remind you that the <code>INSTANCE</code> variable will never be null, don't rely on that, IDE can make mistake too.

[^1]: [Simply Singleton: Navigate the deceptively simple Singleton pattern](http://www.javaworld.com/article/2073352/core-java/simply-singleton.html)
[^2]: [Effective Java, 2nd edition](http://www.amazon.com/Effective-Java-Edition-Joshua-Bloch/dp/0321356683)
[^3]: [Singleton design pattern in Java](http://howtodoinjava.com/2012/10/22/singleton-design-pattern-in-java/)
