---
layout: default
title: What is ABSOLVENTA?
description: "Jobs for students, graduates and young professionals: What is ABSOLVENTA?"
---

# {{ page.title }}

## The Platform

ABSOLVENTA is a job platform for students, graduates and young professionals in
Germany. We actively support students with their career preparations, get
graduates their first regular job and help young professionals work their way
up the corporate ladder.

ABSOLVENTA allows users to browse job offers traditionally. They may also
upload their CV and can be contacted passively by recruiting companies who
browse our database. Never fear: all data will stay anonymized unless a user
explicitely grants permission to see his or her name and contact info.

ABSOLVENTA provides companies with tailored recruiting-tools for the targeted
audience of young academics and conjointly deploys efficient employer-branding
concepts. Founded in 2008, ABSOLVENTA GmbH currently employs about 40 people in
the heart of Berlin.

## Technology

ABSOLVENTA runs on a pretty standard Ruby on Rails stack using a PostgreSQL
database backend. This is the place where the development team talks about it.

Please find a selection of recent articles below:

<div id="related">
  <ul class="posts">
    {% for post in site.posts limit:3 %}
    <li>
      <span>{{ post.date | date_to_string }} &raquo;</span> <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
    {% endfor %}
  </ul>
</div>
