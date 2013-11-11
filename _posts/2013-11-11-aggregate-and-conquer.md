---
layout: post
title: Aggregate And Conquer
date: 2013-11-11 09:00
author: Robin Neumann
comments: true
tags:
  - activerecord
  - postgresql
  - rails
  - statistics
---

Collecting stats is awesome. Analytical brains love being fed with stats. So let’s
fire these tiny queries at our database asking for favourite customer choices of recent
years and – meanwhile – grab a coffee … or write a book or a whole new app until your answer is served.

Persisting events over years is no problem for modern database systems in general,
but accessing them later could be. Of course, you can come over it anytime by
pimping up your server or database setup. But often it’s not the best solution
to scale a mid-size app artificially this way only because you want to save historical data.

In our case, we’re deeply interested in pageview statistics related to templates for
model instances of a Rails app. For that use case we created a custom Rails Engine
(»Kanyotoku«) that provides this service for some of our bigger apps.

And of course, when developing our tool, I directly ran into the problems as described above. To come over 
this, I was inspired by a 
<a target="_blank" href="http://www.railstips.org/blog/archives/2011/06/28/counters-everywhere/">blog post</a> 
by John Nunemaker, one of the creators of Gaug.es, a 
really awesome pageview tracking service. Even though most of his thoughts concern schemaless data, it
was easy to adopt and modify these principles for a PostgreSQL system.

### Be write-heavy, but read-lazy

One of his ideas was choosing appropriate resolutions, i.e. time frame resolutions for your 
data. Let’s translate this principle as follows: Do we need to know how many pageviews occurred 
in the last ten minutes exactly? Is this really relevant? 

For our situation it was much more interesting to know the count of pageviews per month or 
maybe per day. It is also interesting for us to count it per year or for 3 months,
but any smaller timeframe is nigh irrelevant for all our purposes.

For that reason, we introduced daily and monthly reports that are saved and/or updated each time a
pageview record is created. Of course this means we’re much more write-heavy than before.

{% gist 7409048pageviews.rb %}

On each page visit, a pageview gets saved and written to the database and in addtion to 
that a daily report is created or updated if it exists for that day. A monthly report is
handled the same way.

All reports save the pageviews count of the related time frame as an integer column which is
simply incremented if desired. As a result, we have done some pre-processing for 
data analysis in the backend - we have aggregated tables that fit perfectly to our range needs. In 
out backoffice data aggregation context, we can simply sum these integer values. If you’re interested in
the pageviews of the last 5 years, you can collect your data like

{% gist 7409097elephant.rb %}

Assumimg our elephant receives around 10,000 pageviews per year, it is great
that we don’t have to load 50,000 pageview records. Summing up integer values of less than 5 * 12 = 60 reports
should be an easy excercise for our database. 

## Conclusion

In out situation, being a little more write heavy has no
measurable negative impact on request times in our frontend context, but it highly improves
out backoffice data aggregation performance – q.e.d!
