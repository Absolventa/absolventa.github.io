---
layout: post
title: "Upgrading from a PostgreSQL version Homebrew already abandoned"
date: 2017-01-16
author: carp
comments: true
tags:
  - homebrew
  - postgresql
teaser: "
While Homebrew supports installing older versions of a piece of software,
not all old versions remain available indefinitely. This article briefly
describes how to migrate your PostgreSQL data from a version whose tap
had been removed from Homebrew's index.
"
---

When my PostgreSQL installation didn't come up after [an unexpected reboot][moss_reboot]
and showed errors about a missing `libreadline.dylib` (a familiar message after
upgrading to macOS Sierra), I figured I could remedy it with a simple `brew
reinstall postgresql`. Sadly, the PostgreSQL version Homebrew reinstalled was not
the one that was already installed, but the most recent version it knew of.

I may have cranked  `brew cleanup postgresql` a tad bit too early 😱 and so I ended up
with my 9.3 installation files already gone and only v9.6.1 available. Sadly, the
old binaries are needed to access the old data format.

The earliest PostgreSQL version Homebrew still had in stock was 9.4. I checked
[postgresql.org][download_overview] to see whether
there were old source or binary distribution tarballs available. Indeed, it
had a link pointing advanced users to a [list of zip files containing binaries][download_binaries].

With all tools prepared ([and some cross-referencing][upgrade_gist]) I could repair
my broken installation (assuming the new pgsql is already installed):

```shell
# Move the old data files out of the way
mv /usr/local/var/postgres /usr/local/var/postgres93

# Since my computer previously crashed, I had a stale pid file
rm /usr/local/var/postgres93/postmaster.pid

# Create a new data dir for the most recent pgsql version
initdb --pgdata=/usr/local/var/postgres

# Migrate data files, pointing old-bindir to the location of
# the extracted zip file
pg_upgrade \
  --old-datadir=/usr/local/var/postgres93 \
  --new-datadir=/usr/local/var/postgres \
  --old-bindir=$HOME/Downloads/pgsql/bin \
  --new-bindir=/usr/local/Cellar/postgresql/9.6.1/bin \
  --verbose

# Start PostgreSQL again
brew services start postgresql

```

Everthing should be up and running again.

[moss_reboot]: https://www.youtube.com/watch?v=EKtPWGvmAXw "I tried turning it on off and on again"
[download_overview]: https://www.postgresql.org/download/macosx/
[download_binaries]: https://www.enterprisedb.com/download-postgresql-binaries
[upgrade_gist]: https://gist.github.com/chbrown/647a54dc3e1c2e8c7395
