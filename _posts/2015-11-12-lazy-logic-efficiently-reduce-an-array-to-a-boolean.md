---
layout: post
title: "Lazy Logic: Efficiently Reduce An Array To A Boolean"
title: "Lazy Logic: Efficiently Reduce An Array To A Boolean"
date: 2015-11-12 15:04
author: carp
teaser: Ruby is smart when it comes to evaluating boolean expressions. Curiosity about how to best reduce an Enumerable to a boolan value led us to Enumerable#all?(&:call) (lambdas! <3) and a short story about how to exploit Ruby's way of boolean evaluation.
comments: true
tags:
  - ruby
  - lambda
  - reduce
---

Robin and I have been playing around with reducing a list using the `to_proc`'ed
version of the logical `AND` operator. It started out whether something like
`my_list.reduce(:'&&')` would make it through the Ruby interpreter
(spoiler alert: it doesn't, it returns <em>NoMethodError: undefined method '&&' for true:TrueClass</em> instead).

Since we were dealing with a list of booleans, the next best thing is:

{% highlight ruby %}
[true, false, true].reduce(:&)
=> false

[true, true, true].reduce(:&)
=> true
{% endhighlight %}

By the way, there's an alternative way of calling reduce/inject with an initial
value and a block in the form of `my_list.reduce(false) { |memo, obj| memo && obj }`
that I wasn't aware of:

{% highlight ruby %}
[true, true, true].reduce(false, :&)
=> false
{% endhighlight %}

Reducing using a logical `AND` can be done with the `Enumerable#all?`, for me, reading
a predicate method is far less cognitive load than passed Procs / Symbols.

What we wanted to know is how clever `Enumerable#all?` is in terms of lazy
boolean evaluation. Will it stop iterating when the result is already clear?

Quick recap: if you `&&` three expressions and the first two already evaluate
to false, Ruby instantly breaks the evaluation: there is no way that an already
falsy statement will return `true` when `&&`'ed. A good way to take
advantage of this smart feature is putting the cheap calculations first and the
more expensive ones on the right side of the expression.

Our `irb` test scenario looked like this:

{% highlight ruby %}
a = -> { true }
b = -> { false }
c = -> { fail 'Iteration should have stopped before :(' }

[a, b, c].all? { |ƛ| ƛ.call }
{% endhighlight %}

We didn't see the exception, meaning that `Enumerable#all?` indeed stopped after
evaluating the first two elements. Good!

The use of lambdas as elements instead of eagerly evaluated objects was for our
quick test, but it also allows taking the concept of evaluating the most
expensive expressions last (see recap above) one step further:
When an array is instantiated, all elements will be evaluated. Having lambdas/procs
as elements will defer their evaluation and we can iterate over "cheap" items
when reducing them with logical operators.

Imagine this example:

{% highlight ruby %}
cheap1 = -> { true }
cheap2 = -> { [true, false].sample }
expensive1 =  -> { sleep(10); false }
expensive2 =  -> { sleep(20); true }

[cheap1, cheap2, expensive1, expensive2].all? &:call
{% endhighlight %}

There is .5 chance that `expensive1` will never have to be `call`ed (in this case:
eliminating a 10 second wait-time) and it will _never_ run `expensive2`.
