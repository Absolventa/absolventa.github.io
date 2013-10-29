---
layout: post
title: "Watching A Certificate Revocation List Using Event Girl"
date: 2013-10-28 15:34
author: Carsten Zimmermann
comments: true
tags: 
  - event-girl
  - openssl
  - devops
  - wifi
---

ABSOLVENTA uses WPA2-Enterprise to control acccess to the company's wifi
network.  Authentication is done using x509 certificates issued by our local,
self-signed Certificate Authority (CA). We use a FreeRadius server on a
[FreeBSD](https://www.freebsd.org) machine to check for the validity of the
presented certificate.

I wrote a simple <abbr title="Command Line Interface">CLI</abbr> to faciliate
the certificate management and to automatically configure the Radius server,
including certificate revocation. The next-update field of the generated
[certificate revocation
list](http://en.wikipedia.org/wiki/Certificate_revocation_list) (CRL) file is
set to +100 days whenever a certificate is revoked, assuming there will always
be at least an intern who leaves the company within three months' time.

Yesterday morning, I was denied access to our wifi network. Working from an ethernet-less
Macbook Air I had to wire myself somewhere to debug the problem. The Radius server
was alive and kicking but looking at the logs it was evident that it couldn't
verify certificates anymore as its CRL was no longer valid.

Regenerating a new CRL was easy enough, but it was clear that my initial
assumption of »100 days is a safe enough bet« was false. I didn't want to
increase the lifetime of the CRL as an overly old CRL kind of defeats its
purpose.  I wrote a simple Ruby CRL parser named
[crl_watchdog](https://github.com/Absolventa/crl_watchdog) and a shell script
as an early warning system:

{% gist 7196906 %}

The file resides in ``/etc/periodic/daily`` which is FreeBSD's equivalent of
``/etc/cron.daily`` on most Linux distributions.

You'll notice the various [Event Girl](https://github.com/Absolventa/event_girl)
statements: The shell script makes use of the
[event_girl_client](https://github.com/Absolventa/event_girl_client) CLI. Event Girl
is the outcome of the [Rails Girls Summer of Code](http://railsgirlssummerofcode.org)
team [Highway to Rails](http://highwaytorails.tumblr.com) we've been hosting
from July 1st to September 30th and it's awesome.

The Event Girl app has two event expectations: one »backward« expectation that
will inform us when the daily script doesn't run for whatever reason and a
»forward« one which will send an email instantly when the CRL will expire
within the next 14 days (as notified in line 22).

Using Event Girl, I can not only make sure not to run into an outdated CRL again,
but also feel at ease that the CRL actually remains being monitored. Also, I
don't have to bother about sending mails from the server and can leave that
to the Event Girl app.
