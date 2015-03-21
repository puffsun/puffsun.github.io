---
layout: post
title: "solutions to scala for the impatient - chapter5"
date: 2014-03-09 20:03:35 +0800
comments: true
categories: Scala
keywords: Scala for the impatient solutions
description: Scala for the impatient solutions
---

1\. Improve the Counter class in Section 5.1, “Simple Classes and Parameterless Methods,” on page 49 so that it doesn’t turn negative at <code>Int.MaxValue</code>.

```scala
class Counter {
  private var value = 0

  def increment() {
    if ((value + 1) < Int.MaxValue)
      value += 1
  }
  def current = value
}
```
<!--more-->
2\. Write a class <code>BankAccount</code> with methods <code>deposit</code> and <code>withdraw</code>, and a read-only property <code>balance</code>.

```scala
class BankAccount(private var _balance: Double = 0.0) {

  def deposit(amount: Double) = {
    _balance += amount
  }

  def withdraw(amount: Double) = {
    if (balance > amount) {
      _balance -= amount
      true
    } else {
      false
    }
  }

  def balance = _balance
}
```

3\. Write a class <code>Time</code> with read-only properties <code>hours</code> and <code>minutes</code> and a method <code>before(other: Time): Boolean</code> that checks whether this time comes before the other. A <code>Time</code> object should be constructed as <code>new Time(hrs, min)</code>, where hrs is in military time format (between 0 and 23).
```scala
class Time(val hours: Int, val minutes: Int) {
  // Validations goes here.
  if (hours > 23 || hours < 0) {
    throw new IllegalArgumentException("should larger than 0 or smaller than 23")
  }

  if (minutes < 0 || minutes > 59) {
    throw new IllegalArgumentException("should larger than 0 or small than 59")
  }

  def before(other: Time): Boolean = {
    if (hours < other.hours) {
      if (minutes <  other.minutes) {
        return true
      }
    }
    false
  }
}
```

4\. Reimplement the Time class from the preceding exercise so that the internal representation is the number of minutes since midnight (between 0 and 24 × 60 – 1). Do not change the public interface. That is, client code should be unaffected by your change.

```scala
class Time(private val _hours: Int, private val _minutes: Int) {

  if (hours > 23 || hours < 0) {
    throw new IllegalArgumentException("should larger than 0 or smaller than 23")
  }

  if (minutes < 0 || minutes > 59) {
    throw new IllegalArgumentException("should larger than 0 or small than 59")
  }

  private val _total_minutes = _hours * 60 + _minutes

  def minutes = _minutes
  def hours = _hours

  def before(other: Time): Boolean = {
    if (_total_minutes < other._total_minutes)
      true
    else
      false
  }
}
```

5\. Make a class <code>Student</code> with read-write JavaBeans properties name (of type <code>String</code>) and id (of type <code>Long</code>). What methods are generated? (Use javap to check.) Can you call the JavaBeans getters and setters in Scala Should you?[^1]
```scala
import scala.reflect.BeanProperty

class Student(@BeanProperty var name: String, @BeanProperty var id: Long) {  
}

object Main extends App {
  val human = new Student("Dave Grohl", 1337)
  
  // You can call the JavaBeans getter and setter, just like this
  human.setName("Kurt Cobain")
  human.setId(5000)
  
  // But why would you, when the Scala-esque methods are generated?
  human.name = "Krist Novoselic"
  human.id = 5001
}
```

The output of javap as below:
```
$ javap Student
// Compiled from "ch5_exercises.scala"
public class Student {
  public java.lang.String name();
  public void name_$eq(java.lang.String);
  public void setName(java.lang.String);
  public long id();
  public void id_$eq(long);
  public void setId(long);
  public java.lang.String getName();
  public long getId();
  public Student(java.lang.String, long);
}
```
You can see that Scala generate getters/setters for you, as well as <code>name</code>, <code>name\_=</code>, <code>id</code>, and <code>id_=</code>.

