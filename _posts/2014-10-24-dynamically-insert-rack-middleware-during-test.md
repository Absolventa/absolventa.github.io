---
layout: post
title: 'Insert Rack Middleware during Tests'
date: 2014-10-24 16:23
author: carp
comments: true
teaser: "Test-driving the behaviour of a Rails Engine with and without the host app including a certain middleware is tricky. Two simple hacks in your rspec request spec make it happen."
tags:
tags:
  - ruby
  - rack
  - middleware
  - test
---

We recently deployed a wee internal Rails Engine that logs raw API data
in order to save the original payload that was pushed over the wire. That
little piece of software, however, had a bug (which is _pretty_ uncommon for
software, right?): The logged payload was always empty.

The Engine's host application uses Rails' `ActionDispatch::XmlParamsParser`
middleware. It would seem that it messed up the contents of `rack.input`
which in turn holds the POST data.

`rack.input` is an IO-like object and it needs to be rewound after
it has been read from. XmlParamsParser reads from it, but apparently doesn't
rewind it. It seams easy enough to fix, but how to testdrive it?

I didn't want to statically include a rack.input-modifying middleware into the
Engine's dummy app. Instead I wanted add another context that tested against a
version of the dummy infected with a mock middleware and have all other tests
run without it.

Here's how I did it. Enjoy!

{% highlight ruby %}
# spec/requests/rack_input_modifiable_spec.rb
describe 'Tracks data with rack.input modifying middleware in place' do

  # Need to bind this to a constant in order to hook it in later.
  # It does nothing except putting the IO read marker to the end.
  ReadsFromRackInput = Struct.new(:app) do
    def call(env)
      env['rack.input'].read # moves to EOF
      app.call(env)
    end
  end

  # Dummy::Application is already loaded at this stage. We can't modify
  # the middleware stack as it's frozen, so subclass it and infect:
  def app_with_middleware
    @app_with_middleware ||= Class.new(Rails.application.class) do
      config.middleware.insert_after \
        ActionDispatch::ParamsParser, ReadsFromRackInput
    end
  end

  # `app´ is what RSpec tests against in request specs. Think `controller´
  # for controller specs. Overwrite it with our infected app.
  def app; @app ||= app_with_middleware end

  # Tweaks to get the subclassed app to get all of its parent's behaviour.
  before do
    Dir.chdir(Rails.root)
    allow(app).to receive(:routes).and_return Dummy::Application.routes
  end

  it_behaves_like 'tracks raw data' # i.e. the actual tests
end
{% endhighlight %}
