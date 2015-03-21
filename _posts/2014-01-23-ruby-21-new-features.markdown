---
layout: post
title: "Ruby 2.1 new features"
date: 2014-01-23 11:22:29 +0800
comments: true
categories: ["Ruby"]

keywords: "Ruby 2.1, new features, Refinements, Rational and Complex Number literals, Required keywords, GC"
description: "What's new in Ruby 2.1"
---


Ruby 2.1 has been released several days before. It brought a number of optimisations as well as smaller and useful features. In this article, I will go through several very welcoming features, without further ado, let's move on to the world of Ruby 2.1.

###Installing Ruby 2.1
If you're on rvm(run `rvm get head` to get 2.1.0): 

```bash
$ rvm get head
$ rvm install ruby-2.1.0
$ rvm use ruby-2.1.0
```
Or if you like me, using `rbenv`:

```bash
$ rbenv install 2.1.0
$ rbenv rehash
$ rbenv shell 2.1.0
```
To make sure you're using the newly installed Ruby, run:

```bash
$ ruby -v
```
<!--more-->
###New features that we'll talk about
For complete release notes, go ahead to <https://github.com/ruby/ruby/blob/v2_1_0/NEWS>. Below is the list of features we'll talk about.

1. Required Keyword Arguments
2. Rational and Complex Number Literals
3. def-expr Now Return Value
4. Array#to_h & Enumerable#to_h
5. Exception#cause
6. Generational GC a.k.a RGenGC
7. Refinements
8. Object Allocation Tracing
9. Scrubbing Strings

---
#####1. Required Keyword Arguments
In previous Ruby 2.0 release, it introduced keyword arguments:

```ruby
def foo(a: 10)
  puts "a: #{a}"
end
```
Unfortunately, the keyword always require a default value, which you avoid in Ruby 2.1 by using required keyword arguments:

```ruby
def foo(a:)
  puts "a: #{a}"
end
```

#####2. Rational and Complex Number Literals
Ruby 2.1 introduced the i `i` suffix to support Complex numbers, also the `r`suffix to support Rational numbers.

```bash
irb(main):015:0> 10+3i + Complex(5, 6i)
=> (9+3i)
irb(main):014:0> 10/3r
=> (10/3)
```
#####3. def-expr Now Return Value
In previous versions of Ruby, the return value of a method definition has always been nil, now in version 2.1, the return value of def expression is the Symbol of the method name. 

```bash
irb(main):016:0> def foo
irb(main):017:1> end
=> :foo
```
The idea was to allow inlining of private/protected/public keywords and method definitions.

```ruby
module Foo
  def public_method
  end
 
  private def some_other_method
  end
 
  private def a_private_method
  end
end
```
#####4. Array#to_h & Enumerable#to_h
Previously if you want convert an Array object to Hash object, you should do:

```bash
irb(main):023:0> my_array = [[:key1, "one"], [:key2, "two"], [:key3, "three"]]
=> [[:key1, "one"], [:key2, "two"], [:key3, "three"]]
irb(main):024:0> Hash[*my_array.flatten]
=> {:key1=>"one", :key2=>"two", :key3=>"three"}
```
It's a bit tedious, now with Ruby 2.1, you can do:

```bash
my_array.to_h
=> {:key1=>"one", :key2=>"two", :key3=>"three"
```
It's much more simple.
#####5. Exception#cause
Sometimes when rescuing an exception in Ruby, it’s useful to handle the error scenario by raising another, different exception. The trouble with this technique is that it throws away all the information held by the original exception. This makes debugging harder, as there’s no stack trace to follow back to the root cause of the failure.

```ruby
begin
  begin
    raise "Error A"
  rescue => error
    raise "Error B"
  end
rescue => error
  puts "Current failure: #{error.inspect}"
  puts "Original failure:  #{error.cause.inspect}"
end
# >> Current failure: #<RuntimeError: Error B>
# >> Original failure:  #<RuntimeError: Error A>
```
For detailed explanation, go ahead to <http://devblog.avdi.org/2013/12/25/exception-causes-in-ruby-2-1/>.
#####6. Generational GC a.k.a RGenGC
Now almost every programmer knows GC, thanks to Java. In more traditional programming languages, like C/C++, you must manually free resources when you finish using them, this make the program more error prone, also put a huge burden onto programmer.

