---
short-description: General considerations when programming with GLib
title: Writing GLib Applications
...

# Writing GLib Applications

## Threads

The general policy of GLib is that all functions are invisibly
threadsafe with the exception of data structure manipulation functions,
where, if you have two threads manipulating the *same* data structure,
they must use a lock to synchronize their operation.

GLib creates a worker thread for its own purposes so GLib applications
will always have at least 2 threads.

See the sections on [threads](#glib-Threads) and
[threadpools](#glib-Thread-Pools) for GLib APIs that support
multithreaded applications.

## Security

When writing code that runs with elevated privileges, it is important to
follow some basic rules of secure programming. David Wheeler has an
excellent book on this topic, [Secure Programming for Linux and Unix
HOWTO](http://www.dwheeler.com/secure-programs/Secure-Programs-HOWTO/index.html).

When it comes to GLib and its associated libraries, GLib and GObject are
generally fine to use in code that runs with elevated privileges; they
don't load modules (executable code in shared objects) or run other
programs 'behind your back'. GIO has to be used carefully in privileged
programs, see the [GIO
documentation](http://developer.gnome.org/gio/stable/ch02.html) for
details.
