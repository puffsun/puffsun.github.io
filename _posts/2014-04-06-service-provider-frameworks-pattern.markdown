---
layout: post
title: "service provider frameworks"
date: 2014-04-06 22:39:44 +0800
comments: true
categories: Java
keywords: JDBC Service Provider Frameworks pattern SPI service provicer mechanism
description: JDBC Service Provider Frameworks pattern SPI service provicer mechanism
---

###Overview
In [Effective Java, second edition](http://www.amazon.com/Effective-Java-Edition-Joshua-Bloch/dp/0321356683), the author mentioned a pattern called Service Provider frameworks. The author wrote:
> A service provider framework is a system in which multiple service providers implement a service, and the system makes the implementations available to its clients, decoupling them from the implementations.

In this blog post, I'll talk about the details of Service Provider Frameworks thru it's application in JDK, to be specific, in JDBC and Codec lookup.
<!--more-->
[^1]As the author said, there are three essential components of a service provider framework: a **service interface**, which providers implement; a **provider registration API**, which the system uses to register implementations, giving clients access to them; and a **service access API**, which clients use to obtain an instance of the service. The service access API typically allows but does not require the client to specify some criteria for choosing a provider. In the absence of such a specification, the API returns an instance of a default implementation. The service access API is the “flexible static factory” that forms the basis of the service provider framework.

An optional fourth component of a service provider framework is a **service provider interface**, which providers implement to create instances of their service implementation. *In the absence of a service provider interface, implementations are registered by class name and instantiated reflectively*.

###Service Provider Frameworks in JDBC
In the case of [JDBC](http://www.oracle.com/technetwork/java/javase/jdbc/index.html), [Connection](http://docs.oracle.com/javase/7/docs/api/java/sql/Connection.html) plays the part of the service interface, [DriverManager.registerDriver](http://docs.oracle.com/javase/7/docs/api/java/sql/DriverManager.html#registerDriver\(java.sql.Driver\)) is the provider registration API, [DriverManager.getConnection](http://docs.oracle.com/javase/7/docs/api/java/sql/DriverManager.html#getConnection\(java.lang.String\)) is the service access API, and [Driver](http://docs.oracle.com/javase/7/docs/api/java/sql/Driver.html) is the service provider interface.

Usually in order to use JDBC, you should execute below code:
```java
      // Register JDBC driver, before JDBC 4.0 only
      Class.forName("com.mysql.jdbc.Driver");

      // Open a connection
      System.out.println("Connecting to database...");
      conn = DriverManager.getConnection(DB_URL,USER,PASS);
```
The <code>Class.forname("com.mysql.jdbc.Driver")</code> can be eliminated after JDBC 4.0, as you can see in the document of [DriverManager](http://docs.oracle.com/javase/7/docs/api/java/sql/DriverManager.html)
> The DriverManager methods getConnection and getDrivers have been enhanced to support the Java Standard Edition [Service Provider](http://docs.oracle.com/javase/7/docs/technotes/guides/jar/jar.html#Service%20Provider) mechanism. JDBC 4.0 Drivers must include the file META-INF/services/java.sql.Driver. This file contains the name of the JDBC drivers implementation of java.sql.Driver. For example, to load the my.sql.Driver class, the META-INF/services/java.sql.Driver file would contain the entry:
> my.sql.Driver
> Applications no longer need to explicitly load JDBC drivers using <code>Class.forName()</code>. Existing programs which currently load JDBC drivers using Class.forName() will continue to work without modification.

###Service Provider mechanism[^2]
Now let's dig deeper into the **Service Provider mechanism** and **Service Provider configuration file**, you can find it [here](http://docs.oracle.com/javase/7/docs/technotes/guides/jar/jar.html#Service%20Provider):

Files in the <code>META-INF/services</code> directory are service provider configuration files. A service is a well-known set of interfaces and (usually abstract) classes. A service provider is a specific implementation of a service. The classes in a provider typically implement the interfaces and subclass the classes defined in the service itself. Service providers may be installed in an implementation of the Java platform in the form of extensions, that is, jar files placed into any of the usual extension directories. Providers may also be made available by adding them to the applet or application class path or by some other platform-specific means.

A service is represented by an abstract class. A provider of a given service contains one or more concrete classes that extend this service class with data and code specific to the provider. This provider class will typically not be the entire provider itself but rather a proxy that contains enough information to decide whether the provider is able to satisfy a particular request together with code that can create the actual provider on demand. The details of provider classes tend to be highly service-specific; no single class or interface could possibly unify them, so no such class has been defined. The only requirement enforced here is that provider classes must have a zero-argument constructor so that they may be instantiated during lookup.

###Provider-Configuration File
A service provider identifies itself by placing a provider-configuration file in the resource directory <code>META-INF/services</code>. The file's name should consist of the fully-qualified name of the abstract service class. The file should contain a newline-separated list of unique concrete provider-class names. Space and tab characters, as well as blank lines, are ignored. The comment character is '#' (0x23); on each line all characters following the first comment character are ignored. The file must be encoded in UTF-8.

###Example of Service Provider mechanism
Suppose we have a service class named <code>java.io.spi.CharCodec</code>. It has two abstract methods:
```java
public abstract CharEncoder getEncoder(String encodingName);
public abstract CharDecoder getDecoder(String encodingName);
```
Each method returns an appropriate object or null if it cannot translate the given encoding. Typical CharCodec providers will support more than one encoding.

If <code>sun.io.StandardCodec</code> is a provider of the <code>CharCodec</code> service then its jar file would contain the file <code>META-INF/services/java.io.spi.CharCodec</code>. This file would contain the single line:
<code>sun.io.StandardCodec    # Standard codecs for the platform</code>
To locate an encoder for a given encoding name, the internal I/O code would do something like this:
```java
   CharEncoder getEncoder(String encodingName) {
       Iterator ps = Service.providers(CharCodec.class);
       while (ps.hasNext()) {
           CharCodec cc = (CharCodec)ps.next();
           CharEncoder ce = cc.getEncoder(encodingName);
           if (ce != null)
               return ce;
       }
       return null;
   }
```
The provider-lookup mechanism always executes in the security context of the caller. Trusted system code should typically invoke the methods in this class from within a privileged security context.

###How Service Provider mechanism works in JDBC
Now we can draw the conclusion that with service provider configuration file, during jar loading, the loading thread will find the configuration first, then register the services by name from the configuration file. The service configuration file contains the name of the JDBC driver's implementation of <code>java.sql.Driver</code>. For example, to load the JDBC driver to connect to a Apache Derby database, the <code>META-INF/services/java.sql.Driver</code> file would contain the following entry:
<code>org.apache.derby.jdbc.EmbeddedDriver</code>. Then it will call <code>DriverManager.registerDriver</code> to register specific Database driver automatically, so you don't need to call <code>Class.forName()</code> yourself. After that, you can access the service thru **Service Access Interface**, <code>java.sql.DriverManager.getConnection</code> to get the service, which in this case, is a implementation of <code>java.sql.Connection</code>.

However, before JDBC 4.0, you don't have the facility, you have to register database driver manually by calling <code>Class.forName</code> to register the driver before accessing the service interface.

[^1]: [Effective Java, second edition](http://www.amazon.com/Effective-Java-Edition-Joshua-Bloch/dp/0321356683)
[^2]: [jar file specification#Service Provider](http://docs.oracle.com/javase/7/docs/technotes/guides/jar/jar.html#Service%20Provider) 
