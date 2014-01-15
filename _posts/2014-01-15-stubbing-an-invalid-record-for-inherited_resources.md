---
layout: post
title: "Stubbing an invalid record for inherited_resources"
date: 2014-01-15 17:58
author: Carsten Zimmermann
comments: true
tags:
  - ruby
  - rspec
  - mocking
  - stubbing
  - inherited_resources
---

Saving a record in a controller has at least two outcomes\: it can succeed or it
can fail and you want to test this branching.

In order to decouple a functional test from the concrete attributes or other
states that define its validity, it's good to skip the actual validation
and stub it out instead.

Stubbing a failing record for a create or update request is fairly easy for
a vanilla controller:

{% highlight ruby %}
describe MyController do
  describe 'POST create' do
    it 'creates a record' do
      allow_any_instance_of(MyModel).to receive(:valid?).and_return(true)
      expect { post :create }.to change { MyModel.count }
    end

    it 'fails to create a record' do
      allow_any_instance_of(MyModel).to receive(:valid?).and_return(false)
      expect { post :create }.not_to change { MyModel.count }
    end
  end
end
{% endhighlight %}

However, it isn't that easy with [inherited_recources](https://github.com/josevalim/inherited_resources).
(or more accurately with [responders](https://github.com/plataformatec/responders) which is used
behind the scenes). Inherited Resources considers a record invalid when it has ``errors`` (see also
[this issue](https://github.com/josevalim/inherited_resources/issues/38)).

Here is an approach to properly stub record validity for use with Inherited Resources:

{% highlight  ruby %}

# spec/support/advanced_validity_stubbing.rb
module AdvancedValidityStubbing
  def stub_validity(klass, validity)
    allow_any_instance_of(klass).to receive(:valid?).and_return(validity)
    unless validity
      errors = klass.new.errors
      errors.add(:base, 'Stubbed to be bad')
      allow_any_instance_of(klass).to receive(:errors).and_return(errors)
    end
  end
end

# spec/spec_helper.rb
RSpec.configure do |config|
  # ...
  config.include AdvancedValidityStubbing
end

{% endhighlight %}

The modified controller spec may look like this:

{% highlight ruby %}
describe MyController do
  describe 'POST create' do
    it 'creates a record' do
      stub_validity MyModel, true
      expect { post :create }.to change { MyModel.count }
    end

    it 'fails to create a record' do
      stub_validity MyModel, false
      expect { post :create }.not_to change { MyModel.count }
    end
  end
end
{% endhighlight %}

