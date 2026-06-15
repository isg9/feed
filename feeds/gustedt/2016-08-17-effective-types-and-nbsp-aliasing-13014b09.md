---
title: Effective types and&nbsp;aliasing
url: https://gustedt.wordpress.com/2016/08/17/effective-types-and-aliasing/
published: "2016-08-17T13:49:25Z"
feed: gustedt
guid: https://gustedt.wordpress.com/?p=2320
---

# Effective types and&nbsp;aliasing

I already have posted about the [evilness of cast](https://gustedt.wordpress.com/2014/04/02/dont-use-casts-i/) some time ago, but recently I have seen that there is still much confusion out there about the C rules that pointer casts, also called *type punning*, break. So I will try to give it a more systematic approach, here.

In fact, I think we should distinguish two different sets of rules, the *effective type rules* that are explicitly written up in the C standard (6.5 p6 and p7), and the *strict aliasing rules* that are consequence of these, but that concern only a very special use case of type punning.  They represent only one of the multiple ways a violation of the effective type rules can damage the program state.

*Note:* To simplify things, I will not deal with qualifications such as `const` or `volatile` in the following, nor with the fact that types only have to be *compatible* to be correct.

**Variables.** For objects that have a declaration, that is all your plain, normal variables, the effective type rules are in fact quite simple.

> Variables must be accessed through their declared type or through a pointer to a character type.

So taking the address of a variable with `&` and then cast the result to a different pointer type is always wrong.

```
float f = 55;
uint32_t trans = *(uint32_t*)&amp;f;   // evil

Observe that there is no aliasing involved, there is only one pointer at any moment to the object f. There are many things that can go wrong on a particular platform with the above code: the alignment of f could be wrong for the new type, it could be allocated in a special memory section that speeds up floating point arithmetic, whatever. The C standard didn’t wanted to impose any restriction of what compilers can do with different data types, and so such access is simply forbidden. Don’t do it, there is no excuse for such evil code.
The exception that is explicitly allowed are character types. That is this code is ok
float f = 55;
unsigned char* bytes = (unsigned char*)&amp;f;
printf(&quot;the first two bytes are %hhu and %hhu\n&quot;, bytes[0], bytes[1]);

That is inspecting the bytes of any object type is always permitted. In fact, if you are careful, you may even change the object representation of an object by changing its bytes through such a character pointer.
float f = 55;
uint32_t trans;
memcpy(&amp;trans, &amp;f, sizeof trans);   // ok

This is ok, because

under the hood this reads f and modifies trans through character pointers
the type uint32_t is such that all bit representations lead to a valid value.

Whereas accessing an arbitrary object through the byte angle is ok, the converse is not true:
Character arrays must not be reinterpreted as objects of other types.
Dynamically allocated objects. Objects that are not declared but allocated dynamically (e.g with malloc) fall under rules that are a bit weaker. This is so, because they initially have no type, and so we can’t request anything for it. This is where the notion of effective type comes into play. Such an allocated object is associate with a type, its effective type, once we write data with a known type into it. Such a “write” can happen through an assignment or through functions like memcpy.
A simple such case is if we access the object through a pointer with the sought type, and this should always be your preferred way to initialize a dynamically allocated object:
unsigned a = 55;
float* gp = malloc(sizeof *gp);
*gp = a;                          // *gp now is a float

Another is if we use memcpy from an object of known type:
float f = 55;
void* vp = malloc(sizeof f);
memcpy(vp, &amp;f, sizeof f);        // *vp now is a float
float* gp = malloc(sizeof *gp);
memcpy(gp, &amp;f, sizeof *gp);      // *gp now is a float

But by such copies we easily can get it wrong
float f = 55;
uint32_t* up = malloc(sizeof *up);
memcpy(up, &amp;f, sizeof *up);                  // *up now is a float!
printf(&quot;the value is %&quot; PRIu32 &quot;\n&quot;, *up);   // bad: accessing float as uint32_t

So you should always be careful and ensure that an object actually has the correct type.
Change the type of an object. Where the type of a variable can never change, a dynamically allocated object actually can.
float f = 55;
void* vp = malloc(sizeof f);
memcpy(vp, &amp;f, sizeof f);                   // *vp now is a float
float* fp = vp;
// do things with *fp
printf(&quot;the value is %g\n&quot;, *fp);           // ok
uint32_t u = 77;
memcpy(vp, &amp;u, sizeof u);                   // *vp now is a uint32_t
uint32_t* up = vp;
printf(&quot;the value is %&quot; PRIu32 &quot;\n&quot;, *up);  // ok

Here, the second memcpy not only changes the values of each byte of the object, but it also changes its effective type.
Unions. C provides a simple tool to look at an object with different type views, a union. All this pointer casting garbage is really unnecessary if we just use the appropriate tool.
union both {
  float f;
  uint32_t u;
} b = {
  .f = 55,
};
printf(&quot;the value is %&quot; PRIu32 &quot;\n&quot;, b.u);   // ok

Here, we make it explicit that we want to see b as both, sometimes a float, sometimes a uint32_t. Now the compiler can take all precautions that they need to deal with that situation. This will be as efficient as the compiler can get it.
If you really need to reinterpret representations very often (but you shouldn’t) you can use a macro to do so:
#define CONV32(X) ((const union { float _f; uint32_t _u;}){ ._f = (X), }._u)

float f = 55;
printf(&quot;the value is %&quot; PRIu32 &quot;\n&quot;, CONV32(f));   // ok

This uses a temporary object of union type. Any decent compiler should realize this with out actually using a temporary object and will most probably just move a value from one hardware register to another. But this is their dealings, not ours.
Aliasing. Only if we have taken care of all of the above we actually come to aliasing problems. Aliasing is the property of two pointers that point to the same object and problems may occur with that if we change the underlying object through one of them.
In view of the effective type rule, C makes a simple assumption
Non-character pointers of different type cannot alias.
That is if a function see two pointers like here
void doit(float* fp, uint32_t* up) {
   *up = 3;
   *fp = 77;
   printf(&quot;the value is %&quot; PRIu32 &quot;\n&quot;, *up);   // may always print 3
}

it can always assume that the change of *fp doesn’t affect the object *up and thus that the value to be printed is 3, unconditionally. If you messed around with the types and passed pointers to the same object, you are on your own, even if, maybe, your type reinterpretation was correct through one of the exceptions that we discussed above.
So it is very likely that when you stumble into an aliasing problem with code as in this function, you already are in nowhere land of undefined behavior, because you had your types wrong from the beginning.
Wait, so why does my networking code actually work? Traditional networking code with its reinterpretation of socket address data is notorious for stretching the effective type rules to their very limits.
Consider the following typical network code snippet:
void fun(int fd) {
  socklen_t addrlen = sizeof(sockaddr_storage);
  void* p = malloc(addrlen);
  struct sockaddr_storage* sockp = p;     // *sockp has no type
  int ret = getsockname(fd, p, &amp;addrlen); // *sockp now has type, but which?
  if (ret) { /* handle error */ }
  switch (sockp-&gt;ss_family) {             // valid access
  case AF_UNIX: {
    struct sockaddr_un* sockp = p;        // *sockp is a unix socket
    // do something
    break;
  }
  case AF_INET: {
    struct sockaddr_in* sockp = p;        // *sockp is an internet v4 socket
    // do something
    break;
  }
  }
  free(p);
}

This approach only works because a very specific chaining of events. The first important event is that the function getsockname provides a type for the object behind p (or maybe changes that type, if it had another one, before). Now we have learned above that this is only possible because that object is allocated dynamically.
Then, the next critical access is to read the ss_family member of that object. This is possible, because the definition of all sockaddr derivate types is such that they all have a member of the same type (sa_family_t) at exactly the same position in the structure. Accessing a member of an object via -> or . is guaranteed not to access the whole object (here that would be *sockp) but only the partial object that is described by the member. Thus reading sockp->ss_family always accesses an object of the same type (sa_family_t) and so the access obeys the effective type rules.
Finally, when we are inside the different cases, we know which of the types is the effective type and we can correctly access and deal with data of a unix or inet socket, for example.
So, all of this only worked because of a very precise sequencing of accesses and because the object had been allocated dynamically.
I personally prefer to go for less error prone networking code and to use a union:
void fan(int fd) {
  union {
    struct sockaddr_storage ss;
    struct sockaddr_un un;
    struct sockaddr_in in;
  } sock;
  socklen_t addrlen = sizeof sock;
  int ret = getsockname(fd, &amp;sock, &amp;addrlen);
  if (ret) { /* handle error */ }
  switch (sock.ss.ss_family) {             // valid access
  case AF_UNIX: {
    // do something with sock.un
    return;
  }
  case AF_INET: {
    // do something with sock.in
    return;
  }
  }
}

Such code is more robust, because we clearly announce that the object may have different interpretations, because it doesn’t depend on the storage class, and because we can’t forget to free the object at the end.

```
