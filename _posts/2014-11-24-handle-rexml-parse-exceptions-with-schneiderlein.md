---
layout: post
title: "Handle REXML::ParseExceptions with 'Schneiderlein'"
date: 2014-11-25 12:00
author: Robin Neumann
googleplus: "https://plus.google.com/u/0/106424233859444481952"
comments: true
tags:
  - middleware
  - API design
  - XML 
  - ActionPack
  
---
In order to keep our XML API compliant with RESTful constraints, we observed dissonant behavior regarding
the correct use of the HTTP protocol semantics: Whenever a customer accidentally sends malformed XML attached 
to a formally valid (w.r.t to header data and authorization) request, it will be responded with HTTP status 
code 500 by the request catching Rails application.

From the perspective of API design this is not best practice for several reasons: First of all, 
the customer does not get any information about what went wrong and what to do next, so that's 
somehow the opposite of a solid hypermedia approach. 

Second, it's not correct behavior to reflect the *global* situation. Formally the HTTP status 
code is correct, since the regular application cyclce is broken on the server side. But initially
it was caused by problems contained in the post data - and it needs to be fixed on the client side. 
Consequently it would be way better to handle these errors more confidently and respond with a 4xx 
type status code. In particular, customer input should not be able to break the server in general.

To come over it, we started a pair session to learn more about the internals of [``ActionDispatch::XmlParamsParser``](https://github.com/rails/actionpack-xml_parser), 
which is the one we use for parsing the submitted XML data. The bottleneck for our problem is the invocation of ``Hash.from_xml``, L10 below:

{% highlight ruby %}  
# Note that this is just a snippet, original definition: 
# rails/actionpack-xml_parser/master/lib/action_dispatch/xml_params_parser.rb
def parse_formatted_parameters(env)
  …
  if mime_type == Mime::XML
    # Rails 4.1 moved #deep_munge out of the request and into ActionDispatch::Request::Utils
    munger = defined?(Request::Utils) ? Request::Utils : request
               
    data = munger.deep_munge(Hash.from_xml(request.body.read) || {})
    request.body.rewind if request.body.respond_to?(:rewind)
    data.with_indifferent_access
  else
    …
  end
  …
end
{% endhighlight %}

Passing in a string containing problematic XML as argument to ``Hash.from_xml``, e.g. missing closing tags, will raise a ``REXML::ParseException``. 

There is a nice 
[blog post by thoughtbot](http://robots.thoughtbot.com/catching-json-parse-errors-with-custom-middleware) that 
highly inspired our variant of solving the problem: A custom Rack middleware that is invoked before ``ActionDispatch::XmlParamsParser``. 

In contrast to the solution presented in blog post mentioned we don't want to respond directly on middleware layer. Instead we save
the excetion information in an additional environment variable, which we can be read on the controller layer afterwards
to a proper response:

{% highlight ruby %}  
class FlyCatcher < Struct.new(:app)
  def call(env)
    begin
      app.call(env)
    rescue ActionDispatch::ParamsParser::ParseError => e
      env['rack.schneiderlein.parse_errors'] = Array(e.message)
      app.call(remove_errors_from(env))
    end
  end

  private

  def remove_errors_from(env)
    env['rack.input']     = StringIO.new
    env['rack.errors']    = StringIO.new
    env['RAW_POST_DATA']  = ''
    env['CONTENT_LENGTH'] = '0'
    env
  end
end
{% endhighlight %}

Note that the middleware needs to be invoked before params parsing, e.g.:

{% highlight ruby %}  
Rails.application.configure do |config|
  config.middleware.insert_before 'ActionDispatch::ParamsParser', 'FlyCatcher'
end
{% endhighlight %}


Putting these ingredients together was the birth of our gem [Schneiderlein](https://github.com/Absolventa/schneiderlein). Inspired 
by the fairytale »Das Tapfere Schneiderlein« (»The Valiant Little Tailor«) by the Grimm Brothers, our little tailor catches tiny errors. Since the gem structure 
is engine-like the custom middleware is integrated automatically by loading the gem. Occurring parse errors can be handled
in the responsible controller then:

{% highlight ruby %}  
class ApiController < ApplicationController
  before_action :handle_parse_errors
   
  respond_to :json, :xml
      
  protected
         
  def handle_parse_errors
    schneiderlein = Schneiderlein::Catch.new(request)
    respond_with schneiderlein.errors, status: 422 if schneiderlein.errors.any?
  end
end
{% endhighlight %}
