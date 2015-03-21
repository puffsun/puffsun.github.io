---
layout: post
title: "Bindings In Ruby"
date: 2014-01-24 14:43:40 +0800
comments: true
categories: ["Ruby"]

keywords: "Ruby, Binding, Blocks, eRuby, erb, template, debugging"
description: "How to use Binding object and Kernel#binding in Ruby"
---

### Binding overview
In Ruby, bindings can be made available by calling binding method, which is a Kernel method that returns Binding object. Let's see the [rdoc](http://www.ruby-doc.org/core-2.1.0/Binding.html) of Binding object:

>Objects of class Binding encapsulate the execution context at some particular place in the code and retain this context for future use. The variables, methods, value of self, and possibly an iterator block that can be accessed in this context are all retained. Binding objects can be created using Kernel#binding, and are made available to the callback of Kernel#set_trace_func. These binding objects can be passed as the second argument of the Kernel#eval method, establishing an environment for the evaluation.
<!--more-->
Like rdoc said, with bindings, mostly you can do is pass it as a second parameter to Kernel#eval, Ruby will use the binding to resolve open variable references by evaluating a string, other than that, bindings are pretty opaque.

```ruby
class Demo
  def initialize(n)
    @secret = n
  end
  def get_binding
    return binding()
  end
end

k1 = Demo.new(99)
b1 = k1.get_binding
k2 = Demo.new(-3)
b2 = k2.get_binding

eval("@secret", b1)   #=> 99
eval("@secret", b2)   #=> -3
eval("@secret")       #=> nil
eval("a = 101", b1)   #Bind 'a' to 101
```

### Binding in practise
#### Local Scope Bindings
With Binding object along with Kernel#binding method, we can capture the variable bindings available at a particular point in the execution context, then pass the binding to another place for further reference, let's see an example:
```ruby
def value_of(bind)
    eval "b", bind
  end

  def my_scope
    b = 123
    value_of(binding)
  end

  my_scope               # => 33
```
Bindings can be returned from a function as well. This example shows that a binding continues to exist after the function that defines the binding has exited[^1].

```ruby
def func
  a = 22
  b = 33
  binding
end

func_vars = func()

eval "a", func_vars         # => 22
eval "b", func_vars         # => 33
eval "a = 101", func_vars
eval "a", func_vars         # => 101
```
The bizarre feature of this example shows that not only does binding persist beyond the scope of the function that created them, but that you can modify these bindings just by evaluating an assignment within their context.

#### Blocks and Bindings

A block in Ruby is a chunk of code that can be called like a function. The block automatically carries with it the bindings from the code location where it was created. For example:
```ruby
  a = 33
  block = lambda { a }
  def redefine_a(b)
    a = 44
    b.call
  end
  redefine_a(block)      # => 33
```
The block returns the value of a in the binding where the block was defined (a == 33), not the binding where the block was called (a == 44).

This combination of code block and binding is called a closure, and is a very powerful tool in a Ruby programmers toolbox.

#### [ERuby](http://en.wikipedia.org/wiki/ERuby) and Bindings
ERuby stands for Embened Ruby, it's a templating system that used to embed Ruby code into text document. One of it's implementation in Ruby is called erb. With erb, you can write your template like this:
```ruby
template = "<html><body><%= @title %></body></html>"
```
Later, you can evaluate the template with Ruby code:
```ruby
@title = "The web and all that jazz"
erb = ERB.new(template)
html = erb.result(binding)
```
Note the binding call in the last line of code snippet, it pass in current execution context, which contains the @title variable definition. If we call erg like this:
```ruby
html = erb.result
```
Ruby will assumes that the binding as TOPLEVEL_BINDING, which is not what we want obviously.

#### Bindings for everyday use
Here I'll show you how to use binding to do quick debugging, first of all, we open the Array class, add a debug method[^2]:
```ruby
class Array
  def debug binding
    each do |arg|
      puts "arg = #{eval(arg, binding).inspect}"
    end
  end
end
```

You can use this method to inspect a list of snippets of Ruby code to see what each snippet returns:
```ruby
['user','current_resource', 'user.owns?(current_resource)'].debug(binding)
```
Keep mind that each element in the array will be evaluated as Ruby code, then return the execution result of the code snippet. In this example, the current execution context was captured by calling Kernel#binding method, then these code snippet was execution in the context that binding method captured.

### Conclusion
Being able to pass around binding lead to very powerful Ruby code under certain circumstances. Someone would say that passing around execution context could violate encapsulation, but in my opinion, it's ok to break some rules once a while, at least Ruby give the power of breaking rule to you, a programmer. Not even to say one of most powerful features in Ruby that comes along with Binding: Closure.



Resources:
[^1]:[Variable Bindings in Ruby](http://onestepback.org/index.cgi/Tech/Ruby/RubyBindings.rdoc)
[^2]:http://stackoverflow.com/questions/1605774/real-world-use-of-binding-objects-in-ruby
[Introduction to Bindings in Ruby](http://webjazz.blogspot.com/2006/07/introduction-to-bindings-in-ruby.html)
