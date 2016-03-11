---
layout: post
title: "Including a Module. With Parameters!"
teaser: "The Ruby include statement only allows for one type of argument: the module constant(s) to include. I've found myself in a situation where I wished I could make the include statement more dynamic, like passing extra arguments that influence the module to be included. This article describes how to use a module factory to achieve exactly that."
date: 2016-03-11 16:34
author: carp
comments: false
tags:
  - ruby
  - meta-programming
---

The Ruby `include` statement only allows for one type of argument: the module
constant(s) to include. Oftentimes I wish it would be possible to pass
arguments that the included module receives as optional parameters to its
`included` callback like so:

{% highlight ruby %}
# Note: This would be nice … but this code doesn't compile
module MyMixin
  def self.included(base, *here_be_more_args)
    # Have access to the additional args passed
    # in the include statement below
  end
end

class MyClass
  include MyMixin, :param1, :param2 # ...
end

{% endhighlight %}

[Module#included](http://ruby-doc.org/core-2.3.0/Module.html#method-i-included)
is the method that gets called when the module is included somewhere. Unlike in
my would-be-nice example, it only receives one argument: the constant of the
including class.

Additional arguments to `include` are expected to be modules, allowing you to
include more than one module on one line.

A way to parameterize the behaviour of included modules is to add a **singleton**
method in the included class and then have that class method accept parameters to
do whatever (e.g. set class-wide variables or dynamically define methods).

{% highlight ruby %}
module MyMixin
  def self.included(base)
    base.define_singleton_method :define_greeter do |method_name, output|
      # A dynamically defined method that dynamically defines another
      # method. What a crazy world!
      define_method(method_name) { output }
    end
  end
end

class MyClass1
  include MyMixin
  define_greeter :morning, 'Good morning, sir/madam'
end

class MyClass2
  include MyMixin
  define_greeter :afternoon, 'Good Afternoon! Would you like some tea?'
end

MyClass1.new.morning   # => 'Good morning, sir/madam!'
MyClass2.new.afternoon # => 'Good Afternoon! Would you like some tea?'
{% endhighlight %}

This is obviously a contrived example, but what I dislike about it is the two
lines of meta programming that serve the same purpose: dynamically adding
something to the including class. Also, the `define_greeter` class method is
probably not needed anywhere else in the class and this is against my sense of
aesthetics.

Now, how about **dynamically creating a module** and than including that one?
Like … a module factory? How do we even start? It's actually quite easy: the
`include` statement only accepts arguments of type `Module`. So all we have to
make sure that whatever we pass to `include` returns a proper Ruby `Module`.
I give you this:

{% highlight ruby %}
class ModuleFactory # Yes, it's a class
  def self.new(greeter_name, greeter_output) # overriding the constructor
    Module.new do
      define_method(greeter_name) { greeter_output }
    end
    # Important: We don't return an instance of ModuleFactory here. The
    # (implicit) return value is the anonymous module.
  end
end

class MyClass
  include ModuleFactory.new(:evening, 'Good evening!')
end

MyClass.new.evening # => 'Good evening!'
{% endhighlight %}

Voila! `ModuleFactory.new(…)` returns a module which can be included, thus
combining the include statement with the dynamic meta-programming. The provided
examples are all a bit abstract but I will provide a more concrete (Rails 5!)
example in a follow-up post.
