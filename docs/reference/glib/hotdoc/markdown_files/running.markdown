---
short-description: How to run and debug your GLib application
title: Running GLib Applications
...

# Running and debugging GLib Applications

## Environment variables

The runtime behaviour of GLib applications can be influenced by a number
of environment variables.

**Standard variables.**

GLib reads standard environment variables like LANG, PATH, HOME, TMPDIR,
TZ and LOGNAME.

**XDG directories.**

GLib consults the environment variables XDG\_DATA\_HOME,
XDG\_DATA\_DIRS, XDG\_CONFIG\_HOME, XDG\_CONFIG\_DIRS, XDG\_CACHE\_HOME
and XDG\_RUNTIME\_DIR for the various XDG directories. For more
information, see the [XDG basedir
spec](http://standards.freedesktop.org/basedir-spec/basedir-spec-latest.html).

**G\_FILENAME\_ENCODING.**

This environment variable can be set to a comma-separated list of
character set names. GLib assumes that filenames are encoded in the
first character set from that list rather than in UTF-8. The special
token "@locale" can be used to specify the character set for the current
locale.

**G\_BROKEN\_FILENAMES.**

If this environment variable is set, GLib assumes that filenames are in
the locale encoding rather than in UTF-8. G\_FILENAME\_ENCODING takes
priority over G\_BROKEN\_FILENAMES.

**G\_MESSAGES\_PREFIXED.**

A list of log levels for which messages should be prefixed by the
program name and PID of the application. The default is to prefix
everything except `G_LOG_LEVEL_MESSAGE` and `G_LOG_LEVEL_INFO`. The
possible values are `error`, `warning`, `critical`, `message`, `info`
and `debug`. You can also use the special values `all` and `help`.

This environment variable only affects the default log handler,
g\_log\_default\_handler().

**G\_MESSAGES\_DEBUG.**

A space-separated list of log domains for which informational and debug
messages should be printed. By default, these messages are not printed.

You can also use the special value `all`.

This environment variable only affects the default log handler,
g\_log\_default\_handler().

**G\_DEBUG.**

This environment variable can be set to a list of debug options, which
cause GLib to print out different types of debugging information.

  - fatal-warnings  
    Causes GLib to abort the program at the first call to g\_warning()
    or g\_critical().

  - fatal-criticals  
    Causes GLib to abort the program at the first call to g\_critical().

  - gc-friendly  
    Newly allocated memory that isn't directly initialized, as well as
    memory being freed will be reset to 0. The point here is to allow
    memory checkers and similar programs that use Boehm GC alike
    algorithms to produce more accurate results.

  - resident-modules  
    All modules loaded by GModule will be made resident. This can be
    useful for tracking memory leaks in modules which are later
    unloaded; but it can also hide bugs where code is accessed after the
    module would have normally been unloaded.

  - bind-now-modules  
    All modules loaded by GModule will bind their symbols at load time,
    even when the code uses %G\_MODULE\_BIND\_LAZY.

The special value all can be used to turn on all debug options. The
special value help can be used to print all available options.

**G\_SLICE.**

This environment variable allows reconfiguration of the GSlice memory
allocator.

  - always-malloc  
    This will cause all slices allocated through g\_slice\_alloc() and
    released by g\_slice\_free1() to be actually allocated via direct
    calls to g\_malloc() and g\_free(). This is most useful for memory
    checkers and similar programs that use Boehm GC alike algorithms to
    produce more accurate results. It can also be in conjunction with
    debugging features of the system's malloc() implementation such as
    glibc's MALLOC\_CHECK\_=2 to debug erroneous slice allocation code,
    although `debug-blocks` is usually a better suited debugging tool.

  - debug-blocks  
    Using this option (present since GLib 2.13) engages extra code which
    performs sanity checks on the released memory slices. Invalid slice
    addresses or slice sizes will be reported and lead to a program
    halt. This option is for debugging scenarios. In particular, client
    packages sporting their own test suite should *always enable this
    option when running tests*. Global slice validation is ensured by
    storing size and address information for each allocated chunk, and
    maintaining a global hash table of that data. That way, multi-thread
    scalability is given up, and memory consumption is increased.
    However, the resulting code usually performs acceptably well,
    possibly better than with comparable memory checking carried out
    using external tools.
    
    An example of a memory corruption scenario that cannot be reproduced
    with `G_SLICE=always-malloc`, but will be caught by
    `G_SLICE=debug-blocks` is as
    follows:
    
    ``` 
                void *slist = g_slist_alloc (); /* void* gives up type-safety */
                g_list_free (slist);            /* corruption: sizeof (GSList) != sizeof (GList) */
              
    ```

The special value all can be used to turn on all options. The special
value help can be used to print all available options.

**G\_RANDOM\_VERSION.**

If this environment variable is set to '2.0', the outdated pseudo-random
number seeding and generation algorithms from GLib 2.0 are used instead
of the newer, better ones. You should only set this variable if you have
sequences of numbers that were generated with Glib 2.0 that you need to
reproduce exactly.

**LIBCHARSET\_ALIAS\_DIR.**

Allows to specify a nonstandard location for the `charset.aliases` file
that is used by the character set conversion routines. The default
location is the libdir specified at compilation time.

**TZDIR.**

Allows to specify a nonstandard location for the timezone data files
that are used by the \#GDateTime API. The default location is under
`/usr/share/zoneinfo`. For more information, also look at the `tzset`
manual page.

## Locale

A number of interfaces in GLib depend on the current locale in which an
application is running. Therefore, most GLib-using applications should
call `setlocale (LC_ALL, "")` to set up the current locale.

On Windows, in a C program there are several locale concepts that not
necessarily are synchronized. On one hand, there is the system default
ANSI code-page, which determines what encoding is used for file names
handled by the C library's functions and the Win32 API. (We are talking
about the "narrow" functions here that take character pointers, not the
"wide" ones.)

