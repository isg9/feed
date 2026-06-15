---
title: a negative result — wingolog
url: https://wingolog.org/archives/2023/08/08/a-negative-result
published: "2023-08-08T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2023/08/08/a-negative-result
---

# a negative result — wingolog

## [a negative result](/archives/2023/08/08/a-negative-result)

8 August 2023 9:03 AM

- [whippet](/tags/whippet)
- [garbage collection](/tags/garbage%20collection)
- [make](/tags/make)

*Update: I am delighted to have been wrong! See the end.*

Briefly, an interesting negative result: consider benchmarks `b1`, `b2`, `b3` and so on, with associated `.c` and `.h` files. Consider libraries `p` and `q`, with their `.c` and `.h` files. You want to run each benchmark against each library.

P and Q implement the same API, but they have different ABI: you need to separately compile each benchmark for each library. You also need to separate compile each library for each benchmark, because `p.c` also uses an abstract API implemented by `b1.h`, `b2.h`, and so on.

The problem: can you implement a short GNU `Makefile` that produces executables `b1.p`, `b1.q`, `b2.p`, `b2.q`, and so on?

The answer would appear to be “no”.

You might think that with `call` and all the other functions available to you, that surely this could be done, and indeed it’s easy to take the cross product of two lists. But what we need are new rules, not just new text or variables, and you can’t programmatically create rules. So we have to look at rules to see what facilities are available.

Consider the rules for one target:

```
b1.p.lib.o: p.c
	$(CC) -o $@ -include b1.h $<
b1.p.bench.o: b1.c
	$(CC) -o $@ -include p.h $<
b1.p: b1.p.lib.o b1.p.bench.o
    $(CC) -o $@ $<

```

