---
short-description: How to compile GLib itself
title: Compiling the GLib Package
...

# Building the Library on UNIX

On UNIX, GLib uses the standard GNU build system, using autoconf for
package configuration and resolving portability issues, automake for
building makefiles that comply with the GNU Coding Standards, and
libtool for building shared libraries on multiple platforms. The normal
sequence for compiling and installing the GLib library is thus:
`./configure` `make` `make install`

The standard options provided by GNU autoconf may be passed to the
`configure` script. Please see the autoconf documentation or run
`./configure --help` for information about the standard options.

The GTK+ documentation contains [further
details](../gtk/gtk-building.html) about the build process and ways to
influence it.

# Dependencies

Before you can compile the GLib library, you need to have various other
tools and libraries installed on your system. Beyond a C compiler (which
must implement C90, but does not need to implement C99), the two tools
needed during the build process (as differentiated from the tools used
in when creating GLib mentioned above such as autoconf) are `pkg-config`
and GNU make.

  - [pkg-config](http://www.freedesktop.org/software/pkgconfig/) is a
    tool for tracking the compilation flags needed for libraries that
    are used by the GLib library. (For each library, a small `.pc` text
    file is installed in a standard location that contains the
    compilation flags needed for that library along with version number
    information.) The version of `pkg-config` needed to build GLib is
    mirrored in the `dependencies` directory on the [GTK+ FTP
    site.](ftp://ftp.gtk.org/pub/gtk/v2.2/)

  - The GLib Makefiles make use of several features specific to [GNU
    make](http://www.gnu.org/software/make), and will not build
    correctly with other versions of `make`. You will need to install it
    if you don't already have it on your system. (It may be called
    `gmake` rather than `make`.)

A UNIX build of GLib requires that the system implements at least the
original 1990 version of POSIX. Beyond this, it depends on a number of
other libraries.

  - The [GNU libiconv library](http://www.gnu.org/software/libiconv/) is
    needed to build GLib if your system doesn't have the `iconv()`
    function for doing conversion between character encodings. Most
    modern systems should have `iconv()`, however many older systems
    lack an `iconv()` implementation. On such systems, you must install
    the libiconv library. This can be found at:
    <http://www.gnu.org/software/libiconv>.
    
    If your system has an `iconv()` implementation but you want to use
    libiconv instead, you can pass the --with-libiconv option to
    configure. This forces libiconv to be used.
    
    Note that if you have libiconv installed in your default include
    search path (for instance, in `/usr/local/`), but don't enable it,
    you will get an error while compiling GLib because the `iconv.h`
    that libiconv installs hides the system iconv.
    
    If you are using the native iconv implementation on Solaris instead
    of libiconv, you'll need to make sure that you have the converters
    between locale encodings and UTF-8 installed. At a minimum you'll
    need the SUNWuiu8 package. You probably should also install the
    SUNWciu8, SUNWhiu8, SUNWjiu8, and SUNWkiu8 packages.
    
    The native iconv on Compaq Tru64 doesn't contain support for UTF-8,
    so you'll need to use GNU libiconv instead. (When using GNU libiconv
    for GLib, you'll need to use GNU libiconv for GNU gettext as well.)
    This probably applies to related operating systems as well.

  - The libintl library from the [GNU gettext
    package](http://www.gnu.org/software/gettext) is needed if your
    system doesn't have the `gettext()` functionality for handling
    message translation databases.

  - A thread implementation is needed. The thread support in GLib can be
    based upon POSIX threads or win32 threads.

  - GRegex uses the [PCRE library](http://www.pcre.org/) for regular
    expression matching. The default is to use the internal version of
    PCRE that is patched to use GLib for memory management and Unicode
    handling. If you prefer to use the system-supplied PCRE library you
    can pass the `--with-pcre=system` option to, but it is not
    recommended.

  - The optional extended attribute support in GIO requires the
    getxattr() family of functions that may be provided by glibc or by
    the standalone libattr library. To build GLib without extended
    attribute support, use the `--disable-xattr` option.

  - The optional SELinux support in GIO requires libselinux. To build
    GLib without SELinux support, use the `--disable-selinux` option.

  - The optional support for DTrace requires the `sys/sdt.h` header,
    which is provided by SystemTap on Linux. To build GLib without
    DTrace, use the `--disable-dtrace` configure option.

  - The optional support for
    [SystemTap](http://sourceware.org/systemtap/) can be disabled with
    the `--disable-systemtap` configure option.

# Extra Configuration Options

In addition to the normal options, the `configure` script in the GLib
library supports these additional arguments:

**--enable-debug.**

Turns on various amounts of debugging support. Setting this to 'no'
disables g\_assert(), g\_return\_if\_fail(), g\_return\_val\_if\_fail()
and all cast checks between different object types. Setting it to
'minimum' disables only cast checks. Setting it to 'yes' enables
[runtime debugging](running.markdown#g_debug). The default is 'minimum'.
Note that 'no' is fast, but dangerous as it tends to destabilize even
mostly bug-free software by changing the effect of many bugs from simple
warnings into fatal crashes. Thus `--enable-debug=no` should *not* be
used for stable releases of GLib.

**--disable-gc-friendly and --enable-gc-friendly.**

By default, and with --disable-gc-friendly as well, Glib does not clear
the memory for certain objects before they are freed. For example, Glib
may decide to recycle GList nodes by putting them in a free list.
However, memory profiling and debugging tools like
[Valgrind](http://www.valgrind.org) work better if an application does
not keep dangling pointers to freed memory (even though these pointers
are no longer dereferenced), or invalid pointers inside uninitialized
memory. The --enable-gc-friendly option makes Glib clear memory in these
situations:

  - When shrinking a GArray, Glib will clear the memory no longer
    available in the array: shrink an array from 10 bytes to 7, and the
    last 3 bytes will be cleared. This includes removals of single and
    multiple elements.

  - When growing a GArray, Glib will clear the new chunk of memory. Grow
    an array from 7 bytes to 10 bytes, and the last 3 bytes will be
    cleared.

  - The above applies to GPtrArray as well.

  - When freeing a node from a GHashTable, Glib will first clear the
    node, which used to have pointers to the key and the value stored at
    that node.

  - When destroying or removing a GTree node, Glib will clear the node,
    which used to have pointers to the node's value, and the left and
    right subnodes.

Since clearing the memory has a cost, --disable-gc-friendly is the
default.

**--disable-mem-pools and --enable-mem-pools.**

Many small chunks of memory are often allocated via collective pools in
GLib and are cached after release to speed up reallocations. For sparse
memory systems this behaviour is often inferior, so memory pools can be
disabled to avoid excessive caching and force atomic maintenance of
chunks through the `g_malloc()` and `g_free()` functions. Code currently
affected by this:

  - GMemChunks become basically non-effective

  - GSignal disables all caching (potentially very slow)

  - GType doesn't honour the GTypeInfo n\_preallocs field anymore

  - the GBSearchArray flag `G_BSEARCH_ALIGN_POWER2` becomes
    non-functional

**--with-threads.**

Specify a thread implementation to use. Available options are 'posix' or
'win32'. Normally, `configure` should be able to work out the system
threads API on its own.

**--disable-regex and --enable-regex.**

Do not compile GLib with regular expression support. GLib will be
smaller because it will not need the PCRE library. This is however not
recommended, as programs may need GRegex.

**--with-pcre.**

Specify whether to use the internal or the system-supplied PCRE library.

  - 'internal' means that GRegex will be compiled to use the internal
    PCRE library.

  - 'system' means that GRegex will be compiled to use the
    system-supplied PCRE library.

Using the internal PCRE is the preferred solution:

  - System-supplied PCRE has a separated copy of the big tables used for
    Unicode handling.

  - Some systems have PCRE libraries compiled without some needed
    features, such as UTF-8 and Unicode support.

  - PCRE uses some global variables for memory management and other
    features. In the rare case of a program using both GRegex and PCRE
    (maybe indirectly through a library), this variables could lead to
    problems when they are modified.

**--disable-included-printf and --enable-included-printf.**

By default the `configure` script will try to auto-detect whether the C
library provides a suitable set of printf() functions. In detail,
`configure` checks that the semantics of snprintf() are as specified by
C99 and that positional parameters as specified in the Single Unix
Specification are supported. If this not the case, GLib will include an
implementation of the printf() family.

These options can be used to explicitly control whether an
implementation of the printf() family should be included or not.

**--disable-Bsymbolic and --enable-Bsymbolic.**

By default, GLib uses the -Bsymbolic-functions linker flag to avoid
intra-library PLT jumps. A side-effect of this is that it is no longer
possible to override internal uses of GLib functions with LD\_PRELOAD.
Therefore, it may make sense to turn this feature off in some
situations. The `--disable-Bsymbolic` option allows to do that.

**--disable-gtk-doc and --enable-gtk-doc.**

By default the `configure` script will try to auto-detect whether the
gtk-doc package is installed. If it is, then it will use it to extract
and build the documentation for the GLib library. These options can be
used to explicitly control whether gtk-doc should be used or not. If it
is not used, the distributed, pre-generated HTML files will be installed
instead of building them on your machine.

**--disable-man and --enable-man.**

By default the `configure` script will try to auto-detect whether
xsltproc and the necessary Docbook stylesheets are installed. If they
are, then it will use them to rebuild the included man pages from the
XML sources. These options can be used to explicitly control whether man
pages should be rebuilt used or not. The distribution includes
pre-generated man pages.

**--disable-xattr and --enable-xattr.**

By default the `configure` script will try to auto-detect whether the
getxattr() family of functions is available. If it is, then extended
attribute support will be included in GIO. These options can be used to
explicitly control whether extended attribute support should be included
or not. getxattr() and friends can be provided by glibc or by the
standalone libattr library.

**--disable-selinux and --enable-selinux.**

By default the `configure` script will auto-detect if libselinux is
available and include SELinux support in GIO if it is. These options can
be used to explicitly control whether SELinux support should be
included.

**--disable-dtrace and --enable-dtrace.**

By default the `configure` script will detect if DTrace support is
available, and use it.

**--disable-systemtap and --enable-systemtap.**

This option requires DTrace support. If it is available, then the
`configure` script will also check for the presence of SystemTap.

**--enable-coverage and --disable-coverage.**

Enable the generation of coverage reports for the GLib tests. This
requires the lcov frontend to gcov from the [Linux Test
Project](http://ltp.sourceforge.net). To generate a coverage report, use
the lcov make target. The report is placed in the `glib-lcov` directory.

**--with-runtime-libdir=RELPATH.**

Allows specifying a relative path to where to install the runtime
libraries (meaning library files used for running, not developing, GLib
applications). This can be used in operating system setups where
programs using GLib needs to run before e.g. `/usr` is mounted. For
example, if LIBDIR is `/usr/lib` and `../../lib` is passed to
--with-runtime-libdir then the runtime libraries are installed into
`/lib` rather than `/usr/lib`.

**--with-python.**

Allows specifying the Python interpreter to use, either as an absolute
path, or as a program name. GLib can be built with Python 2 (at least
version 2.5) or Python 3.
