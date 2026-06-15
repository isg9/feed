---
title: Unicode operators for&nbsp;C
url: https://gustedt.wordpress.com/2017/03/09/unicode-operators-for-c/
published: "2017-03-09T16:54:28Z"
feed: gustedt
guid: https://gustedt.wordpress.com/?p=2668
---

# Unicode operators for&nbsp;C

C11 has added a certain level of Unicode support to C, but I think for C2x it will be time to go a step further and put C in line with general usage of special characters as they are normalized by Unicode. In particular, it is time to get rid of restrictions in operator naming that stem from the limited availability of special characters 30 years ago, when all of this was invented.

Let us first revise the different levels of support that the C language provides nowadays. I really mean the core language, not library interfaces such as `locale`, for example. Neglecting some subtleties such as Endianess, for example, this gives:

1. Some punctuator characters that might not be present on a keyboard can be represented by *trigraphs.* For example, the sequence `??/` can be used for a `\` character. This replacement is done regardless of the lexical context.
2. Unicode code points can be entered as `\uXXXX` or `\UXXXXXXXX` where the `X` represent the short or long form of the code point. For example, the code `\u030C` corresponds to the Greek letter π. This replacement is done regardless of the lexical context.
3. Some operators have another spelling as *digraphs.* For example, the token `<:` is equivalent to the token `[`. This replacement is done on a syntax level, after tokenization. So it only occurs, where the corresponding operators may occur. It is not performed inside strings.
4. Wide character characters and strings can be used to deal with special characters internally in a C program. For example, the wide character `L'\u030C'` should result in a single entity of type `wchar_t` containing the character π expressed in the encoding of the execution environment.
5. Prefixes `u` and `U` introduce UTF-16 (or UCS-2) and UTF-32 characters and strings. They can thereby be used to ensure a specific encoding of characters.  For example `u'π'` should have the character π represented in the UTF-16 encoding, that is the value `0x030C`.
6. UTF-8 encoding of *strings* can be enforced with the special prefix `u8`. For example, `u8"π"` is an array of three `char` with contents `0x80 0xCF 0x00`.

Points 1. to 4. are independent of any *encoding*  in which we may write our source file. Using 4. ensures the transition from the source encoding to the execution environment, which may in fact be different.

Points 5. and 6. where added by C11 to ensure interoperability  such that we know which encoding is used for a particular character or  string.  Note that this implies in particular that a conforming compiler implements conversions between different encodings:

- the source encoding
- the execution environment encoding
- UTF-16 (or UCS-2), UTF-32 and UTF-8.

For example, even if your platform has some weird historic source encoding but which includes the character ö, the strings `u"söng"`, `U"söng"` and `u8"söng"` have a well-defined contents that is independent of the execution environment.

Now let us have a look at the user side. Here is my wish list for a comfortable C coding experience:

1. Arbitrary code points in strings and comments.
2. Arbitrary code points in identifiers, within the allowed range demanded by the C standard.
3. Source ( *native)* encoding of operator characters. This should be provided for operators where C still has multi-character tokens e.g `≤` for `<=`, `≥` for `>=`, `∧` for `&&`, or where a character is used with an non-standard semantic, e.g `×` instead of `*` for multiplication.
4. Wide character encoding support for sources.
5. UTF-8 source code support.

Sometimes 1. is *“implemented by negligence”* because UTF-8 and similar encodings can just be used as such and so the compiler just takes the bytes and stuffs them into strings. Subtle bugs may be triggered by this, such as character arrays being longer than expected or UTF-8 sequences cut in half, in particular if the library support for these things is incomplete.

Generally, points 1. and 2. are guaranteed by the long list above, but in a very weird form, namely by using `\uXXXX` encoding all over the place. If you have to write C code with strings in a human language other than English, this is effectively not usable.  To maintain strings as `L"\u0395\u1f50\u03ba\u03bb\u03b5\u03af\u03b4\u03b7\u03c2"` for Εὐκλείδης must be a nightmare.

Point 2. has the same disadvantages, but becomes even weirder when seen alone. Even if the `\uXXXX` encoding allows to have the code point for π as an identifier, if we have to write  `\u030C` such a specification serves no real purpose.

In my personal view, supporting real Unicode identifiers in C code is important. First of all, who are we westerners to impose the usage of the overly restricted Latin alphabet to the rest of the world? Aren’t we a bit arrogant? Then, my personal usage for these things is not for German or French, but for mathematics.  Mathematics often uses Greek letters, some special characters etc to express complicated facts, and I want to use these in my daily writing.

C code should map as closely as possible to different national or professional cultures.   Everything else is simply not acceptable in the 21st century.

Unfortunately, many compilers are stuck and don’t go beyond point 2. This is even so, as we have seen above, all the components to deal with different encodings must be present in a C11 compiler. Providers of such compilers really should get their objectives straight and think just a tiny little bit in terms of usability and comfort for their clients. As we will see below, implementing 3., 4. and 5. is a piece of cake. Basically something that is doable by an intern.

Since 5. implies 4., let us concentrate on 5. that is UTF-8 source code encoding. Other encodings should be similarly easy. The “ *only*” thing that is needed to implement UTF-8 source encoding for a conforming C11 compiler is

- `tools-dump8` a tool to transform UTF-8 multibyte characters into `\uXXXX` or `\UXXXXXXX` notation.

*[Modular C](http://cmod.gforge.inria.fr/doxygen/index.html)* offers a simple tool that does just this. This is a stand-alone utility programmed in about 100 lines of C code. You can simply integrate it in your tool-chain as a source-to-source pre-compiler. It is fast and easy, there is no excuse not to integrate such a tool in your compiler tool-chain.

So now that we have settled 4. and 5. let’s have a look into 3. C abuses a lot of weird characters and character combinations that have clear descriptions as code points in Unicode. They have satisfactory support on all platforms, text processing, fonts etc. Being stuck in the character restrictions of ancient machines that are out of order since 30 years is not acceptable, either. Let’s put up a list:

- `\u00AC`, glyph ¬, operator `!`
- `\u00D7`, glyph ×, operator `*`, binary only
- `\u00F7`, glyph ÷, operator `/`, binary only
- `\u2026`, glyph …, operator `...`
- `\u2192`, glyph →, operator `->`
- `\u2227`, glyph ∧, operator `&&`
- `\u2228`, glyph ∨, operator `||`
- `\u2229`, glyph ∩, operator `&`, binary only
- `\u222A`, glyph ∪, operator `|`, binary only
- `\u2254`, glyph ≔, operator `=`, assignment
- `\u2A74`, glyph ⩴, operator `=`, initialization
- `\u2260`, glyph ≠, operator `!=`
- `\u2261`, glyph ≡, operator `==`
- `\u2264`, glyph ≤, operator `<=`
- `\u2265`, glyph ≥, operator `>=`
- `\u2AA1`, glyph ⪡, operator `<<`
- `\u2AA2`, glyph ⪢, operator `>>`

Some points could probably be discussed in detail, but I hope the idea is clear. There is a well established character for “less or equal” namely ≤, let’s simply use this.

Again, implementing this in a compiler tool-chain is quite easy. *Modular C* does this with a multi-stage process (hiding characters and strings, decoding, unhiding of characters and strings), but you could certainly come up with an appropriate way to do this directly in your tool.

Integrating these operators in the standard would also not be very difficult, either. They can just be added as “punctuators” (6.4.6) . Then the replaced tokens could be added to p3 of that section as “digraphs”. E.g we could simply add `||` to the list of digraphs and say that its meaning is `∨`.