On the other hand, there is the C library's current locale. The
character set (code-page) used by that is not necessarily the same as
the system default ANSI code-page. Strings in this character set are
returned by functions like `strftime()`.

glib ships with a set of python macros for the gdb debugger. These
includes pretty printers for lists, hashtables and gobject types. It
also has a backtrace filter that makes backtraces with signal emissions
easier to read.

To use this you need a recent enough gdb that supports python scripting.
Gdb 7.0 should be recent enough, but branches of the "archer" gdb tree
as used in Fedora 11 and Fedora 12 should work too. You then need to
install glib in the same prefix as gdb so that the python gdb autoloaded
files get installed in the right place for gdb to pick up.

General pretty printing should just happen without having to do anything
special. To get the signal emission filtered backtrace you must use the
"new-backtrace" command instead of the standard one.

There is also a new command called gforeach that can be used to apply a
command on each item in a list. E.g. you can do

    gforeach i in some_list_variable: print *(GtkWidget *)l

Which would print the contents of each widget in a list of widgets.

## SystemTap

[SystemTap](http://sourceware.org/systemtap/) is a dynamic whole-system
analysis toolkit. GLib ships with a file `libglib-2.0.so.*.stp` which
defines a set of probe points, which you can hook into with custom
SystemTap scripts. See the files `libglib-2.0.so.*.stp`,
`libgobject-2.0.so.*.stp` and `libgio-2.0.so.*.stp` which are in your
shared SystemTap scripts directory.

## Memory statistics

g\_mem\_profile() will output a summary g\_malloc() memory usage, if
memory profiling has been enabled by calling `g_mem_set_vtable
(glib_mem_profiler_table)` upon startup.

If GLib has been configured with `--enable-debug=yes`, then
g\_slice\_debug\_tree\_statistics() can be called in a debugger to
output details about the memory usage of the slice allocator.
