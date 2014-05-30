---
layout: post
title: "A Lightweight Firewall/Blacklist as Rack-Middleware"
date: 2014-02-04 17:03
author: Carsten Zimmermann
googleplus: "https://plus.google.com/u/0/101728608052847168461"
comments: true
tags:
  - ruby
  - rack
  - middleware
  - blacklist
---

I recently fell in love with [Rack middleware](http://railscasts.com/episodes/151-rack-middleware).
Rack's simplicity of serving web requests by nothing but an `Array` with three elements
alone is charming. But using it as a bouncer to handle (and possibly already
return) all sorts of stuff before your requests even touch your Rails app
is the _real_ appeal.

We had an annoying bot hitting one of our applications today. There was no
harm done but while it was nowhere near a
[DoS](http://en.wikipedia.org/wiki/Denial-of-service_attack "Denial of Service"),
its requests did put some load on our servers. All requests came from one
source IP and it would have been easy to block it on the network layer, but as
we host our sites on Heroku, that was not an option.

Enter a quick middleware hack:


{% highlight ruby %}

class Firewall < Struct.new(:app)

  class << self
    def blacklist
      @blacklist ||= ['192.0.2.1'].freeze # Example IP, see RFC 5735
    end
  end

  def call(env)
    if self.class.blacklist.include? env['HTTP_X_FORWARDED_FOR']
      message = 'Your IP-address has been banned for security reasons.' +
                'If you feel this is a mistake, please contact ' +
                'support@absolventa.de'
      [403, {}, [message]]
    else
      app.call(env)
    end
  end
end
{% endhighlight %}

It's pretty good to test-drive as well: [take a look at the spec](https://gist.github.com/carpodaster/8807139#file-firewall_spec-rb).
There is nice article that
[illustrates how to test Rack-apps](http://taylorluk.com/post/54982679495/how-to-test-rack-middleware-with-rspec).

We have our middleware classes stowed in `app/middleware`. You can hook it in with
a simple one-liner in your `config/application.rb`:

{% highlight ruby %}
config.middleware.insert_after Rack::Runtime, 'Firewall'
{% endhighlight %}

Sure, you need a deployment for every new IP that needs to be blacklisted. But
it's fast and simple. You're heartily invited to
[improve the code](https://gist.github.com/carpodaster/8807139#file-firewall-rb)
 (send a PR our way).
