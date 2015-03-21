---
layout: post
title: "ruby design patterns"
date: 2014-01-23 13:27:14 +0800
comments: true
categories: ["Ruby"]

keywords: "Design patterns, Builder, Factory, Builder, Composite, Decorator, Proxy, Command, Singleton"
description: "Design patterns in Ruby"
---


I just finished reading the great book [Design Patterns in Ruby](http://www.amazon.com/Design-Patterns-Ruby-Russ-Olsen/dp/0321490452). Actually just like me, most of developers know Design Patterns well, what I want to learn from the book is Design Patterns in Ruby style, there is a saying: *Just because you have duck-typing doesn't mean you can ignore common OO idioms!*

In this blog post, I'm going to show you how to implement common design patterns in Ruby, without further ado, let's get started!

#### Factory 
The classic implementation in the book [Design Patterns: Elements of Reusable Object-Oriented Software](http://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612/ref=sr_1_1?s=books&ie=UTF8&qid=1390451541&sr=1-1&keywords=design+patterns) is inheriance-based, this kind of style is hard to find in Ruby code, Ruby programmers are more prefer to more dynamic version, like this:

```ruby
class Product
  #..
end

class Factory
  attr_accessor :product
  def produce
    @product.new
  end
end

fac = Factory.new
fac.product = Product
fac.produce
```
<!--more-->
#### Builder
Ruby's build method is more dynamic, like the code shown:

```ruby
class Director
  def build_with builder
	acc = ''
	[:header, :body, :footer].each do |m|
	  acc += builder.__send__ m if builder.respond_to? m
	end
	acc
  end
end

class HTMLBuilder
  def header; '<html><title>html builder</title>';end
  def body;	  '<body>html builder</body>'        ;end
  def footer; '</html>'                          ;end
end

class XMLBuilder
  def header; '<?xml version="1.0" charset="utf-8">';end
  def body;   '<root>xml builder</root>'            ;end
end

d = Director.new
puts(d.build_with HTMLBuilder.new)
puts(d.build_with XMLBuilder.new)

```
The fun part is that above code will check the method signature, if the object we want to build don't need specify part, we could leave the method unimplemented that's ok to Ruby.

#### Adapter
```ruby
class Adaptee
  def talk; puts 'Adaptee';end
end
class Adapter < Adaptee
  alias talkee talk
  def talk
	puts 'before Adaptee'
	talkee
	puts 'after Adaptee'
  end
end
Adapter.new.talk
```

#### Composite
I'm totally agree with the point that the Composite pattern is built into Ruby with the include keyword. As you know, with composite pattern, there're two concepts: Whole and Parts. In Ruby, you can imagine the the whole is the Mixiner, while the part is the Mixinee.

#### Decorator
This is another demonstration of Ruby's power, it built with a lot of proved best practice. In Ruby, you can implement Decorator pattern by extending modules, like:
```ruby
module Colorful
  attr_acessor :color
end
class Widget
end
w = Widget.new 
w.color = 'blue' #NoMethod error
w.extend Colorful 
w.color = 'blue'
puts w.color
```

#### Proxy
Just like adapter, you can implement Proxy pattern by delegating its interface to another instance. But keep in mind the difference of Proxy and Adapter, with Proxy, you should not change interface, you can use Proxy pattern for caching purpose or for security reasons, just keep interface the same to its subject; but with Adapter pattern, you can transform the interface of the inner object.

#### Command
In Ruby, you can implement Command pattern leverage code blocks:
```ruby
class Command
  def initialize
	@executors = []
  end

  def add_executor &block
	@executors << block
  end
  
  def execute
	@executors.each {|x| x.call }
  end
end

c = Command.new
c.add_executor{ puts 'executor 1' }
c.add_executor{ puts 'executor 2' }
c.execute
```

#### Template method
Here is code snippet quoted from the book:
```ruby
require 'webrick'
class HelloServer < WEBrick::GenericServer
  def run(socket)
    socket.print('Hello TCP/IP world')
  end
end
```
It'a simple HTTP server implementation, WEBrick library complete most of heavy lift, like listening on a port, accepting new connections, cleaning up when a connection is finished etc.. What we do is to provide HTTP response content, that's the beauty of Template method.

#### Singleton
In Ruby, Singleton is as simple as:
```ruby
module SingletonClass
  class << self
	# methods
  end
end
```
or you can leverage the built-in standard library:
```ruby
require 'singleton'

class SingletonLogger < SimpleLogger
  include Singleton
  # Logger implementation details.
end
```

There're still some patterns that I didn't cover, but perhaps you have got the idea that Ruby has many design patterns built into it in the first place. As a saying that "design patterns are usually a flaw in the language/framework", although I doubted that argue, apparently Ruby is not in that place.

