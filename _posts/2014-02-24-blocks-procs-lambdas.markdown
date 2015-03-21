---
layout: post
title: "Differences among Blocks, Procs, and Lambdas in Ruby"
date: 2014-02-24 22:17:52 +0800
comments: true
categories: Ruby
keywords: Block Proc Lambda Ruby
description: Difference among blocks, procs, and lambdas in Ruby programming language.
---


Almost every Rubyist love Ruby's Blocks, as well as Procs and Lambdas. These features allow you to pass code to a method then execute the piece of code at a later time. In this blog post, I'll talk about the differences among them.

###Blocks
A block is a piece of code passed to a method via curly braces <code>{...}</code> or <code>do...end</code>. By convention, <code>{...}</code> is used within one line of code while <code>do...end</code> for multi-line code. After passing into method, the block could be triggered later by <code>yield</code> keyword to get a return value. The <code>yield</code> keyword can accept parameters, which will be passed into the block it's calling at that time. Let's see a example:
```ruby
	def try
      yield if block_given?   
  	else  
      puts "no block"  
  end  
end  
```
<!--more-->
Here we define a method call <code>try</code>, you can detect if a block exist with the method <code>block_given?</code>, which is defined in Kernel module. We can invoke the method <code>try</code> like this:
```ruby
	try  								=> "no block"
	try {puts "Hey, we see a block"} 	=> "Hey, we see a block"
```
It's only a small demo, there're block usage in Ruby code everywhere, most programmer from other programming languages are shocked by the power of Ruby blocks, which leads to concise code.

### Procs
A proc (Note that the proc is lowercase here) is also a piece of code, which are stored in a Proc instance. We can reuse this object any times we want. We can store a piece of code into a Proc instance, assign it to a variable, then pass it around. Let me show you another example:
```ruby
	def proc_demo(proc_obj)
	  proc_obj.call
	end
```
Rather than the <code>yield</code>, we use the <code>call</code> method here to invoke the proc object passed in. Like the <code>yield</code> keyword, the <code>call</code> method can accept arguments either. We can invoke the above demo like this:
```ruby
	p = Proc.new { puts "Hey, we see a proc" }
	proc_demo(p)
```
or
```ruby
	p0 = proc { puts "Hey, we see a proc" }
	proc_demo(p0)
```
the code <code>proc {...}</code> is equivalent to <code>Proc.new {...}</code>, which produce a instance of Proc. You can check rdoc for further details about [Proc](http://www.ruby-doc.org/core-2.1.0/Proc.html)

### Lambdas
The difference between a lambda and Proc instance is small. Basically there're two differences between them.
* A lambda checks for the number of arguments it received, and will return <code>ArgumentError</code> if the argument number don't match. Let's poke them around inside [pry](http://pryrepl.org/), which is alternative of irb but can do much much more:

First, let see the behaviour of lambda:
```ruby
[16] pry(main)> l = lambda {|name| puts "Hello, #{name}"}
=> #<Proc:0x007fc6a50f4560@(pry):9 (lambda)>
[17] pry(main)> l.call("George")
Hello, George
=> nil
[18] pry(main)> l.call
ArgumentError: wrong number of arguments (0 for 1)
from (pry):9:in `block in __pry__'
[19] pry(main)> l.call("George", "Puff")
ArgumentError: wrong number of arguments (2 for 1)
from (pry):9:in `block in __pry__'
```
Then with proc:
```ruby

[21] pry(main)> p = proc {|name| puts "Hello #{name}"}
=> #<Proc:0x007fc6a4a6c5f8@(pry):14>
[22] pry(main)> p.call
Hello
=> nil
[23] pry(main)> p.call("George")
Hello George
=> nil
[24] pry(main)> p.call("George", "Puff")
Hello George
=> nil
```
The difference of processing the incoming arguments is pretty clear with above example.

[^1]Another difference is that lambda provide diminutive returns, which means when a Proc encounters a return statement during it's execution, it halts the enclosing method then return the provided value. Lambdas, on the other hand, return the value to the method, then return to the enclosing method, continue the program execution. This means procs behave more like a piece of code, while lambdas more like a method, you can take advantage of them of your own.

If you still not clear about the differences among them, I recommand you to read the excellent book [Metaprogramming Ruby](http://www.amazon.com/Metaprogramming-Ruby-Program-Like-Pros/dp/1934356476), check the book comments on Amazon.


[^1]: [Ruby on Rails Study Guide: Blocks, Procs, and Lambdas](http://code.tutsplus.com/tutorials/ruby-on-rails-study-guide-blocks-procs-and-lambdas--net-29811)
