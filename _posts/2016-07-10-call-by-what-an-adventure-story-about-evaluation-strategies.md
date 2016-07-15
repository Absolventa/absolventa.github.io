---
layout: post
title: "Call by what? An adventure story about evaluation strategies"
date: 2016-07-15
author: rbn
teaser: "Does Ruby evaluate its methods using call-by-value or call-by-reference? 
         The author felt a bit confused and tries to clean his opaque glasses
         recapitulating thoughts from stackoverflow, Wikipedia, and the good old λ-calculus."
comments: true
tags:
  - Ruby
  - C++
  - Theory of programming langugaes
  - Evaluation strategies
---

### Evaluation of methods

When I learned C++ at university some time ago, the concept of references and pointers
fascinated and confused me simultaneously. But for some reason I liked tinkering with it. 

I was remembered to that time when I was recently confronted with so-called *evaluation strategies*
in a completely different context. Talking about programming languages, an *evaluation strategy* is a 
description on what happens to the arguments that have been passed to a function when the function 
is executed. 

When it comes to evaluation strategies, C++ has a pretty clear and simple answer to a theorist's questions: 
The programmer can explictly control if a function/method should use the 
strategy *call-by-value* or *call-by-reference* for method evaluation. 
This is possible because in C++ there is an explicit seperation between
values (content of memory blocks) and references/pointers to them. 

#### What does call-by-value and call-by-reference mean?

*Call by value* and *call-by-reference* are probably the most popular evaluation strategies
used in major programming languages today.

Losely spoken, *call-by-value* means that the argument that has been passed into
a function gets copied before it is used during execution. The original expression 
is not modified or touched and the function body gets its own copy to play with.

Let's assume we own a picture of monkey. And now we meet a pirate, that loves drawing. 
The pirate wants to draw a banana onto our picture. If the pirate likes *call-by-value*, 
he'll copy the picture first, draw a banana onto his copy and hands back the copy with the banana.

On the other hand, *call-by-reference* means that the expression that is passed to
the function is not copied. It is directly used in the function body. That means, 
the pirate would directly draw his banana onto the monkey picture that we own. 
Our original picture has been modified. 

#### Eh, can you convert these thoughts to some concrete code, please?

let's assume we have written a C++ snippet as follows:

{% highlight C++ %}
#include <iostream>
#include <string>

class Picture {
  public:
    std::string content;
};

void drawBananaOntoACopyOf( Picture p ) {
  p.content = "A monkey with a banana";
};

int main() {
  Picture p;
  p.content = "A monkey";
  std::cout << p.content << "\n";

  drawBananaOntoACopyOf(p);

  std::cout << p.content << "\n";
  return 0;
}
{% endhighlight %}

The function *drawBananaOntoACopyOf* will take an instance
of an data structure called `Picture`. When we would have written it this
way, the value *p* will be *copied* to the function's
body data available during execution, when *drawBananaOntoACopy* is invoked. Running
the program will cause the following output:

{% highlight bash %}
A monkey
A monkey
{% endhighlight %}

If we would conversly write

{% highlight C++ %}
#include <iostream>
#include <string>

class Picture {
  public:
    std::string content;
};

void drawBananaDirectlyOnto( Picture& p ) {
  p.content = "A monkey with a banana";
};

int main() {
  Picture p;
  p.content = "A monkey";
  std::cout << p.content << "\n";

  drawBananaDirectlyOnto(p);

  std::cout << p.content << "\n";
  return 0;
}
{% endhighlight %}

then, when *drawBananaDirectlyOnto* is invoked, *p* will be a *reference* to an existing picture structure
somewhere in the memory. The `&`-sign indicates this to the compiler, it's a syntactic method
of C++ to control the evaluation strategy (*call-by-reference* in the latter case). 

The output would be:

{% highlight bash %}
A monkey
A monkey with a banana
{% endhighlight %}

### Hm. Ok. Seems pretty clear. Where's the joke?

