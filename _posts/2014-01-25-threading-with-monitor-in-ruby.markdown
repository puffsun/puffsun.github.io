---
layout: post
title: "Threading with Monitor in Ruby"
date: 2014-01-25 00:45:07 +0800
comments: true
categories: ["Ruby"]

keywords: "Todo, Thread, Ruby, Deadlock, Monitor, Monitormixin, sychronize"
description: "How to write multi-threaded code leverage MonitorMixin in Ruby"
---

Write multi-threaded program is pretty straightforward in Ruby, there's a Thread class that help to achieve parallelism code. If you didn't know thread or don't know how to write multi-threaded code in Ruby, I encourage you stop reading this blog post, move to this brilliant tutorial [Ruby Multithreading](http://www.tutorialspoint.com/ruby/ruby_multithreading.htm) to learn the basics of the Thread in Ruby.

Well, this blog post will show you how to make your life easier as a Ruby developer when you faced with multi-threading -- the MonitorMixin library, it's one of my favourite libraries. It make the task of writing complex synchronisation logic easy which is hard to write using a simple exclusive locking mechanism. MonitorMixin lets you write a nested lock, so you can use it as a more convenient version of plain old Mutex.

Let's see an example with deadlock, then we resolve it with MonitorMixin library.
<!--more-->

Here's the defected code:
```ruby
require 'thread'

class Todo
  def initialize
    @items = {}
    @serial = 0
    # Exclusive mutex instance
    @mutex = Mutex.new
  end

  def [](key)
    # No need to synchronize here because we only read value here
    # But keep in mind that in Ruby you can redefine method,
    # so this method may not behave as it should be.
    @items[key]
  end

  def add(str)
    @mutex.synchronize do
      # Deadlock happens here.
      key = serial
      @items[@key] = str
      return key
    end
  end

  def delete(key)
    @mutex.synchronize do
      @items.delete(key)
    end
  end

  def print_todos
    @mutex.synchronize do
      @items.keys.sort.collect { |item| [item, @items[item]] }
    end
  end

  private

  # Generate next serial number
  def serial
    @mutex.synchronize do
      @serial += 1
      return @serial
    end
  end
end
```
Let's try it in interactive Ruby, note that I use [Pry](http://pryrepl.org/) as replacement of plain old irb, you should give it a try too, it's pretty powerful.

```ruby
[30] pry(main)> require_relative 'deadlock_example'
=> true
[31] pry(main)> todo = Todo.new
=> #<Todo:0x007f866977fb90 @items={}, @mutex=#<Mutex:0x007f866977fb40>, @serial=0>
[32] pry(main)> todo.add('Item 1')
ThreadError: deadlock; recursive locking
from /Users/gsun/prog/ruby/george/deadlock_example.rb:39:in `synchronize'
[33] pry(main)>
```
Sure enough, we have a deadlock on our hands. These types of bugs can be difficult to track down in real life programming.

Now let's see how to use MonitorMixin to enable nested lock, above example was refactored like this:
```ruby
require 'monitor'

class Todo
  include MonitorMixin

  def initialize
    # Remember to call super in constructor to initialize MonitorMixin
    # Also no need the @mutex instance at all
    super
    @items = {}
    @serial = 0
  end

  def [](key)
    # No need to synchronize here because we only read value here
    # But keep in mind that in Ruby you can redefine method,
    # so this method may not behavior as it should be.
    @items[key]
  end

  def add(str)
    synchronize do
      # Deadlock happens here.
      key = serial
      @items[@key] = str
      return key
    end
  end

  def delete(key)
    synchronize do
      @items.delete(key)
    end
  end

  def print_todos
    synchronize do
      @items.keys.sort.collect { |item| [item, @items[item]] }
    end
  end

  private

  # Generate next serial number
  def serial
    synchronize do
      @serial += 1
      return @serial
    end
  end
end
```

Now let's try it in Pry:
```ruby
[1] pry(main)> require_relative 'example'
=> true
[2] pry(main)> todo = Todo.new
=> #<Todo:0x007ff1f2a30a98
 @items={},
 @mon_count=0,
 @mon_mutex=#<Mutex:0x007ff1f2a30a48>,
 @mon_owner=nil,
 @serial=0>
[3] pry(main)> todo.add('Item 1')
=> 1
[4] pry(main)>
```
The deadlock is gone since the synchronize method can handle nested locks, you don't have to worry about having a deadlock, unlike the Mutex version. If you know Java, you can see the synchronize method is similar to the synchronized keyword in Java. 

The moral is that we should consider MonitorMixin library for every case that need synchronize, even the simplest one.

