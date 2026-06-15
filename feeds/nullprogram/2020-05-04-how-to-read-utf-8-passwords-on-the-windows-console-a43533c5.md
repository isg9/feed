---
title: How to Read UTF-8 Passwords on the Windows Console
url: https://nullprogram.com/blog/2020/05/04/
published: "2020-05-04T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2020/05/04/
---

# How to Read UTF-8 Passwords on the Windows Console

## [How to Read UTF-8 Passwords on the Windows Console](/blog/2020/05/04/)

May 04, 2020

nullprogram.com/blog/2020/05/04/

*This article was discussed [on Hacker News](https://news.ycombinator.com/item?id=23064864).*

Suppose you’re writing a command line program that [prompts the user for
a password or passphrase](/blog/2017/03/12/), and Windows is one of the supported platforms ( [even very old versions](/blog/2018/04/13/)). This program uses [UTF-8
for its string representation](/blog/2019/05/29/), [as it should](http://utf8everywhere.org/), and so ideally it receives the password from the user encoded as UTF-8. On most platforms this is, for the most part, automatic. However, on Windows finding the correct answer to this problem is a maze where all the signs lead towards dead ends. I recently navigated this maze and found the way out.

I knew it was possible because [my passphrase2pgp tool](/blog/2019/07/10/) has been using the [golang.org/x/crypto/ssh/terminal](https://pkg.go.dev/golang.org/x/crypto/ssh/terminal) package, which gets it very nearly perfect. Though they were still fixing subtle bugs [as
recently as 6 months ago](https://github.com/golang/crypto/commit/6d4e4cb37c7d6416dfea8472e751c7b6615267a6).

The first step is to ignore just everything you find online, because it’s either wrong or it’s solving a slightly different problem. I’ll discuss the dead ends later and focus on the solution first. Ultimately I want to implement this on Windows:

```
// Display prompt then read zero-terminated, UTF-8 password.
// Return password length with terminator, or zero on error.
int read_password(char *buf, int len, const char *prompt);

```

I chose `int` for the length rather than `size_t` because it’s a password and should not even approach `INT_MAX`.

### The correct way

For the impatient: [**complete, working, ready-to-use example**](https://github.com/skeeto/scratch/blob/master/misc/read-password-w32.c)

On a unix-like system, the program would:

1. `open(2)` the special `/dev/tty` file for reading and writing
2. `write(2)` the prompt
3. `tcgetattr(3)` and `tcsetattr(3)` to disable `ECHO`
4. `read(2)` a line of input
5. Restore the old terminal attributes with `tcsetattr(3)`
6. `close(2)` the file

A great advantage of this approach is that it doesn’t depend on standard input and standard output. Either or both can be redirected elsewhere, and this function still interacts with the user’s terminal. The Windows version will have the same advantage.

Despite some tempting shortcuts that don’t work, the steps on Windows are basically the same but with different names. There are a couple subtleties and extra steps. I’ll be ignoring errors in my code snippets below, but the complete example has full error handling.

#### Create console handles

Instead of `/dev/tty`, the program opens two files: `CONIN$` and `CONOUT$` using [`CreateFileA()`](https://docs.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-createfilea). Note: The “A” stands for ANSI, as opposed to “W” for wide (Unicode). This refers to the encoding of the file name, not to how the file contents are encoded. `CONIN$` is opened for both reading and writing because write permissions are needed to change the console’s mode.

```
HANDLE hi = CreateFileA(
    "CONIN$",
    GENERIC_READ | GENERIC_WRITE,
    0,
    0,
    OPEN_EXISTING,
    0,
    0
);
HANDLE ho = CreateFileA(
    "CONOUT$",
    GENERIC_WRITE,
    0,
    0,
    OPEN_EXISTING,
    0,
    0
);

```

#### Print the prompt

To write the prompt, call [`WriteConsoleA()`](https://docs.microsoft.com/en-us/windows/console/writeconsole) on the output handle. On its own, this assumes the prompt is plain ASCII (i.e. `"password: "`), not UTF-8 (i.e. `"contraseña: "`):

```
WriteConsoleA(ho, prompt, strlen(prompt), 0, 0);

```

If the prompt may contain UTF-8 data, perhaps because it displays a username or isn’t in English, you have two options:

- Convert the prompt to UTF-16 and call `WriteConsoleW()` instead.
- Use `SetConsoleOutputCP()` with `CP_UTF8` (65001). This is a global (to the console) setting and should be restored when done.

#### Disable echo

Next use [`GetConsoleMode()`](https://docs.microsoft.com/en-us/windows/console/getconsolemode) and [`SetConsoleMode()`](https://docs.microsoft.com/en-us/windows/console/setconsolemode) to disable echo. The console usually has `ENABLE_PROCESSED_INPUT` already set, which tells the console to handle CTRL-C and such, but I set it explicitly just in case. I also set `ENABLE_LINE_INPUT` so that the user can use backspace and so that the entire line is delivered at once.

```
DWORD orig = 0;
GetConsoleMode(hi, &orig);

DWORD mode = orig;
mode |= ENABLE_PROCESSED_INPUT;
mode &= ~ENABLE_ECHO_INPUT;
SetConsoleMode(hi, mode);

```

There are reports that `ENABLE_LINE_INPUT` limits reads to 254 bytes, but I was unable to reproduce it. My full example can read huge passwords without trouble.

The old mode is saved in `orig` so that it can be restored later.

#### Read the password

Here’s where you have to pay the piper. As of the date of this article, **the Windows API offers no method for reading UTF-8 input from the** **console**. Give up on that hope now. If you use the “ANSI” functions to read input under any configuration, they will to the usual Windows thing of *silently mangling your input*.

So you *must* use the UTF-16 API, [`ReadConsoleW()`](https://docs.microsoft.com/en-us/windows/console/readconsole), and then [encode it](/blog/2017/10/06/) yourself. Fortunately Win32 provides a UTF-8 encoder, [`WideCharToMultiByte()`](https://docs.microsoft.com/en-us/windows/win32/api/stringapiset/nf-stringapiset-widechartomultibyte), which will even handle surrogate pairs for all those people who like putting `PILE OF POO` ( `U+1F4A9`) in their passwords:

```
SIZE_T wbuf_len = (len - 1 + 2)*sizeof(*wbuf);
WCHAR *wbuf = HeapAlloc(GetProcessHeap(), 0, wbuf_len);
DWORD nread;
ReadConsoleW(hi, wbuf, len - 1 + 2, &nread, 0);
wbuf[nread-2] = 0;  // truncate "\r\n"
int r = WideCharToMultiByte(CP_UTF8, 0, wbuf, -1, buf, len, 0, 0);
SecureZeroMemory(wbuf, wbuf_len);
HeapFree(GetProcessHeap(), 0, wbuf);

```

I use [`SecureZeroMemory()`](https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/wdm/nf-wdm-rtlsecurezeromemory) to erase the UTF-16 version of the password before freeing the buffer. The `+ 2` in the allocation is for the CRLF line ending that will later be chopped off. The error handling version checks that the input did indeed end with CRLF. Otherwise it was truncated (too long).

#### Clean up

Finally print a newline since the user-typed one wasn’t echoed, restore the old console mode, close the console handles, and return the final encoded length:

```
WriteConsoleA(ho, "\n", 1, 0, 0);
SetConsoleMode(hi, orig);
CloseHandle(ho);
CloseHandle(hi);
return r;

```

The error checking version doesn’t check for errors from any of these functions since either they cannot fail, or there’s nothing reasonable to do in the event of an error.

### Dead ends

If you look around the Win32 API you might notice `SetConsoleCP()`. A reasonable person might think that setting the “code page” to UTF-8 ( `CP_UTF8`) might configure the console to encode input in UTF-8. The good news is Windows will no longer mangle your input as before. The bad news is that it will be mangled differently.

You might think you can use the CRT function `_setmode()` with `_O_U8TEXT` on the `FILE *` connected to the console. This does nothing useful. (The only use for `_setmode()` is with `_O_BINARY`, to disable braindead character translation on standard input and output.) The best you’ll be able to do with the CRT is the same sort of wide character read using non-standard functions, followed by conversion to UTF-8.

[`CredUICmdLinePromptForCredentials()`](https://docs.microsoft.com/en-us/windows/win32/api/wincred/nf-wincred-creduicmdlinepromptforcredentialsa) promises to be both a mouthful of a function name, and a prepacked solution to this problem. It only delivers on the first. This function seems to have broken some time ago and nobody at Microsoft noticed — probably because *nobody has* *ever used this function*. I couldn’t find a working example, nor a use in any real application. When I tried to use it, I got a nonsense error code it never worked. There’s a GUI version of this function that *does* work, and it’s a viable alternative for certain situations, though not mine.

At my most desperate, I hoped `ENABLE_VIRTUAL_TERMINAL_PROCESSING` would be a magical switch. On Windows 10 it magically enables some ANSI escape sequences. The documentation in no way suggests it *would* work, and I confirmed by experimentation that it does not. Pity.

I spent a lot of time searching down these dead ends until finally settling with `ReadConsoleW()` above. I hoped it would be more automatic, but I’m glad I have at least *some* solution figured out.

- [win32](/tags/win32/)
- [c](/tags/c/)
- [tutorial](/tags/tutorial/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20How%20to%20Read%20UTF-8%20Passwords%20on%20the%20Windows%20Console) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=How+to+Read+UTF-8+Passwords+on+the+Windows+Console).

« [When Parallel: Pull, Don't Push](/blog/2020/04/30/)

» [w64devkit: a Portable C and C++ Development Kit for Windows](/blog/2020/05/15/)