6\. In the <code>Person</code> class of Section 5.1, “Simple Classes and Parameterless Methods,” on page 49, provide a primary constructor that turns negative ages to 0.
```scala
class Person(val _name: String, var _age: Int) {
  if (_age < 0)
    _age = 0

  def age = _age
  def age_=(new_age: Int) {
    if (new_age > _age)
      _age = new_age
  }
}
```

7\. Write a class <code>Person</code> with a primary constructor that accepts a string containing a first name, a space, and a last name, such as <code>new Person("Fred Smith")</code>. Supply read-only properties <code>firstName</code> and <code>lastName</code>. Should the primary constructor parameter be a var, a val, or a plain parameter? Why?
```scala
class Person(val fullname: String) {
  if (fullname == null || fullname.split(" ").size < 2)
    throw new IllegalArgumentException("Name is not be in format of first_name last_name")
  private val tmp_name = fullname.split(" ")
  val firstName = tmp_name(0)
  val lastName = tmp_name(1)
}
```
As the primary constructor parameter name suggests, it should be a val to prevent it from being changed, at the same time, it should have a public getter method so that others can get the person's full name.

8\. Make a class <code>Car</code> with read-only properties for <code>manufacturer</code>, <code>model name</code>, and <code>model year</code>, and a read-write property for the <code>license plate</code>. Supply four constructors. All require the <code>manufacturer</code> and <code>model name</code>. Optionally, <code>model year</code> and <code>license plate</code> can also be specified in the constructor. If not, the <code>model year</code> is set to -1 and the <code>license plate</code> to the empty string. Which constructor are you choosing as the primary constructor? Why?

We only need primary constructor here, it can take default parameters, which we can leverage to eliminate the auxiliary constructor, as below code shown:
```scala
class Car(val manufacturer: String, 
    val modelName: String, 
    val modelYear: Int = -1, 
    var licensePlate: String = "") {
}
```

9\. Reimplement the class of the preceding exercise in Java, C#, or C++ (your choice). How much shorter is the Scala class?

Equivalent Java code as below: 
```java
public class Car {
    private String manufacturer;
    private String modelName;
    private int modelYear;
    private String licensePlate;

    public Car(String manufacturer, String modelName, int modelYear, String licensePlate) {
        this.manufacturer = manufacturer;
        this.modelName = modelName;
        this.modelYear = modelYear;
        this.licensePlate = licensePlate;
    }

    public Car(String manufacturer, String modelName) {
        this(manufacturer, modelName, -1, "");
    }

    public Car(String manufacturer, String modelName, int modelYear) {
        this(manufacturer, modelName, modelYear, "");
    }

    public Car(String manufacturer, String modelName, String licensePlate) {
        this(manufacturer, modelName, -1, licensePlate);
    }

    public String getManufacturer() {
        return manufacturer;
    }

    public String getModelName() {
        return modelName;
    }

    public int getModelYear() {
        return modelYear;
    }

    public String getLicensePlate() {
        return licensePlate;
    }

    public void setLicensePlate(String licensePlate) {
        this.licensePlate = licensePlate;
    }
}

```

10\. Consider the class
```scala
class Employee(val name: String, var salary: Double) {
  def this() { this("John Q. Public", 0.0) }
}
```
Rewrite it to use explicit fields and a default primary constructor. Which form do you prefer? Why?

```scala
class Employee(val name: String = "John Q. Public", var salary: Double = 0.0) {}
```
I prefer to above code, using primary constructor with default parameters, also there's no explicit getters/setters. After falling love with Ruby programming language, I prefer to any coding style that can lead to concise and short code, most of time, short code is easy to maintain, to test and to develop.

[^1]: [Scala for the Impatient, by McDamon](https://bitbucket.org/McDamon/scalaimpatient/src/4a11167459b2/ch05/answers.txt)
