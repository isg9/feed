---
title: Demystify undefined behavior
url: https://gustedt.wordpress.com/2015/04/25/demystify-undefined-behavior/
published: "2015-04-25T21:36:25Z"
feed: gustedt
guid: https://gustedt.wordpress.com/?p=2218
---

# Demystify undefined behavior

In discussions about C or other standards (e.g POSIX) you may often have come across the term *undefined behavior,* UB for short. Unfortunately this term is sometimes mystified and even used to scare people unnecessarily, claiming that arbitrary things even outside the scope of your program can happen if a program “ *encounters* undefined behavior”. (And I probably contributed to this at some point.)

First, this is just crappy language. How can something “encounter” behavior?  It can’t. What is simple meant by such slang is that such a program has no defined behavior, or stated yet otherwise, the standard doesn’t define what to do if such a program reaches the state in question. The C standard defines *undefined behavior*

> behavior, upon use of a nonportable or erroneous program construct or of erroneous data,
>
> for which this International Standard imposes no requirements

That’s it.

Simple examples of undefined behavior in C are out-of-bounds access to arrays or overflow of signed integer types. More complicated cases arise when violating aliasing rules or when access to data races between different threads.

Now we often hear statements like “ *UB may format your hard drive*” or “ *UB may steal all your money from your bank account*” implying somehow that a program that is completely unrelated to my disk administration or to my online banking, could by some mysterious force have these evil effects. This is (almost) complete nonsense.

If UB in a simple program of yours formats your hard drive, you shouldn’t blame your program. No simple application run by an unprivileged user should have such devastating consequences, this would be completely inappropriate. If such things happen, it is your system which is at fault, get you another one.

As an analogy from every-day life, take the idea of locking your house at night, which seems to be the rule in some societies. Sure, if you don’t do that, you make it easier for somebody to sneak into your house and shoot you.   But still, that person would be a murderer, to be punished for that by the law, no sane legal system would acquit such a person or see this as mitigating circumstances.

Now things *are* in fact different when there is a direct relation between the object of your negligence and the ill-effect that it causes. If you leave your purse on the table in the pub when going to the bathroom, you should take a share in responsibility if it is stolen. Or to come back to the programming context, if you are programming a security sensible application (e.g handling passwords or bank credentials) then you should be extremely careful and stay on well-defined grounds.

When programming in C there are different kinds of constructs or data for which the program behavior then is undefined.

- Behavior can be explicitly declared undefined or implicitly left undefined.
- The error can be detectable or undetectable at compile time.
- The error can be detectable or undetectable during execution.
- Behavior of certain constructs can be undefined by a specific standard to allow extensions.

Unfortunately, people often use UB as an excuse to evacuate questions about certain behavior of their code (or compiler). This is just *rude behavior* by itself, and you usually shouldn’t accept this. An implementation should have reasons and policies how it deals with UB, otherwise it is just bad, and you should switch to something more friendly, if you may.

> There is no reason not to be friendly, and there are many to be.

That is, in our context, once a (your!) code detects that it has a problem it *must* handle it. Possible strategies are

- abort the compilation or the running program
- return an error code
- report the problem
- define and document the behavior in your own terms

For the first you should always have in mind that the program should be debugable, so you really should use `#error` or `static_assert` for compile time errors and `assert` and `abort` for run time errors. Also, willingly aborting the program is not the same as having the program crash, see below.

Obviously, the second is only possible if the function in question has an established error return convention and if you can expect users of the function to check for that convention. POSIX has many such cases where the documentation says something like “may” return a certain error code. A C (or POSIX) library implementation that detects such a “may” case and doesn’t react accordingly, is of bad quality.

Reporting detected errors is an important alternative and some compiler implementors have chosen this as their default answer to problems. Perhaps you already have seen gcc’s famous “diagnostic” message

> dereferencing pointer ‘bla’ does break strict-aliasing rules

This message supposes that you know what aliasing rules are, and that you also know why it came to that. Observe that it says “ *does*” so the compiler claims to have proven that the aliasing rules are violated, and that the behavior is thus undefined. What the message doesn’t tell, and that is bad, is how it resolves the problem.

The last variant, defining your own stuff, should be handled with extreme care. Not only that what you define must be sensible, you also make a commitment in doing so. You promise your clients that you will follow these new rules in the future and that they may suppose that you will take care of the underlying problem. In most cases, you should leave the definition of extensions to the big players, platform designers or other standards. E.g POSIX defines a lot of cases that are UB for C as such.

As a last alternative, when

- the error is undetectable
- the detection of the error would be too expensive

you should simply let the program crash. The best strategy I know of is to always initialize all variables and always treat 0 as a special value. This may, under rare circumstance deal a tiny bit of performance against security, because on some rare occasions your compiler might not be able to optimize an unused initialization. If you do this, most errors that you will see are dereferences of null pointers.

Dereferencing a null pointer has UB. But modern architecture no how to handle this: they raise a  “segmentation fault” error and terminate the program. This is the nicest failure path that you can offer to your clients, failing anytime, anywhere.
