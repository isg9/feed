---
title: Testcase Reduction for Non-Preprocessed C and C++
url: https://blog.regehr.org/archives/1278
published: "2015-11-17T21:20:37Z"
feed: regehr
guid: http://blog.regehr.org/?p=1278
---

# Testcase Reduction for Non-Preprocessed C and C++

C-Reduce takes a C or C++ file that triggers a bug in a compiler (or other tool that processes source code) and turns it into the smallest possible test case that still triggers the bug. Most often, we try to reduce code that has already been preprocessed. This post is about how to reduce non-preprocessed code, which is sometimes necessary when — due to use of an integrated preprocessor — the compiler bug goes away when a separate preprocessing step is used.

The first thing we need to do is get all of the necessary header files into one place. This is somewhat painful due to things like [computed includes](https://gcc.gnu.org/onlinedocs/cpp/Computed-Includes.html) and `#include_next`. I wrote [a script](https://github.com/csmith-project/creduce/blob/master/scripts/localize_headers) that follows the transitive includes, renaming files and copying them over; it works fine on Linux but sometimes fails on OS X, I haven’t figured out why yet. Trust me that you do not want to look too closely at the Boost headers.

Second, we need to reduce multiple files at once, since they have not yet been glommed together by the preprocessor. C-Reduce, which is a collection of passes that iterate to fixpoint, does this by running each pass over each file before proceeding to the next pass. The seems to work well. A side benefit of implementing multi-file reduction is that it has other uses such as reducing programs that trigger link-time optimization bugs, which inherently requires multiple files.

Non-preprocessed code contains comments, so C-Reduce has a special pass for stripping those out. We don’t want to do this before running C-Reduce because removing comments might make the bug we’re chasing go away. Another pass specifically removes #include directives which tend to be deeply nested in some C++ libraries.

#ifdef … #endif pairs are hard to eliminate from first principles because they are often not located near to each other in the file being reduced, but you still need to eliminate both at once. At first this sounded like a hard problem to solve but then I found Tony Finch’s excellent [unifdef](http://dotat.at/prog/unifdef/) tool and wrote a C-Reduce pass that simply calls it for every relevant preprocessor symbol.

Finally, it is often the case that a collection of reduced header files contains long chains of trivial #includes. C-Reduce fixes these with a pass that replaces an #include with the included text when the included file is very small.

What’s left to do? The only obvious thing on my list is selectively evaluating the substitutions suggested by #define directives. I will probably only do this by shelling out to an external tool, should someone happen to write it.

In summary, reducing non-preprocessed code is not too hard, but some specific support is required in order to do a good job of it. If you have a testcase reduction problem that requires multi-file reduction or needs to run on non-preprocessed code, please try out [C-Reduce](https://github.com/csmith-project/creduce) and let us know — at the [creduce-dev mailing list](http://www.flux.utah.edu/mailman/listinfo/creduce-dev) — how it worked. The features described in this post aren’t yet in a release, just build and install our master branch.

As a bonus, since you’re dying to know, I’ll show you what C-Reduce thinks is the minimal hello world program in C++11. From 127 header files + the original source file, it creates 126 empty files plus this hello.cpp:

```
#include "ostream"
namespace std {
basic_ostream cout;
ios_base::Init x0;
}
int main() { std::cout << "Hello"; }

```

And this ostream:

```
namespace
{
typedef __SIZE_TYPE__ x0;
typedef __PTRDIFF_TYPE__ x1;
}
namespace std
{
template < class > struct char_traits;
}
typedef x1 x2;
namespace std
{
template < typename x3, typename = char_traits < x3 > >struct basic_ostream;
template <> struct char_traits
{
    typedef char x4;
    static x0 x5 ( const x4 * x6 )
    {
        return __builtin_strlen ( x6 );
    }
};
template < typename x3, typename x7 > basic_ostream < x3,
         x7 > &__ostream_insert ( basic_ostream < x3, x7 > &, const x3 *, x2 );
struct ios_base
{
    struct Init
    {
        Init (  );
    };
};
template < typename, typename > struct basic_ostream
{
};
template < typename x3,
         typename x7 > basic_ostream < x3 > operator<< ( basic_ostream < x3,
                 x7 > &x8, const x3 * x6 )
{
    __ostream_insert ( x8, x6, x7::x5 ( x6 ) );
    return x8;
}
}

```

It's obviously not minimal but believe me we have already put a lot of work into domain-specific C++ reduction tricks. If you see something here that can be gotten rid of and want to take a crack at teaching our clang\_delta tool to do the transformation, we'll take a pull request.
