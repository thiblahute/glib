---
short-description: How to cross-compile GLib
title: Cross-compiling the GLib Package
...

# Building the Library for a different architecture

Cross-compilation is the process of compiling a program or library on a
different architecture or operating system then it will be run upon.
GLib is slightly more difficult to cross-compile than many packages
because much of GLib is about hiding differences between different
systems.

These notes cover things specific to cross-compiling GLib; for general
information about cross-compilation, see the autoconf info pages.

GLib tries to detect as much information as possible about the target
system by compiling and linking programs without actually running
anything; however, some information GLib needs is not available this
way. This information needs to be provided to the configure script via a
"cache file" or by setting the cache variables in your environment.

As an example of using a cache file, to cross compile for the "MingW32"
Win32 runtime environment on a Linux system, create a file 'win32.cache'
with the following contents:

``` 
 
glib_cv_long_long_format=I64
glib_cv_stack_grows=no
      
```

Then execute the following commands:

``` 
PATH=/path/to/mingw32-compiler/bin:$PATH
chmod a-w win32.cache   # prevent configure from changing it
./configure --cache-file=win32.cache --host=mingw32
      
```

The complete list of cache file variables follows. Most of these won't
need to be set in most cases.

# Cache file variables

**glib\_cv\_long\_long\_format=\[ll/q/I64\].**

Format used by `printf()` and `scanf()` for 64 bit integers. "ll" is the
C99 standard, and what is used by the 'trio' library that GLib builds if
your `printf()` is insufficiently capable. Doesn't need to be set if you
are compiling using trio.

**glib\_cv\_stack\_grows=\[yes/no\].**

Whether the stack grows up or down. Most places will want "no", A few
architectures, such as PA-RISC need "yes".

**glib\_cv\_working\_bcopy=\[yes/no\].**

Whether your `bcopy()` can handle overlapping copies. Only needs to be
set if you don't have `memmove()`. (Very unlikely)

**glib\_cv\_sane\_realloc=\[yes/no\].**

Whether your `realloc()` conforms to ANSI C and can handle `NULL` as the
first argument. Defaults to "yes" and probably doesn't need to be set.

**glib\_cv\_have\_strlcpy=\[yes/no\].**

Whether you have `strlcpy()` that matches OpenBSD. Defaults to "no",
which is safe, since GLib uses a built-in version in that case.

**glib\_cv\_have\_qsort\_r=\[yes/no\].**

Whether you have `qsort_r()` that matches BSD. Defaults to "no", which
is safe, since GLib uses a built-in version in that case.

**glib\_cv\_va\_val\_copy=\[yes/no\].**

Whether `va_list` can be copied as a pointer. If set to "no", then
`memcopy()` will be used. Only matters if you don't have `va_copy()` or
`__va_copy()`. (So, doesn't matter for GCC.) Defaults to "yes" which is
slightly more common than "no".

**glib\_cv\_rtldglobal\_broken=\[yes/no\].**

Whether you have a bug found in OSF/1 v5.0. Defaults to "no".

**glib\_cv\_uscore=\[yes/no\].**

Whether an underscore needs to be prepended to symbols when looking them
up via `dlsym()`. Only needs to be set if your system uses
`dlopen()`/`dlsym()`.

**ac\_cv\_func\_posix\_getpwuid\_r=\[yes/no\].**

Whether you have a getpwuid\_r function (in your C library, not your
thread library) that conforms to the POSIX spec. (Takes a 'struct passwd
\*\*' as the final argument)

**ac\_cv\_func\_nonposix\_getpwuid\_r=\[yes/no\].**

Whether you have some variant of `getpwuid_r()` that doesn't conform to
to the POSIX spec, but GLib might be able to use (or might segfault.)
Only needs to be set if `ac_cv_func_posix_getpwuid_r` is not set. It's
safest to set this to "no".

**ac\_cv\_func\_posix\_getgrgid\_r=\[yes/no\].**

Whether you have a getgrgid\_r function that conforms to the POSIX spec.

**glib\_cv\_use\_pid\_surrogate=\[yes/no\].**

Whether to use a `setpriority()` on the PID of the thread as a method
for setting the priority of threads. This only needs to be set when
using POSIX threads.

**ac\_cv\_func\_printf\_unix98=\[yes/no\].**

Whether your `printf()` family supports Unix98 style `%N$` positional
parameters. Defaults to "no".

**ac\_cv\_func\_vsnprintf\_c99=\[yes/no\].**

Whether you have a `vsnprintf()` with C99 semantics. (C99 semantics
means returning the number of bytes that would have been written had the
output buffer had enough space.) Defaults to "no".
