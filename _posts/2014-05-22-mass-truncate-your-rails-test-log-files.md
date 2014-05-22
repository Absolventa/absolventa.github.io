---
layout: post
title: "Bulk-Truncate Your Rails Apps' test.logs"
date: 2014-05-22 14:42
author: Carsten Zimmermann
comments: true
tags:
  - rails
  - logging
  - cmdline
teaser: "Constantly running your Rails test suite can grow your test.log pretty fast. Here are two commands to keep their disk usage in check."
---

Every test run for your Rails application generates a ton of log output in `log/test.log` and it can
grow quite big over time. If you're only sporting an SSD, diskspace is precious and you may want
to truncate your test.log again. Here's how you do it for all your locally checked-out Rails apps:

Assuming you have your Rails apps in a directory called, say, `rails-projects`, cd into that directory.
`du -hc */**/test.log` shows you how much diskspace your test logfiles are eating.

You can truncate them with `find . -name test.log -exec cp -v /dev/null {} \;`. If you're on a Linux system,
using `truncate` should also work, but copying `/dev/null` works everywhere.

Your development.log-files probably won't grow as fast, but you can of course put them on a diet
in the same fashion.
