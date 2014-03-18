---
layout: post
title: "Converting large XML documents to Ruby Hashes"
date: 2014-03-18 10:00
author: Robin Neumann
comments: true
tags:
  - XML
  - Ruby
  - nokogiri
  - libxml2
  - active_support
---

A lot of web services out there are dealing with XML documents. To connect to a SOAP-based web 
service to one of our Rails Applications, we need to translate XML documents to Ruby Hashes. 
 
First of all - we already had a solution implemented. For that we parsed the XML with 
``Nokogiri`` and used a custom method to iterate over the node-structure 
Nokogiri detected. The method we used is a slightly modified version of these discussed 
[in this stackoverflow article](http://stackoverflow.com/questions/1230741/convert-a-nokogiri-document-to-a-ruby-hash).  

Encouraged to improve our code quality I dropped our custom XML-Hash-converting method 
in favor of a method provided by ``ActiveSupport`` that looks temptingly elegant at the 
first glance:
 
Once you picked up some xml in a string, say ``content``, just call ``Hash.from_xml content`` 
to get a proper Hash for further processing. And of course you're free to call it on 
``HashWithIndifferentAccess`` if you need to.  
  
Unfortunately this strategy is not stable enough to handle large amounts of xml data. 
Given plenty of xml stuff saved in ``large.xml``, we can watch this method fail with 
a ``RuntimeError``: 

{% gist 9550517 %}

As far I can trace it out, this a known problem with ``REXML``, that is used behind 
the scenes by this method. I tried to came over it by using the famous C-library ``libxml2``, 
that originally was developed for the GNOME project. Happily I noticed that
there's a wrapping gem for it called ``libxml-ruby``, to which we at ABSOLVENTA [already contributed to](https://github.com/xml4r/libxml-ruby/commit/0e96dacd14f6e430750ed58bc26a668bd5415e1f).

All I need was a tiny algorithm for applying ``libxml``'s methods. Having hacked a few 
experimental lines into my editor, I noticed that there already *is* a Ruby gem for this job. 
And it works for me using it in single Ruby script like this:

{% gist 9550623 %}

In particular it worked with my evil dark-matter-xml docs! 

A programmer's world would be so easy without all the *if's* in this world.
Here it means: The solution above works *if I use it seperated from any other gems*. 
I can't get it to work in a living Rails environment together with e.g. Nokogiri. 
For some reason, Nokogiri ships (at least internally) with its own version of **libxml2**. 
In my case this led to errors when trying to load Nokogiri and libxml-ruby together. 
I strongly believe that this can be resolved, but I haven't found the right way to 
puzzle them together yet... (Feel free to drop me a note if you know how it's possible!)

### Finally... 

The intention for all research above was getting replacing the complex 
``xml_node_to_hash``-method in favor of a better tested solution maybe 
encapsulated in a gem or module.

No need to mention that it survived all my tackles to get rid of it. 
And finally we're friends.
