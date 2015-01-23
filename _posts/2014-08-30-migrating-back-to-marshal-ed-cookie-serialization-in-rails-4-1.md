---
layout: post
title: "Back to Marshal'ed Cookie-Serialization"
date: 2014-08-30 12:58
author: carp
googleplus: "https://plus.google.com/u/0/101728608052847168461"
comments: true
teaser: "Upgrading to Rails v4.1, it felt like a good idea to switch to the new default cookie serialization format: JSON. Except later it didn't. Rails offers a migration path from Marshal to JSON, but not vice versa. This article offers a (somewhat hacky) solution to rollback from JSON to Marshal."
tags:
  - ruby
  - rails
  - upgrade
  - rollback
  - session
  - marshal
  - json
---

Upgrading to Rails v4.1, it felt like a good idea to switch to the new default
serialization format: JSON. Upgrading from the Marshal'ed serialization to JSON
was as simple as setting Rails' cookies serializer to `:hybrid`. Easy enough and
»better go with the new Rails default«, I thought.

We were warned about the implications.
[The Rails Upgrade Guide](http://edgeguides.rubyonrails.org/upgrading_ruby_on_rails.html#upgrading-from-rails-4-0-to-rails-4-1)
states:

> When using the :json or :hybrid serializer, you should beware that not all Ruby
> objects can be serialized as JSON. For example, Date and Time objects will be
> serialized as strings, and Hashes will have their keys stringified.


Only we had to realize the hard way that an external library was dumping high
level objects into the (cookie-based) session that couldn't easily and
transparently be deserialized again.

Unfortunately, there is no real rollback option: Setting the serializer back to
`:marshal` will get you parser errors when `Marshal.load` is fed with JSON
data.

Since our application started generating JSON-serialized session data after our
change was rolled out, I came up with the following »rolling rollback« strategy:

{% highlight ruby %}
# app/controllers/application_controller.rb

rescue_from 'TypeError' do |e|
  if e.message.starts_with? 'incompatible marshal file format'
    store = Rails.application.config.session_store.new Rails.application
    request.session = ActionDispatch::Request::Session.new(store, request.env)
    reset_session
  else
    raise e
  end
end
{% endhighlight %}

A simple `reset_session` won't do as that already tries to access the (invalid)
session object. The rescue hook will regenerate a new Marshal-serialized session
for all those who have already received JSON data, but will of course terminate
their existing sessions (login data, shopping cart … you name it).

