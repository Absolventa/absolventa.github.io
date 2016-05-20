---
layout: post
title: "Loading Controller-specific Assets with Sprockets v3.0"
teaser: "An upgrade-cascade lead to an »undefined method `find_asset' for nil:NilClass« on our little system to load controller-specific javascript. This article describes how we bent asset_path to our will to make it work again."
date: 2016-05-20
author: carp
comments: true
tags:
  - rails
  - assets
---

Going from Rails v4.2.4 to v4.2.6 – a mere _patchlevel_ update, mind you –
became a Yak-shaving nightmare: Upgrading two other gems that broke with v4.2.6
brought in Sprockets v3.x. And that broke our little logic to include
**controller-specific javascript**.

We have separate JS files following a naming convention that should be included
in the layout when they are present. The path is constructed as follows:


```ruby
module ApplicationHelper
  def controller_specific_js
    "modules/#{controller_name}"
  end
end
```

In the pre-update version, we included the Javascript like this:

```haml
# somewhere in our layout file:
- if Rails.application.assets.find_asset controller_specific_js
  = javascript_include_tag controller_specific_js
```

`find_asset` was gone post-update and none of the discussed solutions in issue
[rails/rails#311](https://github.com/rails/sprockets-rails/issues/311) were
working for us. `assets_manifest[filename]` was always empty and we didn't
want to pass `asset_path(filename)` without really knowing if the asset exists.

However, we noticed that _existing_ assets had the literal `"/assets/"` in
their path only when found. We used that to determine whether or not to out the
JS include statement:

```ruby
module ApplicationHelper
  def controller_specific_js
    asset_path("modules/#{controller_name}")
  end
end
```

```haml
# somewhere in our layout file:
- if controller_specific_js =~ /\/assets\//
  = javascript_include_tag controller_specific_js
```

Kinda ugly, yes, but seemed to get the job done. Only it didn't. Apparently
`asset_path` has some additional behind-the-scenes magic that, to add insult to
injury, behaves differently in development than on staging/production of
course: It requires an explicit filename extension to locate the asset.

Behold the final version:

```ruby
module ApplicationHelper
  def controller_specific_js
    "modules/#{controller_name}"
  end
end
```

```haml
# somewhere in our layout file:
- if (asset_path(controller_specific_js + '.js') =~ /\/assets\//)
  = javascript_include_tag controller_specific_js
```

YMMV, of course, but we hope it helps.
