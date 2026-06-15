---
title: Learning When Values are Changed by Implicit Integer Casts
url: https://blog.regehr.org/archives/1633
published: "2018-11-02T23:27:05Z"
feed: regehr
guid: http://blog.regehr.org/?p=1633
---

# Learning When Values are Changed by Implicit Integer Casts

C and C++ perform implicit casts when, for example, you pass an integer-typed variable to a function that expects a different type. When the target type is wider, there’s no problem, but when the target type is narrower or when it is the same size and the other signedness, integer values may silently change when the type changes. For example, this program:

```
#include <iostream>

int main() {
  int x = -3;
  unsigned y = x;
  std::cout << y << "\n";
}

```

prints 4294967293. Like unsigned integer wraparound (and unlike signed integer overflow) these changes of value are not undefined behavior, but they may be unintentional and may also be bugs. As of recently, Clang contains support for dynamically detecting value changes and either providing a diagnostic or else terminating the program. [Some fine-grained flags are available](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html#available-checks) for controlling these diagnostics, but you can enable all of them (plus some others, such as signed overflow and unsigned wraparound checks) using `-fsanitize=integer`.

Now we get:

```
$ clang++ implicit2.cpp -fsanitize=integer
$ ./a.out
implicit2.cpp:5:16: runtime error: implicit conversion from type 'int' of value -3 (32-bit, signed) to type 'unsigned int' changed the value to 4294967293 (32-bit, unsigned)
4294967293
$

```

Here’s a truncation example:

```
int main() {
  int x = 300;
  unsigned char c = x;
}

```

It gives:

```
$ clang++ implicit3.cpp -fsanitize=integer
$ ./a.out
implicit3.cpp:3:21: runtime error: implicit conversion from type 'int' of value 300 (32-bit, signed) to type 'unsigned char' changed the value to 44 (8-bit, unsigned)
$

```

To suppress the diagnostic, we can make the conversion explicit using a cast:

```
int main() {
  int x = 300;
  unsigned char c = (unsigned char)x;
}

```

Different parts of this functionality landed in Clang before and after the 7.0.0 release — to get everything, you will want to build a Clang+LLVM from later than 1 Nov 2018 (or else wait for the 8.0.0 release in a few months).

How would you use these checks? Generally, they should be part of a testing campaign, perhaps in support of a code audit. If you enable them on a non-trivial code base, you will run into diagnostics that do not correspond to bugs, just because real C and C++ programs mix integer types so freely. I suggest starting with `-fsanitize=implicit-integer-truncation`. Let’s look at doing this to Clang+LLVM itself and then compiling a hello world program: [here’s the output](https://blog.regehr.org/extra_files/llvm-integer-truncations.txt).

I’d be happy to hear about any interesting bugs located using these checks, if anyone wants to share.

Finally, see [this tweet by Roman Lebedev](https://twitter.com/LebedevRI/status/1058782696942567425) (the author of these checks) showing that the runtime impact of `-fsanitize-trap=implicit-conversion` is very low.