RGenGC is a new garbage collector known as a [generational garbage collector](http://en.wikipedia.org/wiki/Garbage_collection_(computer_science\)\#Generational_GC_.28ephemeral_GC.29), sometimes referred to as an ephemeral garbage collector. This design of garbage collector leverages the fact that most objects collected by the garbage collector were the objects most recently created. In other words, most objects being collected by the garbage collector were temporary objects, used for a short period in a single method and discarded. Generational garbage collectors collect these objects more effectively, putting less strain on the GC when it searches through the older objects to find unreferenced objects.

Prior to Ruby 2.1, Ruby’s garbage collector was running a conservative stop-the-world mark and sweep algorithm. In Ruby 2.1, we are still using the mark and sweep algorithm to garbage collect the young/old generations. However, because we have lesser objects to mark the marking time decreases, which leads to improved collector performance.

There are caveats, however. In order to preserve compatibility with C extensions, the Ruby core team could not implement a “full” generational garbage collection algorithm. In particular, they could not implement the [moving garbage collection algorithm](http://en.wikipedia.org/wiki/Garbage_collection_(computer_science\)\#Moving_vs._non-moving) – hence the “restricted”.

That said, it is very encouraging to see the Ruby core team taking garbage collection performance very seriously. For more details, do check out [this](http://confreaks.com/videos/2866-rubyconf2013-object-management-on-ruby-2-1) excellent presentation by Koichi Sasada.
#####7. Refinements
[^1]Refinements are no longer experimental in Ruby 2.1. If you are new to refinements, it helps to compare it to monkey patching. In Ruby, all classes are open. This means that we can happily add methods to an existing class.

To appreciate the havoc this can cause, let’s redefine String#count

```ruby
class String
  def count
    Float::INFINITY
  end
end
```
If you were to paste the above into irb, every string returns Infinity when count-ed:

```bash
irb(main):001:0> "I <3 Monkey Patching".count
=> Infinity
```
Refinements provide an alternate way to scope our modifications. Let’s make something slightly more useful:

```ruby
module Permalinker
  refine String do
    def permalinkify
      downcase.split.join("-")
    end
  end
end
 
class Post
  using Permalinker
 
  def initialize(title)
    @title = title
  end
 
  def permalink
    @title.permalinkify
  end
end
```
First, we define a module, Permalinker that refines the String class with a new method. This method implements a cutting edge permalink algorithm.

In order to use our refinement, we simply add using Permalinker into our example Post class. After that, we could treat as if the String class has the permalinkify method.

Let’s see this in action:

```bash
irb(main):002:0> post = Post.new("Refinements are pretty awesome")
irb(main):002:0> post.permalink
=> "refinements-are-pretty-awesome"
```
To prove that String#permalinkify only exists within the scope of the Post class, let’s try using that method elsewhere and watch the code blow up:

```bash
irb(main):023:0> "Refinements are not globally scoped".permalinkify
NoMethodError: undefined method `permalinkify' for "Refinements are not globally scoped":String
        from (irb):23
        from /usr/local/var/rbenv/versions/2.1.0/bin/irb:11:in `<main>'
```

#####8. Object Allocation Tracing
If you have a bloated Ruby application, it is usually a non-trivial task to pinpoint the exact source of the problem. MRI Ruby still doesn’t have profiling tools that can rival, for example, the [JRuby profiler](https://github.com/jruby/jruby/wiki/Profiling-Object-Allocations).

Fortunately, work has begun to provide object allocation tracing to MRI Ruby.

Here’s an example:

```ruby
require 'objspace'
 
class Post
  def initialize(title)
    @title = title
  end
 
  def tags
    %w(ruby programming code).map do |tag|
      tag.upcase
    end
  end
end
 
ObjectSpace.trace_object_allocations_start
a = Post.new("title")
b = a.tags
ObjectSpace.trace_object_allocations_stop
 
puts ObjectSpace.allocation_sourcefile(a) # post.rb
puts ObjectSpace.allocation_sourceline(a) # 16
puts ObjectSpace.allocation_class_path(a) # Class
puts ObjectSpace.allocation_method_id(a)  # new
 
puts ObjectSpace.allocation_sourcefile(b) # post.rb
puts ObjectSpace.allocation_sourceline(b) # 9
puts ObjectSpace.allocation_class_path(b) # Array
puts ObjectSpace.allocation_method_id(b)  # map
```
Although knowing that we can obtain this information is great, it is not immediately obvious how this could be useful to you, the developer.

Enter the allocation_stats gem written by Sam Rawlins.

Let’s install it:

```bash
$ gem install allocation_stats
Fetching: allocation_stats-0.1.2.gem (100%)
Successfully installed allocation_stats-0.1.2
Parsing documentation for allocation_stats-0.1.2
Installing ri documentation for allocation_stats-0.1.2
Done installing documentation for allocation_stats after 0 seconds
1 gem installed
```
Here’s the same example as before, except that we are using allocation_stats this time:

```ruby
require 'allocation_stats'
 
class Post
  def initialize(title)
    @title = title
  end
 
  def tags
    %w(ruby programming code).map do |tag|
      tag.upcase
    end
  end
end
 
stats = AllocationStats.trace do
  post = Post.new("title")
  post.tags
end
 
puts stats.allocations(alias_paths: true).to_text
```
Running this produces a nicely formatted table:

```bash
sourcefile  sourceline  class_path  method_id  memsize  class
----------  ----------  ----------  ---------  -------  ------
post.rb             10  String      upcase           0  String
post.rb             10  String      upcase           0  String
post.rb             10  String      upcase           0  String
post.rb              9  Array       map              0  Array
post.rb              9  Post        tags             0  Array
post.rb              9  Post        tags             0  String
post.rb              9  Post        tags             0  String
post.rb              9  Post        tags             0  String
post.rb             17  Class       new              0  Post
post.rb             17                               0  String
```
Sam gave a wonderful presentation that looks into more details of the allocation_stats gem. 
(This section is quoted from http://www.sitepoint.com/look-ruby-2-1/)


#####9. Strings
String is important to every programming language, Ruby is no excuse, you cannot avoid them in almost every real world program. With the release of Ruby 2.1, there are a few of changes you can use to your advantage.

**String Frozen**
In Ruby, every time you declare a string, a new object is created then the string data copied into it. In other word, if you have a string literal in you method, every time you call the method, a new string object will be created, it may lead to performance issue in real world application.

Of course you can avoid this problem with Symbols because every Symbol instance in Ruby is single instance, you always refer to the same instance.

Unfortunately Symbols are not sting, you cannot split it, index it or search through it. You have to convert the Symbol to a string before performing these actions. You can use symbols when referring to "the thing called," but manipulating them in any way is out of the question. What's needed is something in between a symbol and a mutable string. This is what frozen strings are for. 

In the past, you were able to create frozen strings using "some string"f. Notice the f suffix. This syntax has been removed (or deprecated), and the more standard "some string".freeze method syntax optimised. Frozen strings are created once, and are immutable just like symbols, but all of the traditional string methods (barring those that modify the string) are available. While frozen strings are not new, the optimisation of the .freeze method is new.

**String Scrubbing**

[^2]Scrubbing strings is important when working with multibyte encoded strings (which Ruby defaults to now). "Scrubbing" a string is removing invalid byte sequences from a string while keeping all valid byte sequences intact. This is important when sending strings downstream to "dumber" clients if these clients can conceivably display incorrect data as garbage.

The String#scrub method scrubs all invalid character sequences from a string and returns the new string. It can be called with no arguments like "string".scrub, which replaces all invalid sequences with an empty string ("deleting" them). You can also call it with a simple string replacement, such as "abc\u3042\x81".scrub("X") which replaced the invalid sequence \x81 with X, producing the string "abc\u3042X". You can also use a block, and the docs for the method give a very useful method for debugging: a block that replaces all invalid sequences with their hex equivalent so you can see the invalid sequences without delving into the Ruby debugger or IRB: "abc\u3042\xE3\x80".scrub{|bytes| '<'+bytes.unpack('H*')[0]+'>' }. This will produce the string "abc\u3042".

[^1]: http://www.sitepoint.com/look-ruby-2-1/
[^2]: http://ruby.about.com/od/Whats-New-in-Ruby-210/fl/Whats-new-in-Ruby-210-Strings.htm













