---
title: Tricking a Whitebox Testcase Generator
url: https://blog.regehr.org/archives/672
published: "2012-01-24T14:26:49Z"
feed: regehr
guid: http://blog.regehr.org/?p=672
---

# Tricking a Whitebox Testcase Generator

The awesome but apparently discontinued [Underhanded C Contest](http://underhanded.xcott.com/) invited us to write C code that looks innocent but acts malicious. Today’s post is an alternate take on the same idea: we don’t really care what the malicious code looks like, but it needs to escape detection by an automated whitebox testcase generator. In some respects this is much easier than tricking a person, since whitebox tools are (at best) idiot savants. On the other hand, a number of tried-and-true methods for tricking humans will be stopped cold by this kind of tool.

As a simple example, we’ll start with two solutions to my [bit puzzle](https://blog.regehr.org/archives/651) from last month: the slow reference implementation and a much faster optimized version. We’ll also include a little test harness.

```
uint64_t high_common_bits_reference(uint64_t a, uint64_t b)
{
  uint64_t mask = 0x8000000000000000LLU;
  uint64_t output = 0;
  int i;
  for (i = 63; i >= 0; i--) {
    if ((a & mask) == (b & mask)) {
      output |= (a & mask);
    } else {
      output |= mask;
      goto out;
    }
    mask >>= 1;
  }
 out:
  return output;
}

uint64_t high_common_bits_portable(uint64_t a, uint64_t b)
{
  uint64_t x = a ^ b;
  x |= x >> 1;
  x |= x >> 2;
  x |= x >> 4;
  x |= x >> 8;
  x |= x >> 16;
  x |= x >> 32;
  return (a & ~x) | (x & ~(x >> 1));
}

int main(void)
{
  srand(time(NULL));
  int i;
  for (i=0; i < 100000000; i++) {
    uint64_t a = rand64();
    uint64_t b = mutate(a);
    assert (high_common_bits_reference (a,b) ==
            high_common_bits_portable (a,b));
  }
  return 0;
}

```

The code in main() is just a trivial fuzz tester: rand64() gives uniformly distributed 64-bit random integers and mutate() randomly flips 0..63 random bits in its argument (making both arguments independently random does not do a very good job testing this code).

Can we add malicious code that this fuzzer will not detect? Of course this is easy, for example we can add this code to the first line of either function:

```
if (a==0x12345678 && b==0x9abcdef0) return -77;

```

Why return -77? Let’s assume that the return value from this function (which is guaranteed to be in the range 0..63) is used as an array index, and offset -77 lets us turn an array write into a stack smash.

First, will our fuzz tester discover this modification? Nope… even with 100 million random trials, the probability of detection is on the order of 10-30.

Next, let’s try to discover the modification using [Klee](http://klee.llvm.org/), an excellent open source whitebox test case generator. We’ll change the test harness to this:

```
int main(void)
{
  uint64_t a,b;
  klee_make_symbolic (&a, sizeof (a), "a");
  klee_make_symbolic (&b, sizeof (b), "b");
  assert (high_common_bits_reference (a,b) == high_common_bits_portable (a,b));
  return 0;
}

```

As expected, Klee discovers the problem almost immediately:

> ```
> KLEE: ERROR: /mnt/z/svn/code/bitdiff-klee.c:129: ASSERTION FAIL: high_common_bits_reference (a,b) == high_common_bits_portable (a,b)
> ```

How did it find these inputs? The point of a whitebox testcase generator is to take advantage of the structure of the code while generating test cases. So here, it simply asked “What values of the arguments lead to the assertion-failed branch being taken?” The answer, in this case, is trivial.

Do tools like Klee pose a serious problem for our effort to insert a backdoor? Let’s try to stop Klee from seeing our bug. The canonical way to fool a whitebox testcase generator is to exploit the fact that these tools solve hard inverse problems — so we insert a computation that is hard to invert. A checksum computation should do nicely. Our bug code now reads:

```
if (obfuscate_crc(a)==0xdf590898 && obfuscate_crc(b)==0x970b5f84) return -77;

```

where:

```
uint64_t obfuscate_crc (uint64_t x)
{
  return crc32 ((unsigned char *)&x, sizeof (x));
}

```

crc32() is simply a standard 32-bit checksum function that my stupid WordPress plugin for rendering C code cannot render, but you can find it in the [full source code for this example](https://blog.regehr.org/extra_files/bitdiff-klee.c).

Now we run Klee again and — damn! — after about 30 minutes (on my slowish home machine) it finds an input that triggers our buggy code. This is impressive; I did not expect Klee to be able to invert a couple of CRC32 operations. While it’s clear that we could push this line of attack further, and eventually confuse Klee to the point where it gives up, let’s try a different tactic. Our new bug code is:

```
 if (obfuscate_pipe(a)==0x12345678 && obfuscate_pipe(b)==0x9abcdef0) return -77;

```

where:

```
uint64_t obfuscate_pipe (uint64_t x)
{
  int pipefd[2];
  int res = pipe (pipefd);
  // assert (res==0);
  res = write (pipefd[1], &x, sizeof(x));
  // assert (res==sizeof(x));
  uint64_t ret;
  res = read (pipefd[0], &ret, sizeof(x));
  // assert (res==sizeof(x));
  close (pipefd[0]);
  close (pipefd[1]);
  return ret;
}

```

This time, rather than trying to overcome Klee using brute force, we’re counting on its inability to follow data through certain system calls. This time we get the desired result: not only does Klee not find anything amiss with our code, but it reports the same number of total paths (65) as it reports for the unmodified code.

A third way to trick Klee is to exploit a weakness in its goal of achieving full path coverage of the system under test. It is not hard to create a code path that is malicious for particular values of variables, but not for other choices. For example, Klee does not see the problem in this code:

```
int indices[] = { 0, 2, 1, 5, 3, 0, -77, 0, 3, 6 };

value_t storage[7];

void stash (int i, value_t v)
{
  if (i<0 || i>=10) {
     error();
  } else {
    storage[indices[i]] = v;
  }
}

```

When Klee tests the path containing the array store, it won’t notice the malicious index unless it happens to generate inputs that result in stash() being invoked with i==6 (it doesn’t, at least for me).

I feel certain there are additional ways to fool this kind of tool and will be interested to hear people’s thoughts.

Note that [Microsoft does massive whitebox testing](http://research.microsoft.com/en-us/um/people/pg/public_psfiles/talk-issta2010.pdf) of its products at the binary level (Klee, in contrast, operates on LLVM bitcodes). Anyone who wants to put a backdoor into these products had better be covering their tracks carefully using techniques like the ones in this post (of course other strategies, such as ensuring that the affected modules are never tested by SAGE, would seem simpler and more reliable).

Finally, I might be able to come up with a new underhanded C contest where the goal is to fool both humans and a whitebox tester; let me know if that sounds interesting.

\[Notes: [Source code is available here](https://blog.regehr.org/extra_files/bitdiff-klee.c). While preparing this piece I used [this self-contained Klee distribution](http://keeda.stanford.edu/~pgbovine/klee-cde-package.v2.tar.bz2) which is super easy to use and is described in a bit more detail on the [Klee “getting started” page](http://klee.llvm.org/GetStarted.html).\]

**Update from Jan 25:**

As arrowdodger points out below, Klee should find the problem with the third method in this post. The fact that I was unable to get it to do so stemmed from a silly mistake I made: giving Klee the “–optimize” flag, which runs LLVM’s optimizers, which deleted the array write as dead code. Thanks to the people on the Klee mailing list for setting me straight!
