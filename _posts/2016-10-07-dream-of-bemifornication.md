---
layout: post
title: "Dream of BEMifornication – or how we refactored our scss with linting"
date: 2016-10-07
author: axlwaii
comments: true
tags:
  - CSS
  - SCSS
  - refactoring
teaser: "
Writing code – any kind of code – means you'll eventually have to deal with
refactoring it. CSS/SCSS is no exception. In this article, the author jumps
into the mud and illustrates how we refactored an old SCSS codebase using
lint while automating tedious refactoring tasks.
"
---


Writing a lot of code over the years also means to deal with refactoring at some point. Your skills improve over time and technologies evolve. Coming back to an old project often makes it hard to understand what the heck Younger You was thinking when writing and structuring this code.

For example when I started learning SASS, I felt in love with nesting everything. Back then I tought nesting elements would be the best way to avoid css conflicts … I figured out that might just be true up to a certain level of nesting. When [BEM](http://getbem.com/) came around, we started sticking to its convention, leaving previously written css code mostly untouched. Things started to get messy in our frontend codebase.

Lately my fellow co-worker Alex began to refactor our oldest project and we started talking about [scss-lint](https://github.com/brigade/scss-lint/). I was already using [jslint](http://jslint.com/) in my editor but never really thought about linting my scss as well. We sat together and before long, we agreed that it is about time to start using it.

## So what is scsslint?

Scss-lint is a tool to analyse your scss code for potential errors and to make sure the code conventions are respected.

In order to use [scss_lint](https://github.com/brigade/scss-lint) in your rails application you need to include it into your `Gemfile`.

{% highlight ruby %}
# Gemfile
group :development, :test do
  gem 'scss_lint', require: false
end
{% endhighlight %}

Afterwards you can configure the linting rules by adding `.scss_lint.yml` to your applications root directory. You can find a list of all supported linting options [here](https://github.com/brigade/scss-lint/blob/master/lib/scss_lint/linter/README.md).

{% highlight yml %}
scss_files: 'app/assets/stylesheets/**/*.css.scss'

exclude: 'app/assets/stylesheets/vendor/**'

linters:
  BorderZero:
    enabled: false

  Indentation:
    severity: warning
    width: 2

  SelectorFormat:
    convention: hyphenated_BEM
{% endhighlight %}

## A linter walks into a project

![](https://3.bp.blogspot.com/-mB2Cx3d05u4/UOoAHp1BU0I/AAAAAAAAFBc/SWdJcAmH7Vk/w1200-h630-p-nu/troy-barnes.gif)

Running `$ scss-lint` in your terminal, will give you a good overview which files to tackle. If you want to see different information, for example the files which have no warnings, it is worth to checkout the [formaters](https://github.com/brigade/scss-lint#formatters).

To use linting within your editor you need to install a [plugin](https://github.com/brigade/scss-lint#editor-integration). I would suggest mapping linting to a key instead of automaticaly linting when opening or saving a file. If you still want to enable "autolinting", keep in mind that it will slow down your text editor's speed.

## Jumping into the mud

I highly recommend **automating most of the linting** tasks: it saves a lot of time that we can spend focussing on bemifaction.

Introducing [CSScomb](http://csscomb.com/). This is a fantastic tool for automatic linting. It formats your code very efficiently and it's easy to customize. You can create your configuration by using the config generator on the project's website
or download some configs [here](https://github.com/csscomb/csscomb.js/tree/dev/config).
We used `csscomb.json` and tweaked the sort-order to match [Concentric-CSS order](https://github.com/brandon-rhodes/Concentric-CSS).

{% gist e73bed248df2d3af1e489c74d8bbe9be %}

I installed [vim-csscomb](https://github.com/csscomb/vim-csscomb) to use csscomb within my editor but there are plugins for all major editors arround.

Let's start digging!

Alex came up with a simple structure for our scss files:

{% highlight css %}
// BLOCK
.example {
    color: #fff;
}

// MODIFIER
.example--red {
    color: #ff0;
}

// ELEMENT
.example__title {
    font-size: 1.5rem;
}
{% endhighlight %}

Dividing our modules in BEM-blocks ensures the usage of only one block element per file.
Bemify your classes, refactor your views, run CSScomb.

![](https://images.duckduckgo.com/iu/?u=http%3A%2F%2Fmedia.riffsy.com%2Fimages%2F3ccc0e15cbf9bee22c30701649065643%2Ftenor.gif&f=1)

Voilà.

## Conclusion

It can be a long road and in the beginning it seems like a hole with no bottom. But seeing the warning and error messages disappear can be very rewarding. Is it fun? No. Do you end up with cleaner code, better maintainablity and less conflicts in your stylesheets? Yes.
