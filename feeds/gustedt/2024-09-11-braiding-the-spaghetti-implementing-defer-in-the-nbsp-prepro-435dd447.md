---
title: 'Braiding the spaghetti: implementing defer in the&nbsp;preprocessor'
url: https://gustedt.wordpress.com/2024/09/11/braiding-the-spaghetti-implementing-defer-in-the-preprocessor/
published: "2024-09-11T20:34:00Z"
feed: gustedt
guid: https://gustedt.wordpress.com/?p=4172
---

# Braiding the spaghetti: implementing defer in the&nbsp;preprocessor

I already talked about [a proposal for an extension of the C language called

**defer**](https://gustedt.wordpress.com/2020/12/14/a-defer-mechanism-for-c/), that is prospected to either be published in a technical

specification or the next revision, C2Y, of the C standard:

- [Defer Mechanism for C](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2542.pdf)
- [defer reference implementation for C](https://gustedt.gitlabpages.inria.fr/defer)

[EĿlipsis](https://gustedt.gitlabpages.inria.fr/ellipsis/) now implements a simple form of this feature by mixing some specific preprocessor extensions (in particular counters) with the usual macro magic.

This feature is motivated by code patterns as the following, which use

`goto` jumps to ensure that certain cleanup code snippets are executed

in the correct order and under the right circumstances:

```
void fun() {
   void * const p = malloc(25);
   if (!p) goto DEFER0;

   void*const q = malloc(25);
   if (!q) goto DEFER1;

   if (mtx_lock(&mut)==thrd_error) goto DEFER2;

   // all resources acquired

   // ... use p and q under protection of mut ... then
   mtx_unlock(&mut);

 DEFER2:
   free(q);
 DEFER1:
   free(p);
 DEFER0:;
}

```

Such spaghetti patterns are quite commonly used in C because they

currently provide the only systematic tool to do such cleanup. The

pattern has several disadvantages

- cleanup code appears far from the place where its need is detected
- we have to invent a naming scheme for labels that has to be unique within the same function
- adding new resources and new cleanup code easily introduces bugs

The `defer` feature avoids all of these. Using this inside a compound

statement indicates that that code is to be executed at the end of that

compound statement.

```
#include_directives <ellipsis-defer.h>

void fun() {
  void * const p = malloc(25);
  if (!p) return;
  defer free(p);

  void * const q = malloc(25);
  if (!q) return;
  defer free(q);

  if (mtx_lock(&mut)==thrd_error) return;
  defer mtx_unlock(&mut);

  // all resources acquired

  // ... use p and q under protection of mut ... then
}

```

As you can see, this does not force us to make arbitrary naming choices for labels, and has the cleanup close to where its need arises. For this simplified example the idea is that all statements that are prefixed with `defer` are executed at the end of the function in the reversed order in which they appear lexicographically.

[EĿlipsis](https://gustedt.gitlabpages.inria.fr/ellipsis/) enables a simple form of this and the point of this blog entry here is to show that already a slightly enhanced preprocessor can be used to hide all that complexity from the programmer. The main idea is that the labels that are needed have nice properties that only depend on the syntactical placement of the `defer` statement within a compound statement `{...}` . In a first phase eĿlipsis translates the function to something like the following.

```
void fun() {
  void * const p  = malloc(25);
  if (!p) return;
  /* defer */
  [[__maybe_unused__]] register bool DEFER_LOC_ID_0_2 = false;
  DEFER_TO(DEFER_END_ID_1_3, DEFER_ID_1_7): free(p);

  void * const q  = malloc(25);
  if (!q)   /* return mode 1 */
    switch((DEFER_LOC_ID_0_2  = true))
    default: goto DEFER_ID_1_7;
  /* defer */
  DEFER_TO(DEFER_ID_1_7, DEFER_ID_1_8): free(q);

  if (mtx_lock(&mut)==thrd_error)   /* return mode 1 */
    switch((DEFER_LOC_ID_0_2  = true))
    default: goto DEFER_ID_1_8;
  /* defer */
  DEFER_TO(DEFER_ID_1_8, DEFER_ID_1_9): mtx_unlock(&mut);

  goto DEFER_ID_1_9;
DEFER_END_ID_1_3: ;
}

```

So what happend here?

- At each `defer` two names for labels are added. The first identifies the jump target of the next `defer` code; the second identifies the `defer` under construction.
- The definition of a local variable `LOC_ID_0_2` has been added. This holds the information if we are currently in the process of executing `defer` statements or not. This variable is useful when we have nested compound statements; here a `return` has to continue to execute `defer` statements of surrounding compound statements.
- `return` statements are essentially replaced by `goto` with the correct target label, namely the first `defer` code to be executed when returning from that position. Additionally they set the variable to `true` such that the change in state is recorded.
- At the end of the body we see another `goto` that ensures that the `defer` statements are all executed when the end of the function is reached.
- After that, there is an additional label that serves as target of the first `defer`.

Note also that the start of the function up to the first `defer` is not changed and behaves exactly as without any `defer`.

The resulting code might look complicated, and is even much less readable if the `DEFER_TO` macro is expanded in addition; you probably don’t want to see that ever in your life. But the complexity of that resulting code is only to us humans, we are not able to easily follow each thread of the spaghetti. Modern compilers are perfectly fine with this and basically compile all of this as if they had been presented the linearized code that is on top of this post. In addition they may provide you with valuable feedback if we enable static analysis.

The point here is that already the preprocessor can be used to hide all that complexity from the programmer.

The current implementation in eĿlipsis still has some drawbacks

- There is no way to guess a return type of a function. So functions with non- `void` return need an extra macro, `DEFER_TYPE` that provides this information.
- `break` and `continue` statements don’t work across `defer`.

The first is really only a minor inconvenience. If we transform the `fun` function from above to a `main` with returns with expressions, such as in

```
int main() {
  DEFER_TYPE(int);
  void * const p = malloc(25);
  if (!p) return EXIT_FAILURE;
  defer free(p);

  void * const q = malloc(25);
  if (!q) return EXIT_FAILURE;
  defer free(q);

  if (mtx_lock(&mut)==thrd_error) return EXIT_FAILURE;
  defer mtx_unlock(&mut);

  // all resources acquired

  // ... use p and q under protection of mut ... then
  return EXIT_SUCCESS;
}

```

the information given by `DEFER_TYPE` is sufficient to transform this into valid C23 code. The replacement done by eĿlipsis then looks even a bit more complicated than before:

```
int main() {
  typeof(int) DEFER_LOC_ID_0_1;
  [[__maybe_unused__]] register bool DEFER_LOC_ID_0_2 = false;
  if (false)
  DEFER_ID_1_10: return DEFER_LOC_ID_0_1;
  void * const p  = malloc(25);
  if (!p) DEFER_RETURN_TO(DEFER_ID_1_10) EXIT_FAILURE;
  /* defer */
  DEFER_TO(DEFER_ID_1_10, DEFER_ID_1_11): free(p);

  void * const q  = malloc(25);
  if (!q) DEFER_RETURN_TO(DEFER_ID_1_11) EXIT_FAILURE;
  /* defer */
  DEFER_TO(DEFER_ID_1_11, DEFER_ID_1_12): free(q);

  if (mtx_lock(&mut)==thrd_error) DEFER_RETURN_TO(DEFER_ID_1_12) EXIT_FAILURE;
  /* defer */
  DEFER_TO(DEFER_ID_1_12, DEFER_ID_1_13): mtx_unlock(&mut);

  DEFER_RETURN_TO(DEFER_ID_1_13) EXIT_SUCCESS;
  goto DEFER_ID_1_13;
[[__maybe_unused__]] DEFER_END_ID_1_4: ;
}

```

- Another variable, `DEFER_LOC_ID_0_1`, is added up front. It holds the return value of the function, if any.
- The only “real” `return` statement that is left is an artificial one that is only reachable through a label, here `DEFER_ID_1_10`.
- The original `return` statements are replaced by a macro `DEFER_RETURN_TO(label)` where the label identifies the first `defer` that has to be executed after the return value has been determined.

I also have some ideas how to make this `defer` implementation work with `break` and `continue`, but that is unfortunately a bit more nasty.
