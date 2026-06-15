---
title: Non-Transparent Memory Safety
url: https://blog.regehr.org/archives/1175
published: "2014-08-08T03:16:01Z"
feed: regehr
guid: http://blog.regehr.org/?p=1175
---

# Non-Transparent Memory Safety

\[ [This paper](http://www.cs.berkeley.edu/~necula/Papers/deputy-esop07.pdf) contains more detail about the work described in this post.\]

Instrumenting C/C++ programs to trap memory safety bugs is a popular and important research topic. In general, a memory safety solution has three goals:

- efficiency,

- transparency, and

- compatibility.

Efficiency is obvious. Transparency means that we can turn on memory safety with a switch, we don’t have to do anything at the program level. Compatibility means that safe and unsafe code can freely interact, especially when linking against libraries. Compatibility is tricky because it severely limits the ways in which we can change the layout of memory objects, as we might hope to do in order to store the length of an array along with its data.

One of my favorite memory safety solutions for C — the Deputy project from Berkeley — is distinct from most other work on this space because it does not have transparency as a goal. While this initially seems like a bad idea, and it will obviously limit the amount of legacy code that we can run under Deputy, I eventually came to realize that non-transparency can be a good thing. The goal of this piece is to explain why.

When you write a C or C++ program, you usually intend it to be memory safe. And in fact, a large proportion of C/C++ code in the wild is memory safe, meaning that for all valid inputs it fails to access out-of-bounds or unallocated storage (or it [might mean something else](http://www.pl-enthusiast.net/2014/07/21/memory-safety/), but let’s not worry about that). The problem, of course, is that a small fraction of C/C++ code is not memory safe and some of these errors have serious consequences.

For sake of argument, let’s say that you have written a piece of C code that is memory safe. With some effort you can do this for a small and perhaps for a medium-sized program. Now we might ask: Why is the program memory safe? Where does the memory safety live? Well, the memory safety resides in the logic of the program and perhaps also in the input domain. Unless we’ve used some sort of formal methods tool, the reasoning behind memory safety isn’t written down anywhere, so it’s impossible to verify.

Let’s take your memory safe C program and run it under a transparent memory safety solution like perhaps [SoftBound + CETS](http://www.cs.rutgers.edu/~santosh.nagarakatte/softbound/). What we have now are two totally separate implementations of memory safety: one of them implicit and hard to get right, the other explicitly enforced by the compiler and runtime system.

Deputy is based on the premise that we don’t need two separate implementations of memory safety. Rather, Deputy is designed in such a way that the C programmer can tell the system just enough about her memory safety implementation that it can be checked. Let’s look at an example:

```
int lookup (int *array, int index) {
  return array[index];
}

```

If we don’t trust the developer to get memory safety right, we need to change the code to something like this:

```
int lookup (int *array, int index) {
  assert(index >= 0 && index < array.length);
  return array[index];
}

```

In the C programmer’s implementation of memory safety, the assertion is guaranteed not to fire by the surrounding program logic and by restrictions on the input domain. In a compatible memory safe C, the assertion must be statically or dynamically checked, meaning that we need to know how many int-typed variables are stored in the memory region starting at array. This is not so easy because C has no runtime representation for array lengths. The typical solution is to maintain some sort of fast lookup structure that maps pointers to lengths. A significant complication is that array might point into the middle of some other array. The code that actually executes would look something like this:

```
int lookup (int *array, int index) {
  check_read_ok(array + index, sizeof (int));
  return array[index];
}

```

Getting back to Deputy, the question is: How can the programmer communicate her memory safety argument to the system? It is done like this:

```
int lookup (int *COUNT(array.length) array, int index) {
  return array[index];
}

```

COUNT() is an annotation that tells Deputy what it needs to know in order to do a fast bounds check — no global lookup structure is necessary.

When I first saw the example above, I was not very impressed: it looks like Deputy is just being lazy and punting the problem back to me. But after using Deputy for a while, its genius became apparent. First, whenever I needed to tell Deputy something, the information was always available either in my head or in a convenient program variable. This is not a coincidence: if the information that Deputy requires is not available, then the code is probably not memory safe. Second, the annotations become incredibly useful documentation: they take memory safety information that is normally implicit and put it out in the open in a nice readable format. In contrast, a transparent memory safety solution is highly valuable at runtime but does not contribute to the understandability and maintainability of our code.

There are a number of other Deputy annotations, most notably NTS which is used to tell the system about a null-terminated string and NONNULL which of course indicates a non-null pointer. The [Deputy Quick Reference](https://web.archive.org/web/20110814124657/http://deputy.cs.berkeley.edu/quickref.html) shows the complete set of annotations and the [Deputy Manual](https://web.archive.org/web/20110814124345/http://deputy.cs.berkeley.edu/manual.html) explains everything in more detail and has code examples. The [Deputy paper](http://www.cs.berkeley.edu/~necula/Papers/deputy-esop07.pdf) focuses on more academic concerns and unfortunately contains only a single short example of Deputized C code.

Although the preceding example didn’t make this clear, applying Deputy to C code is pretty easy because the Deputy compiler uses type inference to figure out annotations within each function. Thus, many simple functions can be annotated at the prototype and the compiler takes care of the rest. In more involved situations, annotations are also necessary inside functions. The process for applying Deputy to legacy C code is to compile the code at which point Deputy says where annotations are missing. So you add them and repeat. It’s a nice process where you end up learning a lot about the code that you are annotating. In general, an incorrect annotation cannot lead to memory-unsafe behavior, but it can cause a memory safety violation to be incorrectly reported. (You can write truly unsafe code in Deputy using its UNSAFE annotation, but at least the unsafe code is obvious, as it is in Rust.) My guess is that people who enjoy using assertions would also enjoy Deputy; people who hate assertions may well have a different opinion.

Is Deputy perfect? Certainly not. Most seriously, it is only a partial memory safety solution and does not address use-after-free errors. Its memory safety guarantee does not hold if there are data races. One time I ran into a case where Deputy wouldn’t let me tell it the information that it needed to know, I believe it was when the size of an array was in a struct field. Finally, since it is based on [CIL](http://sourceforge.net/projects/cil/), Deputy supports C but not C++.

My group used Deputy as the basis for our [Safe TinyOS](http://tinyos.stanford.edu/tinyos-wiki/index.php/Safe_TinyOS) project. TinyOS was a nice match for Deputy: the extremely lightweight runtime was suitable for embedded chips with 4 KB of RAM and the lack of use-after-free checking wasn’t a problem since TinyOS doesn’t have malloc/free. We found that in many cases it was sufficient to annotate the TinyOS interface files — which serve much the same role as C header files — and then Deputy didn’t need additional annotations. Here’s an example of an annotated interface:

```
/**
 * @param  'message_t* ONE msg'        the received packet
 * @param  'void* COUNT(len) payload'  a pointer to the packet's payload
 * @param  len                         the length of the data region pointed to by payload
 * @return 'message_t* ONE'            a packet buffer for the stack to use for the next
 *                                     received packet.
 */
event message_t* receive(message_t* msg, void* payload, uint8_t len);

```

There are minor differences from standard Deputy, such as ONE pointers (they “point to one object”) instead of SAFE NONNULL, and we put the annotations into the comments, so they automatically get added to the interface documentation, instead of putting them directly into the function prototypes. There were also some changes under the hood. We found that Deputy was generally a pleasure to use and it caught some nasty bugs in various TinyOS programs.

The current status is that Deputy has not been supported for some time, so it would not be a good choice for a new project. The Deputy ESOP paper has been well cited ( [114 times according to Google Scholar](http://scholar.google.com/scholar?cites=18289891863223357742&as_sdt=5,45&sciodt=0,45&hl=en)) but the basic idea of memory safe C/C++ via annotations and type inference has not caught on, which is kind of a shame since I thought it was a nice design point. On the other hand, even if an updated implementation was available, in 2014 I would perhaps not use Deputy for a new safe low-level project, but would give Rust a try instead, since it has a good story not only for out-of-bounds pointers but also use-after-free errors.
