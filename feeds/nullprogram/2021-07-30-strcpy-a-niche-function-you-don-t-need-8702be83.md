---
title: 'strcpy: a niche function you don''t need'
url: https://nullprogram.com/blog/2021/07/30/
published: "2021-07-30T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2021/07/30/
---

# strcpy: a niche function you don't need

## [strcpy: a niche function you don't need](/blog/2021/07/30/)

July 30, 2021

nullprogram.com/blog/2021/07/30/

The C [`strcpy`](https://man7.org/linux/man-pages/man3/strcpy.3.html) function is a common sight in typical C programs. It’s also a source of buffer overflow defects, so linters and code reviewers commonly recommend alternatives such as [`strncpy`](https://man7.org/linux/man-pages/man3/strncpy.3.html) (difficult to use correctly; mismatched semantics), [`strlcpy`](https://man.openbsd.org/strlcpy.3) (non-standard, [flawed](https://nrk.neocities.org/articles/not-a-fan-of-strlcpy.html)), or C11’s optional `strcpy_s` ( [no correct or
practical implementations](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n1967.htm)). Besides their individual shortcomings, these answers are incorrect. `strcpy` and friends are, at best, incredibly niche, and the correct replacement is `memcpy`.

If `strcpy` is not easily replaced with `memcpy` then the code is fundamentally wrong. Either it’s not using `strcpy` correctly or it’s doing something dumb and should be rewritten. Highlighting such problems is part of what makes `memcpy` such an effective replacement.