The situation is a bit more interesting for other programming languages, like
Ruby. Eager readers can short circuit this reading [and head over to a very enlightening discussion
on stackoverflow](http://stackoverflow.com/questions/1872110/is-ruby-pass-by-reference-or-by-value), 
but I'd like to recap the discussion and the final result in my own words. 

First, unlike C++, Ruby has no built-in seperation between values and references. Everything
we save to variables in Ruby is a *reference* to an object. We only have access to the things 
in the memory via these references. 

But what happens if we pass these variables (holding references) into Ruby methods? What kind
of evaluation strategy does Ruby use? Let's examine.

#### Meanwhile in the laboratory

Consider the following Ruby code:

{% highlight Ruby %}
def dabble(human)
  puts "When dabbling, I dabble with the object_id #{human.object_id}"
  human = human.reverse # I am an important line. Why?
end

human = 'Richard Feynman'

puts "The input human is #{human}"
puts "It has the object_id #{human.object_id}"
puts "Let's dabble!"

dabble(human)

puts "After dabbling the scientist is named #{human}"
{% endhighlight %}

Running this snippet results in the following output: 

{% highlight bash %}
The input human is Richard Feynman
It has the object_id 70159717619940
Let's dabble!
When dabbling, I dabble with the object_id 70159717619940
After dabbling the scientist is named Richard Feynman
{% endhighlight %}

At the first hand, this may intuitively feel like Ruby is using *call-by-reference*, since
the `object_id` inside the function body is the same as before, so it looks like
there has no "copying" being performed. What we put in, is obviously *a reference* similar
to the references in the C++-examples.

#### Analysis

But this only true at the surface. Let's remember the fact mentioned above: Ruby has no
*values* in the sense of C++, Ruby *only* has (object) references. 
The only way of modifying a reference is *assignment*. Assignments either change existing references or 
create new ones. Keeping that in mind, concentrate on this line in the Ruby code example:

{% highlight Ruby %}
human = human.reverse
{% endhighlight %}

Let's assume that Ruby would have used *call-by-reference*. Then, according to our "definition" above, 
the argument `human`, which is a reference, that has been passed to the method `dabble`,
has not been copied. It has directly been passed to the method body. 

If so, the assignment `human = human.reverse` within the method body would have modified 
the (original) reference that has been created before method invocation. The expression
`human.reverse` would have created a new string (the reversed) with a new `object_id` and
the existing reference `human` would now point to the new object, the reversed string, because
it has been assigned. 

But, as proven by the output, this is not the case. The original reference points to the same 
thing and has not been changed. If Ruby would use *call-by-reference* `human` would point to the
reversed string in a new memory block afterwards - but it doesn't. 

In the [stackoverflow discussion](http://stackoverflow.com/questions/1872110/is-ruby-pass-by-reference-or-by-value)
I already mentioned are great explanation pictures - I'd like to pick up some thoughts of the answers here:

Check the variable situation when the method body of `dabble` starts:
{% highlight bash%}
var_human_outside_the_method -------→  "Richard Feynman"
                                               ↑ 
var_human_inside_the_method  -------------------
{% endhighlight %}

When running the first line of `dabble`, there are *two* references 
to the string "Richard Feynman". One original and the copy held by `dabble`.
I intentionally chose them both to have the same name, which may is confusing.  

Then, the assignment `human = human.reverse` inside the `dabble` method changes 
the situation as follows:

{% highlight bash %}
var_human_outside_the_method -------→  "Richard Feynman" 
                                                 
var_human_inside_the_method  ------------------→ "namnyeF drahciR"
{% endhighlight %}

#### A bigger picture

As we have seen, one could argue that Ruby uses *call-by-value* when applying the "canoncial"
terminology used in computer science strictly. Up to now we only discussed
two specific evaluaton strategies. But I'd like to underline that the world of 
evaluation strategies is *not* binary - it's a bit more colored than only black and white. 
There are some other approaches beside  *call-by-value* and *call-by-reference* and even the 
latter trategies can be differentiated a bit more. 

The wikipedia article on [evaluation strategies](https://en.wikipedia.org/wiki/Evaluation_strategy#Call_by_sharing)
calls the variant of Ruby is using *call-by-sharing*, which is an interesting point of view, since it is more
suitable for languages that do "wrap" all values, i.e. Ruby with its object references.

The description found in that article reads as follows:

> The semantics of call by sharing differ from call by reference in that assignments 
> to function arguments within the function aren't visible to the caller, (unlike by reference semantics), so e.g. 
> if a variable was passed, it is not possible to simulate an assignment on that variable in the caller's scope

… and that's exactly what happened in our Ruby script. The assignment `human = human.reverse` was not visible
to the "global" scope outside the method `dabble`. 

### Further (related) reading

I ran into this discussion with myself while reading some chapters of the book [Types and Programming Languages](http://www.cis.upenn.edu/~bcpierce/tapl/)
by Benjamin C. Pierce. It contains a discussion of *evaluation/reduction strategies* of the so-called [*λ-calculus*](https://www.youtube.com/watch?v=FITJMJjASUs&ab_channel=Confreaks), which serves as a theoretical
model of computation and can be seen as a great-grandfather of all today's functional programming languages. 

Playing with these concepts encouraged me recapitulating how non-theoretical languages, 
like C++ or Ruby, accomodate this and how the "practical" evaluation strategies performed by Ruby or C++ 
relate to the original definitions given in the context of formal systems like the λ-calculus.

If you want to dig deeper, more details of *call-by-value* as evaluation/reduction strategy for the λ-calculus can be found 
[in a paper by Gordon Plotkin](http://homepages.inf.ed.ac.uk/gdp/publications/cbn_cbv_lambda.pdf) from 1975 as well as, more convenient for non-computer-science-researchers, 
in Chapter 5 of [TAPL](http://www.cis.upenn.edu/~bcpierce/tapl/).

As mentioned, I found the discussion related to [this question on stackoverflow](http://stackoverflow.com/questions/1872110/is-ruby-pass-by-reference-or-by-value)
very educational, so if I left you confused about the topic, there's a good chance that you'll find illumination there. 
