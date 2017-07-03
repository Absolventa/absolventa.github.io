---
layout: post
title: "Authoring a Private Gem with Gemfury"
date: 2017-07-03 13:55
author: carp
comments: false
tags:
  - ruby
  - rails
  - workflow
teaser: "RubyGems are a great way to share code, but not all code is for
the general public. This article illustrates a (semi-automatic) way and
some workflow self-discipline to streamline private Gem authoring using
the Gem hosting solution »Gemfury«.
"
---

We make an effort of extracting shared code into Gems. While
<abbr title="Free and Open Source">FOSS</abbr> code
goes to RubyGems.org of course, our business-internal logic needs a
private Gem server. We use [Gemfury](https://gemfury.com/) as our host
but you can also roll your own with [bundler/gemstash](https://github.com/bundler/gemstash)
or [Gem in a Box](https://github.com/geminabox/geminabox).
This article focuses on my workflow with Gemfury.

<small>Note: we are not affiliated with nor are we sponsored by said provider.</small>

## Gem Skeleton

In order to create all the boilerplate code and to make sure I am following the latest
conventions, I use Bundler's `gem` command:

```
bundle gem lolwat --no-coc --no-mit
```

When I `cd` into the created directory and load the generated `lolwat.gemspec`
into my editor, I can see that it already includes a setting for `allowed_push_host`.
This prevents me from accidentally pushing proprietary code to RubyGems.org.

Unfortunately, Gemfury.com doesn't support a RubyGems compliant API to actually
push my released Gem so there's no need to bother with changing the example host
to something real.

Instead, we'll add the following line to the project's `Rakefile`:

```ruby
ENV['gem_push'] = 'no'
```

We'll get back to it later.

## Gemfury Support
In order to easily push to Gemfury.com, there a few more additions.
First we have to include the gemfury Gem to our `lolwat.gemspec` file:

```ruby 
spec.add_development_dependency "gemfury"
```

The second file we need to edit is our `Rakefile` again. Add the following code
to the end of the file:

```ruby
# Rakefile
desc "Push lolwat-#{Lolwat::VERSION}.gem to Gemfury.com"
task :gemfury do
  package = "pkg/lolwat-#{Lolwat::VERSION}.gem"
  if File.exist? package
    system "fury push #{package}"
  else
    STDERR.puts "E: gem `#{package}' not found."
    exit 1
  end
end
```

Here's a set of shell commands that will automate all this:

```shell
GEMNAME=$(basename $(pwd))
tail -r <$GEMNAME.gemspec \
  |sed -E '1s/(.+)/\1\'$'\n  spec.add_development_dependency "gemfury"/' \
  |tail -r >_new.gemspec \
  && mv _new.gemspec $GEMNAME.gemspec

GEMVERSION="$(grep module lib/$GEMNAME/version.rb|sed 's/module //')::VERSION"
cat >>Rakefile <<EOF

desc "$GEMNAME-#{$GEMVERSION}.gem to Gemfury.com"
task :gemfury do
  package = "pkg/$GEMNAME-#{$GEMVERSION}.gem"
  if File.exist? package
    system "fury push #{package}"
  else
    STDERR.puts "E: gem \`#{package}' not found."
    exit 1
  end
end

desc "Gemfury"
task :release do
  Rake::Task[:gemfury].invoke
end
EOF
```

<small>
Note: This has only been tested on Mac OS which
uses BSD sed. GNU sed (Linux) may behave differently.
</small>

## Prepare Release

While I'm hacking, I track completed features in a **Changelog** section
in my `README.md`. I list the changes under a **HEAD** subsection.

```markdown
## Changelog
### HEAD (not yet released)
* Add foo to bar
* Fix: Baz not correctly responds to boo
```

Be sure to add your git `origin` as explained by GitHub, GitLab, Bitbucket
or whichever provider you chose.

For the first version, `lib/lolwat/version.rb` is already preconfigured
with `0.1.0`. That's a sensible default for a first version to play
around with. For later updates, increase the version number based on
[Semantic Versioning (»Semver«)](http://semver.org/#summary).

I prefer tracking the Changelog using the GitHub's Release system
(I'll get to that in a bit). To that end, I remove the bullet points
in my temporary »HEAD (not yet released) and commit the changed `README.md`
together with the changed `lib/lolwat/version.rb`:

```shell
git commit -m 'Release v0.2.0'
```

## Release the ~~Kraken~~ Gem!

Together with our [changes to the gemspec and Rakefile](#gemfury-support),
releasing the version is as simple as:

```shell
rake release
```

It will create a git version tag, push everything to your git remote, 
package the gemfile and upload the Gem file to Gemfury.

The last step is to head over to your GitHub project page, select **releases**
and **Draft a new Release** in the upper right corner.

![GitHub Releases Overview](/images/2017-07-03-github-release-overview.png)

The ensuing form is pretty self-explanatory (check the explanations on
the right side). For the release name, I just repeat the tag name (e.g. `v0.2.0`).
The text area gets all the info from my temporary Changelog tracker in
the `README.md` file.

## Summary

1. Create gem skeleton using `bundle gem`
2. Adapt `Rakefile` and `*.gemspec` to prevent pushes to public rubygems.org
3. Hackedy-hack-hack + track features/bugfixes in README
4. Update `version.rb`, remove changelog info, commit and `rake release`
5. Create release on GitHub and paste changelog data

Voilá!
