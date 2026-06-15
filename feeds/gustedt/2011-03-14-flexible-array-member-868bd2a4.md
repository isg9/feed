---
title: flexible array member
url: https://gustedt.wordpress.com/2011/03/14/flexible-array-member/
published: "2011-03-14T22:16:01Z"
feed: gustedt
guid: http://gustedt.wordpress.com/?p=994
---

# flexible array member

C99 allows to define a *flexible array member* as the last member of a `struct`, namely an array of undetermined length.

```
P99_DECLARE_STRUCT(package_head);
struct package_head {
   char name[20];
   size_t len;
   uint64_t data[];
};

```

Such a `struct` can then allocated on the heap with a suitable size such that the field data has as much elements as fit in the allocated space from the start of data onward. Usually one would allocate such `struct` with

```
package_head *a = malloc(sizeof(package_head) + 10 * sizeof(uint64_t));
package_head *b = malloc(sizeof(*b) + 12 * sizeof(b->data[0]));

```

This has several disadvantages. First the syntax is clumsy, we have to use a relatively complicated expression that uses two elements of the specification of `a` or `b`.

Then this is wasting space. Due to packing of the `struct` the offset of data “inside” the `struct` may be less than `sizeof(package_head)`. In most cases the real size of the object that we want to build is

```
offsetof(package_head, data) + N * sizeof(uint64_t)

```

so we are wasting

```
sizeof(package_head) - offsetof(package_head, data)

```

bytes.

The above formula for the exact size is only valid for larger values of `N`. We must also make sure that we allocate at least `sizeof(package_head)` bytes. So the complete formula for looks something like

```
P99_MAXOF(sizeof(T), offsetof(T, F) + P99_SIZEOF(T, F[0]) * N)

```

which is probably not something that you want to write on a daily base. A particularity in that expression is `P99_SIZEOF(T, F[0])` which stands for the size of the element `F[0]` inside the `struct` type `T`. C doesn’t have the possibility as C++ to refer to a field in a type with something like `T::F`.

Something similar can be obtained in C99 with the magic formula `sizeof((T){ 0 }.F[0]) `: define a compound literal `(T){ 0 }` of type `T` and take its field `F`. The `sizeof` operator actually ensures that this compound literal is never allocated, only the field `F` is taken for its type and size. This magic works in function scope (the compound literal would be of storage class `auto`) and in file scope (it would be `static`).

P99 provides [several interfaces](http://p99.gforge.inria.fr/p99-html/group__flexible.html) to allocate `struct` with flexible members: `P99_FCALLOC`, `P99_FMALLOC` and `P99_FREALLOC`.