With [pattern
rules](https://www.gnu.org/software/make/manual/html_node/Pattern-Rules.html), you can easily modify these rules to parameterize *either* over benchmark or over library, but not both. What you want is something like:

```
*.%.lib.o: %.c
	$(CC) -o $@ -include $(call extract_bench,$@) $<
%.*.bench.o: %.c
	$(CC) -o $@ -include $(call extract_lib,$@) $<
%: %.lib.o %.bench.o
	$(CC) -o $@ $<

```

But that doesn’t work: you can’t have a wildcard ( `*`) in the pattern rule. (Really you would like to be able to match multiple patterns, but the above is the closest thing I can think of to what `make` has.)

[Static pattern
rules](https://www.gnu.org/software/make/manual/html_node/Static-Pattern.html) don’t help: they are like pattern rules, but more precise as they apply only to a specific set of targets.

You might think that you could use `$*` or other special variables on the right-hand side of a pattern rule, but that’s not the case.

You might think that [secondary
expansion](https://www.gnu.org/software/make/manual/html_node/Secondary-Expansion.html) might help you, but then you open the door to an annoying set of problems: sure, you can mix variable uses that are intended to be expanded once with those to be expanded twice, but the former set better be idempotent upon second expansion, or things will go weird!

Perhaps the best chance for a `make`-only solution would be to recurse on generated makefiles, but that seems to be quite beyond the pale.

To be concrete, I run into this case when benchmarking [Whippet](https://github.com/wingo/whippet-gc): there are some number of benchmarks, and some number of collector configurations. Benchmark code will inline code from collectors, from their header files; and collectors will inline code from benchmarks, to implement the trace-all-the-edges functionality.

So, with Whippet I am left with the strange conclusion that the only reasonable thing is to generate the [`Makefile`](https://github.com/wingo/whippet-gc/blob/main/Makefile) with a little custom generator, or at least generate the part of it to do this benchmark-library cross product. It’s hard to be certain about negative results with `make`; perhaps there is a trick. If so, do let me know!

### epilogue

Thanks to [a kind note from Alexander Monakov](https://mastodon.gamedev.place/@amonakov/110853258601803460), I am very happy to be proven wrong.

See, I thought that [`make` functions](https://www.gnu.org/software/make/manual/html_node/Functions.html) were only really good in variables and rules and the like, and couldn’t be usefully invoked “at the top level”, to define new rules. But that’s not the case! [`eval`](https://www.gnu.org/software/make/manual/html_node/Eval-Function.html) in particular can define new rules.

So a solution with `eval` might look something like this:

```
BENCHMARKS=b1 b2 b3
LIBS=p q

define template
$(1).$(2).lib.o: $(2).c
	$$(CC) -o $$@ -include $(1).h $$<
$(1).$(2).bench.o: $(1).c
	$$(CC) -o $$@ -include $(2).h $$<
$(1).$(2): $(1).$(2).lib.o $(1).$(2).bench.o
    $$(CC) -o $$@ $$<
end

$(foreach BENCHMARK,$(BENCHMARKS),\
  $(foreach LIB,$(LIBS),\
     $(eval $(call template,$(BENCHMARK),$(LIB)))))

```

Thank you, Alexander!

## related articles

- [ahead-of-time wasm gc in wastrel](/archives/2026/02/06/ahead-of-time-wasm-gc-in-wastrel)
- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [whippet lab notebook: guile, heuristics, and heap growth](/archives/2025/05/22/whippet-lab-notebook-guile-heuristics-and-heap-growth)
- [guile on whippet waypoint: goodbye, bdw-gc?](/archives/2025/05/15/guile-on-whippet-waypoint-goodbye-bdw-gc)
- [a whippet waypoint](/archives/2025/05/09/a-whippet-waypoint)
- [partitioning ambiguous edges in guile](/archives/2025/04/25/partitioning-ambiguous-edges-in-guile)

### One response

1. Taylor R Campbell says:[8 August 2023 12:28 PM](#a64bde95b380c30135cc809bdc3b3b9b25028c6b)

   I don’t know about GNU make, but this is very easy in BSD make. I would guess you can do it with GNU make too, but with much more cumbersome syntax to generate targets in the equivalent of a BSD make .for loop.

   ```
   $ cat Makefile
   all:;
   clean:
   	-rm -f p.o
   	-rm -f q.o

   .PHONY: all
   .PHONY: clean

   BENCH+=	b1
   BENCH+=	b2
   BENCH+=	b3

   LIB+=	p
   LIB+=	q

   API_H.p=	"p.h"
   API_H.q=	"q.h"

   .for L in ${LIB}
   .  for B in ${BENCH}
   all: $B.$L.out
   $B.$L.out: $B.$L
   	./$B.$L >$@
   $B.$L: $B.$L.o $L.o
   	${CC} -o $@ $B.$L.o $L.o
   $B.$L.o: $B.c
   	${CC} -o $@ -c $B.c -DAPI_H=${API_H.$L:Q}
   clean: clean-$B.$L
   .PHONY: clean-$B.$L
   clean-$B.$L:
   	-rm -f $B.$L
   	-rm -f $B.$L.o
   	-rm -f $B.$L.out
   .  endfor
   .endfor
   $ head -11 *.[ch]
   ==> b1.c <==
   #include

   #include API_H

   int
   main(void)
   {
           printf("b1: %d\n", api());
           fflush(stdout);
           return ferror(stdout);
   }

   ==> b2.c <==
   #include

   #include API_H

   int
   main(void)
   {
           printf("b2: %d\n", api());
           fflush(stdout);
           return ferror(stdout);
   }

   ==> b3.c <==
   #include

   #include API_H

   int
   main(void)
   {
           printf("b3: %d\n", api());
           fflush(stdout);
           return ferror(stdout);
   }

   ==> p.c <==
   #include "p.h"

   int var = 42;

   ==> p.h <==
   #ifndef P_H
   #define P_H

   extern int var;

   static inline int
   api(void)
   {
           return var;
   }


   ==> q.c <==
   #include "q.h"

   int
   func(void)
   {
           return 9;
   }

   ==> q.h <==
   #ifndef Q_H
   #define Q_H

   #define api()   func()

   int func(void);

   #endif  /* Q_H */
   $ make
   cc -o b1.p.o -c b1.c -DAPI_H=\"p.h\"
   cc -O2   -c p.c
   cc -o b1.p b1.p.o p.o
   ./b1.p >b1.p.out
   cc -o b2.p.o -c b2.c -DAPI_H=\"p.h\"
   cc -o b2.p b2.p.o p.o
   ./b2.p >b2.p.out
   cc -o b3.p.o -c b3.c -DAPI_H=\"p.h\"
   cc -o b3.p b3.p.o p.o
   ./b3.p >b3.p.out
   cc -o b1.q.o -c b1.c -DAPI_H=\"q.h\"
   cc -O2   -c q.c
   cc -o b1.q b1.q.o q.o
   ./b1.q >b1.q.out
   cc -o b2.q.o -c b2.c -DAPI_H=\"q.h\"
   cc -o b2.q b2.q.o q.o
   ./b2.q >b2.q.out
   cc -o b3.q.o -c b3.c -DAPI_H=\"q.h\"
   cc -o b3.q b3.q.o q.o
   ./b3.q >b3.q.out
   $ head *.out
   ==> b1.p.out <==
   b1: 42

   ==> b1.q.out <==
   b1: 9

   ==> b2.p.out <==
   b2: 42

   ==> b2.q.out <==
   b2: 9

   ==> b3.p.out <==
   b3: 42

   ==> b3.q.out <==
   b3: 9
   ```
   ```

Comments are closed.
