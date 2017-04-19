---
short-description: gettext internationalization utility
title: glib-gettextize
...

GLib

glib-gettextize

OPTION

DIRECTORY

# Description

`glib-gettextize` helps to prepare a source package for being
internationalized through gettext. It is a variant of the `gettextize`
that ships with gettext.

`glib-gettextize` differs from `gettextize` in that it doesn't create an
`intl/` subdirectory and doesn't modify `po/ChangeLog` (note that newer
versions of `gettextize` behave like this when called with the
`--no-changelog` option).

# Options

  - `--help`  
    print help and exit

  - `--version`  
    print version information and exit

  - `-c`, `--copy`  
    copy files instead of making symlinks

  - `-f`, `--force`  
    force writing of new files even if old ones exist

# See also

gettextize1
