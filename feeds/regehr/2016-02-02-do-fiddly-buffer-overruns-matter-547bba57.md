---
title: Do Fiddly Buffer Overruns Matter?
url: https://blog.regehr.org/archives/1294
published: "2016-02-02T07:41:49Z"
feed: regehr
guid: http://blog.regehr.org/?p=1294
---

# Do Fiddly Buffer Overruns Matter?

*by Pascal Cuoq and John Regehr*

Using [tis-interpreter](http://trust-in-soft.com/tis-interpreter/) we found a “fiddly” buffer overrun in OpenSSL: it only goes a few bytes out of bounds and the bytes it writes have just been read from the same locations. Fiddly undefined behaviors are common and it can be hard to convince developers to fix them, especially in a case such as this where the behavior is intentional. More accurately, the behavior was intentional when the code was written in the 1990s–a simpler age of more interesting computers. Nevertheless, we reported this bug and it was recently [fixed in the master branch](https://github.com/openssl/openssl/commit/3e9e810f2e047effb1056211794d2d12ec2b04e7). The [new version of the file is far nicer](https://github.com/openssl/openssl/blob/3e9e810f2e047effb1056211794d2d12ec2b04e7/crypto/rc4/rc4_enc.c). The bug is still present in the release branches.

The bug is in the implementation of [RC4](https://en.wikipedia.org/wiki/RC4): a comparatively old cipher that has had a useful life, but is reaching the end of it. Indeed, OpenSSL’s rc4\_enc.c was — before the recent fix — some seriously 1990s C code with plentiful use of the preprocessor, design choices based on characteristics of Alphas and UltraSPARCs, and this dodgy OOB-based performance hack. A new paper “ [Analysing and Exploiting the Mantin Biases in RC4](https://eprint.iacr.org/2016/063.pdf)” might be the final nail in the coffin of RC4.

The encryption code in rc4\_enc.c causes up to `RC4_CHUNK - 1` bytes past the end of the output buffer to be overwritten with values that have been read from the same (out of bounds) locations. There are several variants delineated with #ifdefs. On an ordinary x86-64, RC4\_CHUNK is 8 and execution will go through [line 217](https://github.com/openssl/openssl/blob/349807608f31b20af01a342d0072bb92e0b036e2/crypto/rc4/rc4_enc.c#L217-L232):

```
for (; len & (0 - sizeof(RC4_CHUNK)); len -= sizeof(RC4_CHUNK)) {
    ichunk = *(RC4_CHUNK *) indata;
    otp = ...
    *(RC4_CHUNK *) outdata = otp ^ ichunk;
    indata += sizeof(RC4_CHUNK);
    outdata += sizeof(RC4_CHUNK);
}

```

This loop consumes all the complete 8-byte words found in the input buffer, and writes as many words into the output buffer. So far so good. Things get interesting [after the loop](https://github.com/openssl/openssl/blob/349807608f31b20af01a342d0072bb92e0b036e2/crypto/rc4/rc4_enc.c#L233-L264):

```
if (len) {
    RC4_CHUNK mask = (RC4_CHUNK) - 1, ochunk;

    ichunk = *(RC4_CHUNK *) indata;  // !
    ochunk = *(RC4_CHUNK *) outdata; // !
    ...
    mask >>= (sizeof(RC4_CHUNK) - len) << 3;

    ... something about ultrix ...

    ochunk &= ~mask;
    ochunk |= (otp ^ ichunk) & mask;
    *(RC4_CHUNK *) outdata = ochunk; // !!

```

If there remain between one and seven unprocessed bytes, these are taken care of by reading an entire 8-byte word from indata, reading an entire word from outdata, computing a new word made from encrypted input and original out-of-bound values, and writing that word to outdata.

What could go wrong? At first glance it seems like this overrun could trigger an unwanted page fault, but that isn’t the case on architectures where an aligned word never crosses a page boundary. However, in a concurrent environment, the illegal writing of illegally read data can be directly observed. We wrote a small program that shows this. The key is a struct that places important data directly after a buffer that is the destination of RC4-encrypted data:

```
struct stuff {
    unsigned char data[data_len];
    int important;
};

```

Then, one thread repeatedly puts RC4-encrypted data into the buffer:

```
void *thread2(void *arg) {
    RC4_KEY key;
    const char *key_data = "Hello there.";
    RC4_set_key(&key, strlen(key_data), (const unsigned char *)key_data);
    while (!stop)
        RC4(&key, data_len, src->data, dst->data);
    return 0;
}

```

While the other thread waits for the important field to be corrupted:

```
void *thread1(void *arg) {
    int x = 0;
    while (!stop) {
        dst->important = ++x;
        check(&dst->important, x);
    }
    return 0;
}

```

The check() function is compiled separately to help ensure that the value being checked comes from RAM rather than being cached in a register:

```
void check(int *addr, int val) { assert(*addr == val); }

```

Our [complete code is on github](https://github.com/regehr/rc4-poc). If you run it, you should see the assertion being violated almost immediately. We’ve tested against OpenSSL 1.0.2f on Linux and OS X. On Linux you must make sure to get the C implementation of RC4 instead of the assembly one:

```
./Configure no-asm linux-x86_64
```

Although any of tis-interpreter, Valgrind, or ASan can find the OpenSSL RC4 bug when the encrypted data bumps up against the end of an allocated block of memory, none of them finds the bug here since the “important” field absorbs the OOB accesses. There’s still an UB but it’s subtle: accessing buffer memory via a pointer-to-long is a violation of C’s strict aliasing rules.

So, do fiddly buffer overruns matter? Well, this particular bug is unlikely to have an observable effect on programs written in good faith, though one of us considered submitting as an [Underhanded Crypto Challenge](https://underhandedcrypto.com/about/) entry a backdoored chat program with end-to-end encryption that, each time the race condition triggers, sends a copy of the private message being processed to Eve, encrypted with Eve’s key. Alas, the race window is very short. A fiddly OOB can also be a problem if the compiler notices it. In [an example from Raymond Chen’s blog](https://blogs.msdn.microsoft.com/oldnewthing/20140627-00/?p=633), GCC uses the OOB array reference as an excuse to compile this function to “return true”:

```
int table[4];
bool exists_in_table(int v) {
    for (int i = 0; i <= 4; i++) {
        if (table[i] == v) return true;
    }
    return false;
}

```

That could only happen here if we used link-time optimization to give the compiler access to both the application and the OpenSSL code at once.
