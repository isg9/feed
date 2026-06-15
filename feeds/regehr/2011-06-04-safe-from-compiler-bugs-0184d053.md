---
title: Safe From Compiler Bugs?
url: https://blog.regehr.org/archives/539
published: "2011-06-04T04:45:45Z"
feed: regehr
guid: http://blog.regehr.org/?p=539
---

# Safe From Compiler Bugs?

A few people have asked me: Does there exist a subset of the C language that is not, in practice, miscompiled? The intuition behind the question is perfectly reasonable. First, it is clear that there exist C features, such as bitfields and volatile variables, whose compiler support is not so reliable. Second, there exist C subsets like [MISRA C](http://www.misra-c.com/Activities/MISRAC/tabid/160/Default.aspx) that are intended for safety critical application development and we would hope they can be reliably compiled.

There probably do exist subsets of C that avoid compiler bugs. For example, if we avoid all math, control flow, and I/O, then it’s at least conceivable that we’re on safe ground. If not, then almost certainly we’d be OK by permitting only “return 0;” in function bodies. However, my group’s experience in reporting compiler bugs has convinced me that there is no remotely useful subset of C that reliably avoids compiler bugs. Let’s take this C subset as an example:

- only variables of type int (or unsigned int, if that seems better)
- no dynamic memory allocation
- only functions of 30 lines or less
- only if/else statements, function calls, and simple for loops for flow control (no break, continue, do/while, goto, longjmp, etc.)
- only single-level structs, single-level pointers, and one-dimensional arrays
- no type qualifiers
- all expressions are in three-address form; for example “x = y + z;”

We could add more restrictions, but that would seem to make it hard to get work done in the subset. Of course, to make the subset real we’d need to nail down a lot of additional details and also write a checking tool (neither activity would be difficult).

Next we ask the question: what percentage of the compiler’s optimization passes will have useful work to do when compiling code written in this subset? The answer, unfortunately, is “most of them.” For an aggressive compiler, this amounts to a lot of extremely subtle code. In practice, some of it will be wrong. Thus, I am claiming that even a fairly severe C subset provides little shelter from compiler bugs.

The best way to back up my claim would be to rewrite some large C codes in my subset and show that they are still miscompiled. Unfortunately that is too much work. Rather, I’ll point to a few compiler bugs that we’ve found which, while they are not generally triggered by programs in the subset I outlined above, are triggered by pretty simple test cases. I would argue that — taken together — these effectively kill the idea of a bug-safe subset. Keep in mind that most of these test cases are not fully reduced; in some cases a compiler developer was able to do considerably better (GCC hacker Jakub Jelinek is most amazing test case reducer I know of). Also, my guess is that most of these bugs could be tickled by programs in a smaller subset of C. For example, test cases for bugs that we report often use the volatile qualifier not because it has anything to do with the bug, but rather because volatile objects serve as a convenient mechanism for suppressing optimizations that would otherwise mask the bug.

[GCC bug 42952](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=42952) results in this code being miscompiled:

> ```
> static int g_16[1];
> static int *g_135 = &g_16[0];
> static int *l_15 = &g_16[0];
> ```
>
> ```
> static void foo (void) {
>   g_16[0] = 1;
>   *g_135 = 0;
>   *g_135 = *l_15;
>   printf("%d\n", g_16[0]);
> }
> ```

[**G** CC bug 42512](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=42512) results in this code being miscompiled:

> ```
> int g_3;
> ```
>
> ```
> int main (void) {
>   long long l_2;
>   for (l_2 = -1; l_2 != 0; l_2 = (unsigned char)(l_2 - 1)) {
>     g_3 |= l_2;
>   }
>   printf("g_3 = %d\n", g_3);
>   return 0;
> }
> ```

[GCC bug 41497](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=41497) results in this code being miscompiled:

> ```
> static uint16_t add (uint16_t ui1, uint16_t ui2) {
>   return ui1 + ui2;
> }
>
> uint32_t g_108;
> uint8_t f3;
> uint8_t f0;
>
> void func_1 (void) {
>   for (f3 = 0; f3 <= 0; f3 = 1) {
>     for (g_108 = -13; g_108 == 0; g_108 = add (g_108, 0)) {
>       f0 = 1;
>     }
>   }
> }
> ```

[LLVM bug 2487](http://llvm.org/bugs/show_bug.cgi?id=2487) results in this code being miscompiled:

> ```
> int g_6;
> void func_1 (void) {
>   char l_3;
>   for (l_3 = 0; l_3 >= 0; ++l_3) {
>     if (!l_3) {
>       for (g_6 = 1; 0; --g_6);
>     }
>   }
> }
> ```

[LLVM bug 2716](http://llvm.org/bugs/show_bug.cgi?id=2716) results in this code being miscompiled:

> ```
> int func_3 (void) {
>   long long p_5 = 0;
>   signed char g_323 = 1;
>   return (1 > 0x14E7A1AFC6B86DBELL) <= (p_5 - g_323);
> }
> ```

[LLVM bug 3115](http://llvm.org/bugs/show_bug.cgi?id=3115) results in this code being miscompiled:

> ```
>
> unsigned int g_122;
>
> int func_1 (void) {
>   unsigned int l_19 = 1;
>   if (1 ^ l_19 && 1) return 0;
>   return 1;
> }
> ```

I could go on, but six examples should suffice.

A better question than “Is there a bug-safe C subset?” might be “Is there a subset of C that, when combined with restrictions on the optimization passes used, is safe from compiler bugs?” I don’t know the answer to this, but I do know that disabling optimizations is probably the only reliable way to keep large, possibly-buggy parts of the compiler from executing. Also, organizations creating safety critical systems often limit the amount of optimization that is performed by compilers they use (though that is probably as much to give traceable, debuggable code as it is to avoid miscompilations). In the medium to long run, an even better idea would be to forget about artificial constraints on the language and instead use a verified compiler or a tool such as one of the emerging [translation validators for LLVM](http://dash.harvard.edu/bitstream/handle/1/4762396/pldi84-tristan.pdf).
