---
title: <code>struct</code> tags in C++ are even&nbsp;weirder
url: https://gustedt.wordpress.com/2012/04/20/struct-tags-in-c-are-even-weirder/
published: "2012-04-20T22:14:02Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=1616
---

# <code>struct</code> tags in C++ are even&nbsp;weirder

I already discussed that fact that [struct tags are not identifiers](https://gustedt.wordpress.com/2010/09/12/struct-tags-are-not-identifiers-in-c/) in C++, and in particular that a tag name can be used as the name of a types. Today I learned that the rules for that are even more complicated than I thought. In C an identifier that has been used as a tag (for `struct`, `union` or `enum`) can freely be used as another identifier (variable, `typedef`, label). In C++ only **almost**: there is one sort of identifiers it can’t be used for, `typedef` **unless** it refers to the same type as the tag.

So the rules in C++ are somehow

- `struct toto;` declares a type that can be accessed through the name toto (without struct keyword) if this is unambiguous.
- It can also be used to declare another kind of object, and then to get access to the type we’d have to prefix the name with the keyword `struct`.
- The other kind of object may be anything, but it can’t be a type…
- … unless it refers to exactly the same type as before.

This is the standard example where everything works fine, a tag name an a `typedef` with the same identifier `clean` that refers to the same type:

```
struct clean {
  unsigned a;
};
// A typedef that binds the same type to the name clean as well
// legal for C
// legal for C++
typedef struct clean clean;

```

If we don’t have the `typedef`, in C we can’t use the tag name as a type without the prefix `struct`:

```
struct C {
  unsigned a;
};
// A function that returns the struct we just defined
// legal for C++
// illegal for C, we need the struct before the first "C"
C C(void);

```

But if we define a new type with the tag name, for C++ the type has to be the same and the following (awful) hack doesn’t work:

```
struct CPP {
  unsigned a;
};
// A typedef that binds a different type to the name CPP
// legal for C
// illegal for C++, refers to a different type
typedef struct CPP * CPP;

```

So if we put the last two blobs together in one source file, we have something that doesn’t compile, neither for C or C++. But for completely different reasons. One errors out on the “C” line, the other one on the “CPP” line.

My personal taste tends to converge towards finding the three-fold exception list of C++ just ugly. For me this clearly shows the tendency of complisficating the language. With all the best intentions…
