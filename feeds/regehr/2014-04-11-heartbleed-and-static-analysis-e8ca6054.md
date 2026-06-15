---
title: Heartbleed and Static Analysis
url: https://blog.regehr.org/archives/1125
published: "2014-04-11T04:58:13Z"
feed: regehr
guid: http://blog.regehr.org/?p=1125
---

# Heartbleed and Static Analysis

Today in my Writing Solid Code class we went through some of the 151 defects that [Coverity Scan](https://scan.coverity.com/) reports for OpenSSL. I can’t link to these results but take my word for it that they are a pleasure to read — the interface clearly explains each flaw and the reasoning that leads up to it, even across multiple function calls. Some of the problems were slightly alarming but we didn’t see anything that looks like the next heartbleed. Unfortunately, Coverity did not find the heartbleed bug itself. ( **UPDATE:** See [this followup post](https://blog.regehr.org/archives/1128).) This is puzzling; here’s a bit of speculation about what might be going on. There are basically two ways to find heartbleed using static analysis (here I’ll assume you’re familiar with the bug; if not, [this post](http://blog.cryptographyengineering.com/2014/04/attack-of-week-openssl-heartbleed.html) is useful). First, a taint analysis should be able to observe that two bytes from an untrusted source find their way into the length argument of a memcpy() call. This is clearly undesirable. The Coverity documentation indicates that it taints the buffer stored to by a read() system call (unfortunately you will need to login to Coverity Scan before you can [see this](https://scan.coverity.com/models#N4560C)). So why don’t we get a defect report? One guess is that since the data buffer is behind two pointer dereferences (the access is via `s->s3->rrec.data`), the scanner’s alias analysis loses track of what is going on. Another guess is that two bytes of tainted data are not enough to trigger an alarm. Only someone familiar with the Coverity implementation can say for sure what is going on — the tool is highly sophisticated not just in its static analysis but also in its defect ranking system.

The other kind of static analysis that would find heartbleed is one that insists that the length argument to any memcpy() call does not exceed the size of either the source or destination buffer. [Frama-C in value analysis mode](http://frama-c.com/value.html) is a tool that can do this. It is sound, meaning that it will not stop complaining until it can prove that certain defect classes are not present, and as such it requires far more handholding than does Coverity, which is designed to unsoundly analyze [huge quantities of code](http://cacm.acm.org/magazines/2010/2/69354-a-few-billion-lines-of-code-later/fulltext). To use Frama-C, we would make sure that its own header files are included instead of the system headers. In one of those files we would find a model for memcpy():

```
/*@ requires \valid(((char*)dest)+(0..n - 1));
  @ requires \valid_read(((char*)src)+(0..n - 1));
  @ requires \separated(((char *)dest)+(0..n-1),((char *)src)+(0..n-1));
  @ assigns ((char*)dest)[0..n - 1] \from ((char*)src)[0..n-1];
  @ assigns \result \from dest;
  @ ensures memcmp((char*)dest,(char*)src,n) == 0;
  @ ensures \result == dest;
  @*/
extern void *memcpy(void *restrict dest,
                    const void *restrict src,
                    size_t n);

```

The comments are consumed by Frama-C. Basically they say that src and dest are pointers to valid storage of at least the required size, that the buffers do not overlap (recall that memcpy() has undefined behavior when called with overlapping regions), that it moves data in the proper direction, that the return value is dest, and that a subsequent memcmp() of the two regions will return zero.

The Frama-C value analyzer tracks an integer variable using an *interval*: a representation of the smallest and largest value that the integer could contain at some program point. Upon reaching the [problematic memcpy() call in t1\_lib.c](http://git.openssl.org/gitweb/?p=openssl.git;a=blob;f=ssl/t1_lib.c;h=33afdeba33e5def24274e8d0ca39c5e18809bfc6;hb=0d8776344cd7965fadd870e361503985df9c45bb#l2586), the value of payload is in the interval \[0..65535\]. This interval comes from the n2s() macro which turns two arbitrary-valued bytes from the client into an unsigned int:

```
#define n2s(c,s)        ((s=(((unsigned int)(c[0]))<< 8)| \
                            (((unsigned int)(c[1]))    )),c+=2)

```

The dest argument of this memcpy() turns out to be large enough. However, the source buffer is way too small. Frama-C would gripe about this and it would not shut up until the bug was really fixed.

How much effort would be required to push OpenSSL through Frama-C? I don’t know, but wouldn’t plan on getting this done in a week or three. Interestingly, a company spun off by Frama-C developers has recently [used the tool to create a version of PolarSSL](http://trust-in-soft.com/no-more-heartbleed/) that they promise is immune to CWEs [119](http://cwe.mitre.org/data/definitions/119.html), [120](http://cwe.mitre.org/data/definitions/120.html), [121](http://cwe.mitre.org/data/definitions/121.html), [122](http://cwe.mitre.org/data/definitions/122.html), [123](http://cwe.mitre.org/data/definitions/123.html), [124](http://cwe.mitre.org/data/definitions/124.html), [125](http://cwe.mitre.org/data/definitions/125.html), [126](http://cwe.mitre.org/data/definitions/126.html), [127](http://cwe.mitre.org/data/definitions/127.html), [369](http://cwe.mitre.org/data/definitions/369.html), [415](http://cwe.mitre.org/data/definitions/415.html), [416](http://cwe.mitre.org/data/definitions/416.html), [457](http://cwe.mitre.org/data/definitions/457.html), [562](http://cwe.mitre.org/data/definitions/562.html), and [690](http://cwe.mitre.org/data/definitions/690.html). I think it would be reasonable for the open source security community to start thinking more seriously about what this kind of tool can do for us.

**UPDATES:**

- In a comment below, Masklinn states that OpenSSL’s custom allocators would defeat the detection of the too-large argument to memcpy(). This is indeed a danger. To avoid it, as part of applying Frama-C to OpenSSL, the custom malloc/free functions would be marked as being malloc-like using the “allocates” and “frees” keywords supported by [ACSL 1.8](http://frama-c.com/download/acsl_1.8.pdf). Coverity lets you do the same thing, and so would any competent analyzer for C. Custom allocators are regrettably common in oldish C code.

- I’m interested to see what other tools have to say about heartbleed. If you have a link to results, please put it in a comment.

- I re-ran Coverity after disabling [OpenSSL’s custom freelist](http://www.tedunangst.com/flak/post/analysis-of-openssl-freelist-reuse) and also hacking CRYPTO\_malloc() and friends to just directly call the obvious function from the malloc family. This caused Coverity to report 173 new defects: mostly use-after-free and resource leaks. Heartbleed wasn’t in the list, however, so I stand by my guess (above) that perhaps something related to indirection caused this defect to not be ranked highly enough to be reported.

- HN has some discussion [of PolarSSL](https://news.ycombinator.com/item?id=7572465) and of [this blog post](https://news.ycombinator.com/item?id=7571506). Also [Reddit](http://www.reddit.com/r/programming/comments/22ri2i/heartbleed_wasnt_found_by_static_analysis/).
