---
layout: post
title: "Exploring the Levenshtein algorithm with Ruby"
date: 2015-11-24 09:00
author: rbn
teaser: After a short introduction to the Levenshtein algorithm that determines the similarity of two strings, we'll walk through a concrete example and discuss an exemplary implementation in Ruby. Meant as an encouragement for exploring theoretical algorithms with a language of your choice!
comments: true
tags:
  - ruby
  - levenshtein distance
  - algorithms 
---

Today I'd like to introduce you to an algorithm I stumbled over already tiwce
and that I really like because of its simple idea to address the non-trivial
problem to *quantify* the similarity of two strings: The Levenshtein algorithm.

The algorithm determines the so-called Levenshtein distance of two strings
which is a natural number. It's an example of the mathematical technique of 
*[Dynamic Programming](https://en.wikipedia.org/wiki/Dynamic_programming)* 
and was invented by the Russian mathematician [Vladimir Levenshtein](http://www.keldysh.ru/departments/dpt_10/lev.html) already 
in 1965. The algorithm is used by some major companies (e.g. Yahoo!) in 
production environments until today.

### How it works

The core idea is to measure similarity by the minimal number of *elementary transformations*
that are needed to turn an arbitrary string *S* into another string, say *T*. Here, 
*elementary transformations* are defined as precisely three operations:

* a simple insertion of a single character to the string
* a deletion of a single character of the string
* a substitution of a single character by another
 
Mathematically it works with a very elegant trick of computing
the Levenshtein distances of trivial substrings first (which is very easy) 
and then successively computing distances of more complex substrings by
only using previously evaluated values. 

The mathematical formalism is well-explained at [wikipedia](https://en.wikipedia.org/wiki/Levenshtein_distance) and other resources
and I'd like to focus on a walk-through example now. Let's compute the Levenshtein distance of the words *cat* and *cute*.

Note that the empty word is denoted by *ε*. Practically the algorithm is filling a table (a matrix) where the (i,j)-th entry will 
contain the Levenshtein distance¹ of the prefix of length *i* of the first word and the prefix of the prefix of length *j* of the 
second word. 

We emphasize the (technical) convention that every word starts with the empty word, so *cat* can be written as *εcat*. 
Therefore the first entry with indices (0,0) is the Levenshtein distance of the *0*-prefix of *εcat* and *εcute*, which is the
distance of the empty word *ε* with itself - which is simply zero. Furthermore, to construct any word of length *i* 
starting with the empty word we simply need *i* insertions. With this in mind it is 
easy to write down the first row immediately: 

{% highlight ruby %}
 _| ε c a t
 ε 0 1 2 3
{% endhighlight %}

This translates as: It takes one elementary transformation to turn *ε* into *εc* (an insertion), two transformations
to turn *ε* into *εca* and so on. The first column can be obtained immediately with the same strategy:

{% highlight ruby %}
 _| ε
 ε 0 
 c 1 
 u 2
 t 3
 e 4 → four insertions are needed to turn ε into "cute"
{% endhighlight %}

Let's combine them. Our recurrence matrix *D* has now the form:

{% highlight ruby %}
 _| ε c a t
 ε 0 1 2 3
 c 1 ? ? ?
 u 2 ? ? ?
 t 3 ? ? ?
 e 4 ? ? ?
{% endhighlight %}

All we have to do is to fill the question mark entries by using only the previously computed values. This
is the point were we need to inspect the mathematical rules the algorithm postulates. Suppose we want to obtain the matrix entry (i, j) and
remember that this corresponds to the prefix of length *i* of the word *εcat* and prefix of length *j* of the word *εcute*:

When the *i*-ith character of the first word *S* exactly matches the *j*-th character of the second word *T* we have
the optimal case. No "operation" is required, the global costs are the same as for the distance of the (i-1)-th and the (j-1)-th 
distances of the words, that is in that case. So we take the matrix entry *(i-1, j-1)* and put it into *(i, j)* as well.
Otherwise the characters do not match. We have execute one of the elementary operations. But which one? Here's the answer:

$$ D_{i, j} = \min \big [ \text{insertion costs}, \text{deletion costs}, \text{substitution costs} \big ] + 1 $$

The `insertion costs`, `deletion costs` and `substitution costs` are nothing else
than values of neighbor matrix entries we already have computed! So have a look to the neighbors, 
choose the one with mininal costs and add *+1* to its costs, because we need exactly one more operation
compared to that "minimal" neighbor. Notice that here we have chosen some kind of optimal *path*.

Let's see how it works for our example matrix above. For the entry (1,1) we have the optimal case: 
A *c* is added for both sides (when starting with *ε*) and since the same character is added on both sides 
the Levenshtein distance stays zero because it was for the upper-left neighbor. And it matches the 
intuition if you recapitulate that entry (1,1) means the distance between *εc* and *εc*, which still
is the same string. 

{% highlight ruby %}
 _| ε c a t
 ε 0 1 2 3
 c 1 0 ? ?
 u 2 ? ? ?
 t 3 ? ? ?
 e 4 ? ? ?
{% endhighlight %}

Now the situation gets more interesting. Let's focus on the second row, where two question
marks are left. The more left question mark is the entry *(2, 1)* and corresponds to the
distance of *εca* and *εc*. The algorithm now postulates looking for the minimal distance
relative to previously computed substrings. The already-computed neighbors have values *2*,
*1* and *0*. The mininal one is *0*. We now take the minimal value (0) and add +1 to it to obtain
the value for *(2, 1)*. This procedure is repeated successively for each question mark left. If we
consequently do this we obtain:

{% highlight ruby %}
 _| ε c a t
 ε 0 1 2 3
 c 1 0 1 2
 u 2 1 1 2
 t 3 2 2 1 
 e 4 3 3 2 
{% endhighlight %}

… and finally the matrix entry (m, n) where *m* is the length of the first string and *n* is the length of the second string
is defined as the Levenshtein distance of the two strings. So we have determinded the Levenshtein distance of *cat* and 
*cute*: *2*.

### Implementation

When I reasoned if the algorithm would be good choice for our specific problem I immediately
started hacking together some naive lines in my editor. I knew there were several stable implementations 
out there and my solution most probably wouldn't add any new feature nor would have significant performance improvements,
but I wanted to go through the algorithm on my own step by step. So basically I started reimplenting
it. But why reinvent the wheel? 

As developers we're used to the common folklore law of *You-simply-should-not-reinvent-the-wheel*. This is 
absolutely valid when it comes to the question what to use in production environments (think of
cryptography!). But if you're interested in extending your toolset of concepts the answer is different: Rebuilding the 
algorithm on your own - independent of its difficulty - will make you learn something more than *Copy&Paste*. It's the
same phenomenon that transcribing something from the blackboard in school by yourself will pay off way more than simply photocopying your neighbor's notes 😉 .

By the way, there is no specific advantage in choosing Ruby here. You're fine to use any turing complete programming 
language that you like and do the same. Of course there already are plenty of Ruby variants of
the Levenshtein algorithm, for example I found a very minimalistic implementation [here](http://rosettacode.org/wiki/Levenshtein_distance#Ruby):

{% highlight ruby %}
module Levenshtein
  def self.distance(a, b)
    a, b = a.downcase, b.downcase
    costs = Array(0..b.length) # i == 0
    (1..a.length).each do |i|
      costs[0], nw = i, i - 1  # j == 0; nw is lev(i-1, j)
      (1..b.length).each do |j|
        costs[j], nw = [costs[j] + 1, costs[j-1] + 1, a[i-1] == b[j-1] ? nw : nw + 1].min, costs[j]
      end
    end
    costs[b.length]
  end
end
{% endhighlight %}

Its pretty minified² but therefore hard to read when you want to learn how to
apply the algorithm by youself. I felt the drive for giving it more structure (and therefore
bloating it intentionally). The first object I focused on was
the so-called *recurrence matrix*, usually called *D* symbolized by the `costs` Array in the
minimalistic version. This is the table we filled in the example above. For me it felt like an own container data type
(in the fashion of a monad), a somewhat supercharged Array. So I defined an own class for it: 

{% highlight ruby %}
  class RecurrenceMatrix

    attr_reader :store

    def [](index)
      store[index]
    end

    def []=(index, value)
      store[index] = value
    end

    def initialize(m, n)
      @store = Array.new(m+1) { Array.new(n+1) }

      (0..m).each { |i| store[i][0] = i }
      (0..n).each { |j| store[0][j] = j }
    end

  end
{% endhighlight %}

The crucial point here is that the `RecurrenceMatrix` automatically
fills the entries that are initially known. This container class may look
overengineered here, but note that one can explicitly say

{% highlight ruby %}
d = RecurrenceMatrix.new(m, n)
{% endhighlight %}

which I really like because it says without any comment what it is, what its purpose
is and how to use it ("Matrix" in the class name implictly gives the hint to the 
user to make use of the operator `:[]`, at least this is what I would expect). 

Then I thought about how the outer API of my fictional class `LevenshteinDistance` could look like. The Levenshtein distance
is a natural number³, so what I had in mind is

```
result = LevenshteinDistance.new('cat', 'cute').to_i
```

And the `to_i` method glues the successive computations together
and returns the last value computed. Here, `d` is a recurrence matrix
as reasoned above.

{% highlight ruby %}
  class LevenshteinDistance
    
    …

    def to_i
      (1..n).each do |j|
        (1..m).each do |i|
          d[i][j] = costs_for_step(i, j)
        end
      end

      d[m][n]
    end

  end
{% endhighlight %}

The only item missing is the computation of the costs for each matrix entry. Here's what I 
did based on the mathematical formulation:

{% highlight ruby %}
  class LevenshteinDistance
    
    …


    private

    def same_character_for_both_words_is_added?(i, j)
      s[i-1] == t[j-1]
    end

    def obtain_minimal_value_from_neighbors(i, j)
      [
        d[i-1][j], 
        d[i][j-1], 
        d[i-1][j-1],
      ].min
    end

    def costs_for_step(i, j)
      if same_character_for_both_words_is_added?(i, j)
        d[i-1][j-1]
      else
        obtain_mininal_value_from_neighbors(i, j) + 1
      end
    end

  end
{% endhighlight %}

One could take this one step more consequent and move all the computation logic inside the `RecurrenceMatrix`.
But alltogether the state above felt fine to me and made me ready for switching to use a more performant implementation relying on C instead :)

Note that my implementation works well for strings that aren't long, say below 500 characters. During my heuristically flavoured 
(and therefore non-scientific) benchmark on my local machine it took 1.28 seconds to compute the 
Levenshtein distance of strings of length 1000 and ~30 seconds to compute the Levenshtein distance of 
strings of length 10000. Theoretically the algorithm itself is of complexity *O(mn)* where *m* and *n* are the sizes 
of the input strings. So in conclusion, to compare really large strings you may want to use a Ruby gem 
that relies on native C code extensions or similar like

* [levenshtein-ffi](https://github.com/dbalatero/levenshtein-ffi)
* [damerau-levenshtein](https://github.com/GlobalNamesArchitecture/damerau-levenshtein)

### Further reading

I hope you feel encouraged now playing with algorithms or other concepts from academia that are unknown to you! 

* [source code](https://github.com/Absolventa/levenshtein_rb/blob/master/lib/levenshtein_rb/levenshtein_distance.rb) 
of the implementation discussed here
* [Levenshtein algorithm at wikipedia](https://en.wikipedia.org/wiki/Levenshtein_distance)
* [A very interesting blogpost](http://julesjacobs.github.io/2015/06/17/disqus-levenshtein-simple-and-fast.html) that takes things one step further
and includes thoughts about optimizations and the Levenshtein algorithm seen from an autmata theory perspective (including a Python implementation)
* [A presentation that](https://web.stanford.edu/class/cs124/lec/med.pdf) summarizes more related variants and optimization ideas 

¹) I implictly defined the *cost* of each operation as +1 as it is "per default". This can be modified
of course, but this is not in the scope of this write-up. Just keep in mind
that this is a feature that can be adjusted.

²) … which is totally okay because it was intended as a short concise implementation there!

³) It's completely off the topic, but an interesting philosophical question indeed: The discussion if zero is treated [as a natural number or not](http://math.stackexchange.com/questions/283/is-0-a-natural-number) 😉
