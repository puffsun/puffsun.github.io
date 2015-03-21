---
layout: post
title: "Enum, EnumSet, and EnumMap sample"
date: 2014-04-06 20:50:38 +0800
comments: true
categories: Java
keywords: enum EnumSet EnumMap Collection HashSet HashMap
description: enum EnumSet EnumMap Collection HashSet HashMap
---

[Enum](http://docs.oracle.com/javase/tutorial/java/javaOO/enum.html) was introduced in Java 1.5, although the Java enum can be used with any Java collection, its full power is best leveraged when used with the [EnumMap](http://docs.oracle.com/javase/7/docs/api/java/util/EnumMap.html) and [EnumSet](http://docs.oracle.com/javase/7/docs/api/java/util/EnumSet.html).

Why should you use [EnumMap](http://docs.oracle.com/javase/7/docs/api/java/util/EnumMap.html) and [EnumSet](http://docs.oracle.com/javase/7/docs/api/java/util/EnumSet.html), rather than their counterparts [HashMap](http://docs.oracle.com/javase/7/docs/api/java/util/HashMap.html) and [HashSet](http://docs.oracle.com/javase/7/docs/api/java/util/HashSet.html). The primary reason boil down to performance and memory usage advantage. Let's see their JavaDoc firstly.
<!-- more -->
Document of [EnumMap](http://docs.oracle.com/javase/7/docs/api/java/util/EnumMap.html):
> A specialized Map implementation for use with enum type keys. All of the keys in an enum map must come from a single enum type that is specified, explicitly or implicitly, when the map is created. Enum maps are represented internally as arrays. This representation is extremely compact and efficient.
> ...
> Implementation note: All basic operations execute in constant time. They are likely (though not guaranteed) to be faster than their HashMap counterparts.

Document of [EnumSet](http://docs.oracle.com/javase/7/docs/api/java/util/EnumSet.html):
> A specialized Set implementation for use with enum types. All of the elements in an enum set must come from a single enum type that is specified, explicitly or implicitly, when the set is created. Enum sets are represented internally as bit vectors. This representation is extremely compact and efficient. The space and time performance of this class should be good enough to allow its use as a high-quality, typesafe alternative to traditional int-based "bit flags." Even bulk operations (such as containsAll and retainAll) should run very quickly if their argument is also an enum set.
> ...
> Implementation note: All basic operations execute in constant time. They are likely (though not guaranteed) to be much faster than their HashSet counterparts. Even bulk operations execute in constant time if their argument is also an enum set.

Both of [EnumMap](http://docs.oracle.com/javase/7/docs/api/java/util/EnumMap.html) and [EnumSet](http://docs.oracle.com/javase/7/docs/api/java/util/EnumSet.html) belong to [Java Collections Framework](http://docs.oracle.com/javase/7/docs/technotes/guides/collections/index.html), so they're easy to use because as Java developer, you should familiar with [Java Collections Framework](http://docs.oracle.com/javase/7/docs/technotes/guides/collections/index.html); in the mean time, they're compact and efficient,  they're also enum-powered collections which are easy to use, you can see that in below sample code.

First, let's see an enum deifiniation:
```java
import java.util.EnumSet;

public enum Weekday {
    MONDAY {
        @Override
        public String getDetail() {
            return "Monday";
        }
    },

    TUESDAY {
        @Override
        public String getDetail() {
            return "Tuesday";
        }
    },

    WEDNESDAY {
        @Override
        public String getDetail() {
            return "Wednesday";
        }
    },

    THURSDAY {
        @Override
        public String getDetail() {
            return "Thursday";
        }
    },

    FRIDAY {
        @Override
        public String getDetail() {
            return "Friday";
        }
    },

    SATURDAY {
        @Override
        public String getDetail() {
            return "Saturday";
        }
    },

    SUNDAY {
        @Override
        public String getDetail() {
            return "Sunday";
        }
    };

    public abstract String getDetail();

    public static final EnumSet<Weekday> WORKDAYS = EnumSet.range(MONDAY, FRIDAY);

    public final boolean isWorkday() {
        return WORKDAYS.contains(this);
    }

    public static final EnumSet<Weekday> THE_WHOLE_WEEK = EnumSet.allOf(Weekday.class);
}
```
You can see that one of interesting feature of Java enum is that you can declare **abstract methods**, in above example getDetail() is the abstract method and all the enum fields have implemented it. You can check some other details of Java enum [here](http://docs.oracle.com/javase/tutorial/java/javaOO/enum.html).

Let's move on to our sample code of [EnumMap](http://docs.oracle.com/javase/7/docs/api/java/util/EnumMap.html) and [EnumSet](http://docs.oracle.com/javase/7/docs/api/java/util/EnumSet.html).

```java
import java.util.EnumMap;
import java.util.EnumSet;
import java.util.Map;

public final class EnumSetExample {
    
    public static void main(String... args) {
        System.out.println("Work Schedule:");

        for (Weekday weekday : Weekday.THE_WHOLE_WEEK) {
            String action = weekday.isWorkday()
                    ? "have to work"
                    : "can relax";
            // Enum.ordinal starts with 0
            System.out.println(String.format("%d. On %s you " + action + ".", weekday.ordinal() + 1, weekday));
        }

        System.out.println("Do I have to work the whole week?");

        String result = Weekday.WORKDAYS.containsAll(Weekday.THE_WHOLE_WEEK)
                ? "Yes, unfortunately."
                : "Certainly not.";
        System.out.println(result);

        final EnumSet<Weekday> weekend = Weekday.THE_WHOLE_WEEK.clone();
        weekend.removeAll(Weekday.WORKDAYS);

        System.out.println(String.format("The weekend is %d days long.", weekend.size()));

        // EnumMap example
        EnumMap<Weekday, String> eMap = new EnumMap<>(Weekday.class);
        for (Weekday day : Weekday.THE_WHOLE_WEEK) {
            eMap.put(day, day.getDetail());
        }

        System.out.println("Print weekdays:");
        for (Map.Entry<Weekday, String> entry : eMap.entrySet()) {
            System.out.println(entry.getKey() + "\t" + entry.getValue());
        }
    }
}
```
You can run above source code by yourself, check the result then find related API of [EnumMap](http://docs.oracle.com/javase/7/docs/api/java/util/EnumMap.html) and [EnumSet](http://docs.oracle.com/javase/7/docs/api/java/util/EnumSet.html). The above source code makes use of the following features in Java.[^1]

* enum keyword
* enum custom methods
* enum toString() method
* EnumSet#range()
* EnumSet#allOf()
* EnumSet#clone()
* EnumSet#removeAll()
* EnumSet#size()
* EnumSet iteration
* String#format()
* EnumMap construction and iteration

[^1]:[Fun with EnumSet](https://weblogs.java.net/blog/mkarg/archive/2010/01/03/fun-enumset)
