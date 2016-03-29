---
layout: post
title: "Adhoc Exports on Heroku's Console"
teaser: "More often than not, exporting data could be so easy: a simple Ruby one-liner hacked into the production console and then copying its output. It's easy with redirecting the output of the Heroku console into a local export file."
date: 2016-03-29 11:54
author: carp
comments: true
tags:
  - rails
  - heroku
---

Ever had that _»can you give me a list of … real quick?«_ request coming in?
Most of the time, delivering that list only involves firing a simple Arel query
on the `rails console`, `Array#join` the record's relevant attributes with a
mere `;` and hand it over as a CSV file.

The annoying part is capturing the output: For a large resultset, it can very
well exceed your terminal's output buffer size. Even if it does fits,
copy/pasting everything with your mouse is tedious at best.

Writing to a CSV file on Heroku's Rails console requires uploading it
somewhere, S3 in our case, meaning you'll have to deal with authentication keys
and APIs that I don't know by heart. Also not my idea of quickly running a
simple export.

Creating a rake task that handles everything is certainly the proper way to go,
but it means doing a `git commit`, waiting for the stuff to deploy and potentially
`git revert` the exporter again if it's just throw-away code.

I want to stick with a simple Ruby one-liner and this is how I use the `rails
runner` command, invoked through the Heroku console, together with output
redirection to create my CSV file freshly from production data:

```shell
heroku run rails runner \
  "puts MyModel.where(foo: 'bar').pluck(:id, :email).map { |data| data.join(';') }" \
  -r production > my_adhoc_export.csv
```

If you want to see what's going on while your export does its thing, use `tee(1)`:

```shell
heroku run rails runner \
  "puts 'They are going to take me away haha!'"
  -r production |tee funny_farm.txt
```

Now, a bit of an advise: running code on your production system is not a Good Idea™.
If you have to run anything more complex than a simple query-command, wrap it into
a rake task (or some such). The above example is meant for one-off tasks that output
stuff to my local terminal. You have been warned :)
