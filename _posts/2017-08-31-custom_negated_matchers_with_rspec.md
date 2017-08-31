---
layout: post
title: "Custom negated matchers for RSpec"
date: 2017-08-31 10:00
author: rbn
comments: true
tags:
  - ruby
  - rspec
  - testing
teaser: "Chaining multiple expectations within a single test
case can get messy when you need to test against negation
statements. We study a simple trick offered by RSpec to
define custom negated matchers that will resolve this problem.
"
---

Keeping RSpec examples as atomic and lean as possible is a good rule of thumb.
Nevertheless, sometimes it makes sense to combine multiple expectations into
one test case - for example when the evaluation of the expression you want to test
is "expensive" in some sense.

Let us study the Ruby class `Pirate` and add some specs for it:

```ruby
class Pirate
  attr_accessor :mood

  def initialize
    @mood = 10
  end

  def insult(other_pirate)
    self.mood += 1
    other_pirate.mood -= 1
  end
end
```
When a pirate instance is insulting another pirate,
this is affecting the fighter's moods:

```ruby
guybrush, le_chuck = Pirate.new, Pirate.new

guybrush.mood # => 10
le_chuck.mood # => 10

guybrush.insult(le_chuck)

guybrush.mood # => 11
le_chuck.mood # => 9
```

Let us turn this into proper test cases with RSpec. One way of expressing this is
writing down two seperate test cases:

```ruby
RSpec.describe Pirate do
  let(:guybrush) { Pirate.new }
  let(:le_chuck) { Pirate.new }

  it { expect { guybrush.insult(le_chuck) }.to change { le_chuck.mood }.by(-1) }
  it { expect { guybrush.insult(le_chuck) }.to change { guybrush.mood }.by(1) }
end
```

But now let's assume we really want to combine this into a single example
where the expression `guybrush.insult(le_chuck)` is only evaluated once.
One way to combine the expectations is wrapping one test into another,
like the [composition of functions in mathematics](http://mathworld.wolfram.com/Composition.html):

```ruby
RSpec.describe Pirate do
  let(:guybrush) { Pirate.new }
  let(:le_chuck) { Pirate.new }

  it "affects the fighting pirate's moods" do
    expect { 
      expect { guybrush.insult(le_chuck) }.to change { le_chuck.mood }.by(-1)
    }.to change { guybrush.mood }.by(1)
  end
end
```
Unfortunately the probability is high that this strategy results in messy and
unreadable test code as soon as you combine 3 or more expectations.

Better (imho) is using the neat method `and` that RSpec provides:

```ruby
RSpec.describe Pirate do
  let(:guybrush) { Pirate.new }
  let(:le_chuck) { Pirate.new }

  it "affects the fighting pirate's moods" do
    expect { guybrush.insult le_chuck }
      .to change { le_chuck.mood }.by(-1)
      .and change { guybrush.mood }.by(1)
  end
end
```

This is really elegant, but problems arise once you want to include
tests for the fact that the corresponding expression `guybrush.insult(le_chuck)`
does _not_ affect some other object.

This is a priori not possible with the plain `and`-method
above. But again RSpec has a really nice built-in solution for
this problem: Definition of custom negated matchers! It's pretty simple:

```ruby
# This could live in your spec_helper.rb or wherever you configure RSpec
RSpec::Matchers.define_negated_matcher :not_change, :change
```
Afterwards you can happily use the operator `not_change`, the negated operator
to `change` as we have defined above:

```ruby
RSpec.describe Pirate do
  let(:guybrush) { Pirate.new }
  let(:le_chuck) { Pirate.new }
  let(:otis)     { Pirate.new }
  let(:carla)    { Pirate.new }

  it "affects the fighting pirate's moods" do
    expect { guybrush.insult le_chuck }
      .to change { le_chuck.mood }.by(-1)
      .and change { guybrush.mood }.by(1)
      .and not_change { otis.mood }
      .and not_change { carla.mood }
  end
end
```
Give credit where credit is due: I found this solution within a [StackOverflow answer](https://stackoverflow.com/a/36724913/2159942). Thanks to
the author!