Note: Everything here applies just as much to [`strcat`](https://man7.org/linux/man-pages/man3/strcat.3.html) and friends.

Clarification update: This article is about correctness (objective), not safety (subjective). If the word “safety” comes to mind then you’ve missed the point.

### Common cases

Buffer overflows arise when the destination is smaller than the source. Safe use of `strcpy` requires *a priori* knowledge of the length of the source string length. Usually this knowledge is the exact source string length. If so, `memcpy` is not only a trivial substitute, it’s faster since it will not simultaneously search for a null terminator.

```
char *my_strdup(const char *s)
{
    size_t len = strlen(s) + 1;
    char *c = malloc(len);
    if (c) {
        strcpy(c, s);  // BAD
    }
    return c;
}

char *my_strdup_v2(const char *s)
{
    size_t len = strlen(s) + 1;
    char *c = malloc(len);
    if (c) {
        memcpy(c, s, len);  // GOOD
    }
    return c;
}

```

A more benign case is a static source string, i.e. trusted input.

```
struct err {
    char message[16];
};

void set_oom(struct err *err)
{
    strcpy(err->message, "out of memory");  // BAD
}

```

The size is a compile time constant, so exploit it as such! Even more, a static assertion (C11) can catch mistakes at compile time rather than run time.

```
void set_oom_v2(struct err *err)
{
    static const char oom[] = "out of memory";
    static_assert(sizeof(err->message) >= sizeof(oom));
    memcpy(err->message, oom, sizeof(oom));
}

// Or using a macro:

void set_oom_v3(struct err *err)
{
    #define OOM "out of memory"
    static_assert(sizeof(err->message) >= sizeof(OOM));
    memcpy(err->message, OOM, sizeof(OOM));
}

// Or assignment (implicit memcpy):

void set_oom_v4(struct err *err)
{
    static const struct err oom = {"out of memory"};
    *err = oom;
}

```

This covers the vast majority of cases of already-correct `strcpy`.

### Less common cases

`strcpy` can still be correct without knowing the exact source string length. It is enough to know its *upper bound* does not exceed the destination length. In this example — assuming the input is guaranteed to be null-terminated — this `strcpy` is correct without ever knowing the source string length:

```
struct reply {
    char message[32];
    int x, y;
};

struct log {
    time_t timestamp;
    char message[32];
};

void log_reply(struct log *e, const struct reply *r)
{
    e->timestamp = time(0);
    strcpy(e->message, r->message);
}

```

This is a rare case where `strncpy` has the right semantics. It zeros out unused destination bytes, destroying any previous contents.

```
    strncpy(e->message, r->message, sizeof(e->message));

    // In this case, same as:
    memset(e->message, 0, sizeof(e->message));
    strcpy(e->message, r->message);

```

It’s not a general `strcpy` replacement because `strncpy` might not write a null terminator. If the source string does not null-terminate within the destination length, then neither will destination string.

As before, we can do better with `memcpy`!

```
    static_assert(sizeof(e->message) >= sizeof(r->message));
    memcpy(e->message, r->message, sizeof(r->message));

```

This unconditionally copies 32 bytes. But doesn’t it waste time copying bytes it won’t need? No! On modern hardware it’s far better to copy a large, fixed number of bytes than a small, variable number of bytes. After all, [branching is expensive](/blog/2017/10/06/). Searching for and handling that null terminator has a cost. This fixed-size copy is literally two instructions on x86-64 (output of `clang -march=x86-64-v3 -O3`):

```
vmovups  ymm0, [rsi]
vmovups  [rdi + 8], ymm0

```

It’s faster and there’s no `strcpy` to attract complaints.

### Niche cases

So where *is* `strcpy` useful? Only where all of the following apply:

1. You only know the upper bound of the source string.

2. It’s undesirable to read beyond that length. Maybe storage is limited to the exact length of the string, or the upper bound is very large so an unconditional copy is too expensive.

3. The source string is so long, and the function so hot, that it’s worth avoiding two passes: `strlen` followed by `memcpy`.

These circumstances are very unusual which makes `strcpy` a niche function you probably don’t need. This is the best case I can imagine, and it’s pretty dumb:

```
struct doc {
    unsigned long long id;
    char body[1L<<20];
};

// Create a new document from a buffer.
//
// If body is more than 1MiB, the behavior is undefined.
struct doc *doc_create(const char *body)
{
    struct doc *c = calloc(1, sizeof(*c));
    if (c) {
        c->id = id_gen();
        assert(strlen(body) < sizeof(c->body));
        strcpy(c->body, body);
    }
    return c;
}

```

If you’re dealing with such large null-terminated strings that (2) and (3) apply then you’re already doing something fundamentally wrong and self-contradictory. The pointer and length should be [kept and passed
together](/blog/2019/06/30/). It’s especially essential for a hot function.

```
struct doc_v2 {
    unsigned long long id;
    size_t len;
    char body[];
};

```

### Bonus: `*_s` isn’t helping you

C11 introduced “safe” string functions as an optional “Annex K”, each named with a `_s` suffix to its “unsafe” counterpart. Here is the prototype for `strcpy_s`:

```
errno_t strcpy_s(char *restrict s1,
                 rsize_t s1max,
                 const char *restrict s2);

```

The `rsize_t` is a `size_t` with a “restricted” range ( `RSIZE_MAX`, probably `SIZE_MAX/2`) intended to catch integer underflows. If you [accidentally compute a negative length](/blog/2017/07/19/), it will be a very large number in unsigned form. (An indicator that [`size_t` should have
originally been defined as signed](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1428r0.pdf).) This will be outside the restricted range, and so the operation isn’t attempted due to a likely underflow.

These “safe” functions were modeled after functions of the same name in MSVC. However, as noted, there are no practical implementations of Annex K. The functions in MSVC have different semantics and behavior, and they do not attempt to implement the standard.

Worse, they don’t even do what’s promised in [their documentation](https://docs.microsoft.com/en-us/cpp/c-runtime-library/reference/strcpy-s-wcscpy-s-mbscpy-s?view=msvc-160). The following program should cause a runtime-constraint violation since `-1` is an invalid `rsize_t` in any reasonable implementation:

```
#define __STDC_WANT_LIB_EXT1__ 1
#include
#include

int main(void)
{
    char buf[8] = {0};
    errno_t r = strcpy_s(buf, -1, "hello");
    printf("%d %s\n", (int)r, buf);
}

```

With the latest MSVC as of this writing (VS 2019), this program prints “ `0 hello`”. Using `strcpy_s` did not make my program any safer than had I just used `strcpy`. If anything, it’s *less safe* due to a false sense of security. Don’t use these functions.

- [c](/tags/c/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20strcpy:%20a%20niche%20function%20you%20don't%20need) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=strcpy%3A+a+niche+function+you+don%27t+need).

« [More DLL fun with w64devkit: Go, assembly, and Python](/blog/2021/06/29/)

» [Test cross-architecture without leaving home](/blog/2021/08/21/)
