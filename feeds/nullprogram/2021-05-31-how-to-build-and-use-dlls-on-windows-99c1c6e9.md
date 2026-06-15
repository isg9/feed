---
title: How to build and use DLLs on Windows
url: https://nullprogram.com/blog/2021/05/31/
published: "2021-05-31T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2021/05/31/
---

# How to build and use DLLs on Windows

## [How to build and use DLLs on Windows](/blog/2021/05/31/)

May 31, 2021

nullprogram.com/blog/2021/05/31/

I’ve recently been involved with a couple of discussions about Windows’ dynamic linking. One was [Joe Nelson](https://begriffs.com/) in considering how to make [libderp](https://github.com/begriffs/libderp) accessible on Windows, and the other was about [w64devkit](/blog/2020/05/15/), my Mingw-w64 distribution. I use these techniques so infrequently that I need to figure it all out again each time I need it. Unfortunately there’s a whole lot of outdated and incorrect information online which gets in the way every time this happens. While it’s all fresh in my head, I will now document what I know works.

In this article, all commands and examples are being run in the context of w64devkit (1.8.0).

### Mingw-w64

If all you care about is the GNU toolchain then DLLs are straightforward, working mostly like shared objects on other platforms. To illustrate, let’s build a “square” library with one “exported” function, `square`, that returns the square of its input ( `square.c`):

```
long square(long x)
{
    return x * x;
}

```

The header file ( `square.h`):

```
#ifndef SQUARE_H
#define SQUARE_H

long square(long);

#endif

```

To build a stripped, size-optimized DLL, `square.dll`:

```
$ cc -shared -Os -s -o square.dll square.c

```

Now a test program to link against it ( `main.c`), which “imports” `square` from `square.dll`:

```
#include
#include "square.h"

int main(void)
{
    printf("%ld\n", square(2));
}

```

Linking and testing it:

```
$ cc -Os -s main.c square.dll
$ ./a
4

```

It’s that simple. Or more traditionally, using the `-l` flag:

```
$ cc -Os -s -L. main.c -lsquare

```

Given `-lxyz` GCC will look for `xyz.dll` in the library path.

#### Viewing exported symbols

Given a DLL, printing a list of the exported functions of a DLL is not so straightforward. For ELF shared objects there’s `nm -D`, but despite what the internet will tell you, this tool does not support DLLs. `objdump` will print the exports as part of the “private” headers ( `-p`). A bit of `awk` can cut this down to just a list of exports. Since we’ll need this a few times, here’s a script, `exports.sh`, that composes `objdump` and `awk` into the tool I want:

```
#!/bin/sh
set -e
printf 'LIBRARY %s\nEXPORTS\n' "$1"
objdump -p "$1" | awk '/^$/{t=0} {if(t)print$NF} /^\[O/{t=1}'

```

Running this on `square.dll` above:

```
$ ./exports.sh square.dll
LIBRARY square.dll
EXPORTS
square

```

This can be helpful when debugging. It also works outside of Windows, such as on Linux. By the way, the output format is no accident: This is the [`.def` file format](https://sourceware.org/binutils/docs/binutils/def-file-format.html) ( [also](https://www.willus.com/mingw/yongweiwu_stdcall.html)), which will be particularly useful in a moment.

Mingw-w64 has a `gendef` tool to produce the above output, and this tool is now included in w64devkit. To print the exports to standard output:

```
$ gendef - square.dll
LIBRARY "square.dll"
EXPORTS
square

```

Alternatively Visual Studio provides `dumpbin`. It’s not as concise as `exports.sh` but it’s a lot less verbose than `objdump -p`.

```
$ dumpbin /nologo /exports square.dll
...
          1    0 000012B0 square
...

```

#### Mingw-w64 (improved)

You can get by without knowing anything more, which is usually enough for those looking to support Windows as a secondary platform, even just as a cross-compilation target. However, with a bit more work we can do better. Imagine doing the above with a non-trivial program. GCC doesn’t know which functions are part of the API and which are not. Obviously static functions should not be exported, but what about non-static functions visible between translation units (i.e. object files)?

For instance, suppose `square.c` also has this function which is not part of its API but may be called by another translation unit.

```
void internal_func(void) {}

```

Now when I build:

```
$ ./exports.sh square.dll
LIBRARY square.dll
EXPORTS
internal_func
square

```

On the other side, when I build `main.c` how does it know which functions are imported from a DLL and which will be found in another translation unit? GCC makes it work regardless, but it can generate more efficient code if it knows at compile time (vs. link time).

On Windows both are solved by adding `__declspec` notation on both sides. In `square.c` the exports are marked as `dllexport`:

```
__declspec(dllexport)
long square(long x)
{
    return x * x;
}

void internal_func(void) {}

```

In the header, it’s marked as an import:

```
__declspec(dllimport)
long square(long);

```

The mere presence of `dllexport` tells the linker to only export those functions marked as exports, and so `internal_func` disappears from the exports list. Convenient!

On the import side, during compilation of the original program, GCC assumed `square` wasn’t an import and generated a local function call. When the linker later resolved the symbol to the DLL, it generated a trampoline to fill in as that local function (like a [PLT](https://www.airs.com/blog/archives/41)). With `dllimport`, GCC knows it’s an imported function and so doesn’t go through a trampoline.

While generally unnecessary for the GNU toolchain, it’s good hygiene to use `__declspec`. It’s also mandatory when using [MSVC](https://en.wikipedia.org/wiki/Microsoft_Visual_C%2B%2B), in case you care about that as well.

### MSVC

Mingw-w64-compiled DLLs will work with `LoadLibrary` out of the box, which is sufficient in many cases, such as for dynamically-loaded plugins. For example ( `loadlib.c`):

```
#include
#include

int main(void)
{
    HANDLE h = LoadLibrary("square.dll");
    long (*square)(long) = GetProcAddress(h, "square");
    printf("%ld\n", square(2));
}

```

Compiled with MSVC `cl` (via [`vcvars.bat`](/blog/2016/06/13/#visual-c)):

```
$ cl /nologo loadlib.c
$ ./loadlib
4

```

However, the MSVC linker, unlike Binutils `ld`, cannot link directly with DLLs. It requires an *import library*. Conventionally this matches the DLL name but has a `.lib` extension — `square.lib` in this case. The Mingw-w64 ecosystem conventionally uses `.dll.a`, as in `square.dll.a`, in order to distinguish it from a static library, but it’s the same format. The most convenient way to get an import library is to ask GCC to generate one at link-time via `--out-implib`:

```
$ cc -shared -Wl,--out-implib,square.lib -o square.dll square.c

```

Back to `cl`, just add `square.lib` as another input. You don’t actually need `square.dll` present at link time.

```
$ cl /nologo /Os main.c square.lib
$ ./main
4

```

What if you already have the DLL and you just need an import library? GNU Binutils’ `dlltool` can do this, though not without help. It cannot generate an import library from a DLL alone since it requires a `.def` file enumerating the exports. (Why?) What luck that we have a tool for this!

```
$ ./exports.sh square.dll >square.def
$ dlltool --input-def square.def --output-lib square.lib

```

### Reversing directions

Going the other way, building a DLL with MSVC and linking it with Mingw-w64, is nearly as easy as the pure Mingw-w64 case, though it requires that all exports are tagged with `dllexport`. The `/LD` (case sensitive) is just like GCC’s `-shared`.

```
$ cl /nologo /LD /Os square.c
$ cc -Os -s main.c square.dll
$ ./a
4

```

`cl` outputs three files: `square.dll`, `square.lib`, and `square.exp`. The last can be discarded, and the second will be needed if linking with MSVC, but as before, Mingw-w64 requires only the first.

This all demonstrates that Mingw-w64 and MSVC are quite interoperable — at least for C interfaces that [don’t share CRT objects](/blog/2023/08/27/).

### Tying it all together

If your program is designed to be portable, those `__declspec` will get in the way. That can be tidied up with some macros, but even better, those macros can be used to control ELF symbol visibility so that the library has good hygiene on, say, Linux as well.

The strategy will be to mark all API functions with `SQUARE_API` and expand that to whatever is necessary at the time. When building a library, it will expand to `dllexport`, or default visibility on unix-likes. When consuming a library it will expand to `dllimport`, or nothing outside of Windows. The new `square.h`:

```
#ifndef SQUARE_H
#define SQUARE_H

#if defined(SQUARE_BUILD)
#  if defined(_WIN32)
#    define SQUARE_API __declspec(dllexport)
#  elif defined(__ELF__)
#    define SQUARE_API __attribute__ ((visibility ("default")))
#  else
#    define SQUARE_API
#  endif
#else
#  if defined(_WIN32)
#    define SQUARE_API __declspec(dllimport)
#  else
#    define SQUARE_API
#  endif
#endif

SQUARE_API
long square(long);

#endif

```

The new `square.c`:

```
#define SQUARE_BUILD
#include "square.h"

SQUARE_API
long square(long x)
{
    return x * x;
}

```

`main.c` remains the same. When compiling on unix-like systems, add the `-fvisibility=hidden` to hide all symbols by default so that this macro can reveal them.

```
$ cc -shared -Os -fvisibility=hidden -s -o libsquare.so square.c
$ cc -Os -s main.c ./libsquare.so
$ ./a.out
4

```

### Makefile ideas

While Mingw-w64 hides a lot of the differences between Windows and unix-like systems, when it comes to dynamic libraries it can only do so much, especially if you care about import libraries. If I were maintaining a dynamic library — unlikely since I strongly prefer embedding or static linking — I’d probably just use different [Makefiles](/blog/2017/08/20/) per toolchain and target. Aside from the `SQUARE_API` type of macros, the source code can fortunately remain fairly agnostic about it.

Here’s what I might use as `NMakefile` for MSVC `nmake`:

```
CC     = cl /nologo
CFLAGS = /Os

all: main.exe square.dll square.lib

main.exe: main.c square.h square.lib
	$(CC) $(CFLAGS) main.c square.lib

square.dll: square.c square.h
	$(CC) /LD $(CFLAGS) square.c

square.lib: square.dll

clean:
	-del /f main.exe square.dll square.lib square.exp

```

Usage:

```
nmake /nologo /f NMakefile

```

For w64devkit and cross-compiling, `Makefile.w64`, which includes import library generation for the sake of MSVC consumers:

```
CC      = cc
CFLAGS  = -Os
LDFLAGS = -s
LDLIBS  =

all: main.exe square.dll square.lib

main.exe: main.c square.dll square.h
	$(CC) $(CFLAGS) $(LDFLAGS) -o $@ main.c square.dll $(LDLIBS)

square.dll: square.c square.h
	$(CC) -shared -Wl,--out-implib,$(@:dll=lib) \
	    $(CFLAGS) $(LDFLAGS) -o $@ square.c $(LDLIBS)

square.lib: square.dll

clean:
	rm -f main.exe square.dll square.lib

```

Usage:

```
make -f Makefile.w64

```

And a `Makefile` for everyone else:

```
CC      = cc
CFLAGS  = -Os -fvisibility=hidden
LDFLAGS = -s
LDLIBS  =

all: main libsquare.so

main: main.c libsquare.so square.h
	$(CC) $(CFLAGS) $(LDFLAGS) -o $@ main.c ./libsquare.so $(LDLIBS)

libsquare.so: square.c square.h
	$(CC) -shared $(CFLAGS) $(LDFLAGS) -o $@ square.c $(LDLIBS)

clean:
	rm -f main libsquare.so

```

Now that I have this article, I’m glad I won’t have to figure this all out again next time I need it!

- [win32](/tags/win32/)
- [c](/tags/c/)
- [cpp](/tags/cpp/)
- [linux](/tags/linux/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20How%20to%20build%20and%20use%20DLLs%20on%20Windows) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=How+to+build+and+use+DLLs+on+Windows).

« [The cost of Java's EnumSet](/blog/2021/04/23/)

» [More DLL fun with w64devkit: Go, assembly, and Python](/blog/2021/06/29/)
