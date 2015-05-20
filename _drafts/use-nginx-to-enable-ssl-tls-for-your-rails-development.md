---
layout: post
title: Instant Local Nginx SSL/TLS Proxy
date: 2015-05-20 17:00
author: carp
teaser: Enabling SSL/TLS for your Rails development environment can be a pain. I reduced it by automating the setup of a local Nginx SSL-reverse proxy. Works with Mac and Homebrew.
comment: true
tags:
  - TLS
  - SSL
  - webrick
  - rails
  - development
  - homebrew
  - mac
---

Ideally, your Rails app just has `config.force_ssl = true` configured for its
production environment but when you have TLS- and non-TLS contexts in
your app, things become tricky to test on your local development machine.

For [Dev/Prod Parity](http://12factor.net/dev-prod-parity), I want to have the
same SSL setup locally as on staging/production. Documentation for setting up
a local Nginx instance [is readily available](http://www.cyberciti.biz/faq/howto-linux-unix-setup-nginx-ssl-proxy/),
but it's verbose and involves too many manual steps.

![Automate all the things!](/images/2015-05-20-nginx-ssl-automate-all-the-things.jpg)

I've written a shell script to automate the happy path of the Nginx setup,
namely installing Nginx, creating a self-signed SSL-certificate, writing the
reverse-proxy config directive and optionally starting the Nginx webserver
right away. The latter requires `sudo` superpowers since binding to the SSL
default port 443 requires root privileges. Furthermore, it makes a few
assumptions:

1. You're using Mac and Homebrew
2. You don't have Nginx installed or don't care about config files being overwritten
3. Your Rails app is running on localhost:3000

You can download the script from [Absolventa's Github repo](https://raw.githubusercontent.com/Absolventa/dotfiles/master/nginx-ssl-setup.sh).
Run it on your machine with `bash nginx-ssl-setup.sh` and follow the instructions on the screen.
If you stray from the Happy Path™, feel free to send a PR our way!


