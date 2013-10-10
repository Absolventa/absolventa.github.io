---
layout: post
title: "Rate Limiting Work On ActiveRecord::Base With .find_each In Edge Rails"
date: 2013-10-10 17:04
author: Carsten Zimmermann
comments: true
categories:
  - Enumerator
  - ActiveRecord
  - Enumerable
  - Edge Rails
---

I was working with several maintenance tasks that query external webservices
for a collection of ActiveRecord objects. In order to avoid hitting the 
webservices' rate limit, we pause every other iteration for a fraction of
a second before we continue.

The code looks something like this:

{% gist 6920157old_code.rb %}

I didn't like that the information of our rate limit guard clauses was so scattered: there
were bits before the block and others in the block. I had the urge to refactor it
into a more concise form using a Ruby block itself.

My idea was to create a method that takes the rate limit, the sleep time and an
enumeration and then yield the elements of the enum to a block.

I wrapped that method into a module and out came this:

{% gist 6920157rate_limiter.rb %}

This is how the specs look like:

{% gist 6920157rate_limiter_spec.rb %}

Unfortunately and unlike other enumerable methods, the batch finder ``ActiveRecord::Base.find_each``
does not return an Enumerator when called without a block. Not in Rails 3.2 and not in
Rails 4.0.

Luckily, it has been solved in Edge Rails (see
[840c552](https://github.com/rails/rails/commit/840c552047a660d0a66883fb9c0cb144d5e728fb))
and I quickly created a Rails 4.1.0.beta app to go for the following code:

{% gist 6920157external_service.rb %}

``ActiveRecord::Base.find_each`` now returns a proper ``Enumerator`` that can be passed
into the rate limiter. If you want to learn more about Enumerators in Ruby (or if you'd
like to refresh your memory), I can recommend this excellent [Ruby Tapas](http://devblog.avdi.org/2013/09/10/rubytapas-freebie-enumerator/) episode by [Avdi Grimm](https://twitter.com/avdi).

