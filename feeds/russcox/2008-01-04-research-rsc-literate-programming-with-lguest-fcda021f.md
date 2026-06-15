---
title: 'research!rsc: Literate Programming with Lguest'
url: https://research.swtch.com/lguest
published: "2008-01-04T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/lguest
---

# research!rsc: Literate Programming with Lguest

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Literate Programming with Lguest Russ Cox January 4, 2008 *research.swtch.com/lguest* Posted on Friday, January 4, 2008.

Norman Ramsey defines *literate programming* as “the art of preparing programs for human readers.”

Donald Knuth invented literate programming when Tony Hoare (inventor of quicksort) challenged him to publish TeX (the program). In his article introducing literate programming, Knuth explained:

> I believe that the time is ripe for significantly better documentation of programs, and that we can best achieve this by considering programs to be works of literature. Hence, my title: “Literate Programming.”
>
> Let us change our traditional attitude to the construction of programs: Instead of imagining that our main task is to instruct a computer what to do, let us concentrate rather on explaining to human beings what we want a computer to do.
>
> The practitioner of literate programming can be regarded as an essayist, whose main concern is with exposition and excellence of style. Such an author, with thesaurus in hand, chooses the names of variables carefully and explains what each variable means. He or she strives for a program that is comprehensible because its concepts have been introduced in an order that is best for human understanding, using a mixture of formal and informal methods that reinforce each other.

You can't beat literate programs for completeness. In a good literate program, every line is explained in clear prose. Having to explain the program using English often forces the program itself to be more clearly written too.

Good, full-size literate programs are rare. Knuth's book [TeX: The Program](http://www.amazon.com/-/dp/0201134373) and Chris Fraser and David Hanson's book [lcc, a Retargetable Compiler for ANSI C](http://www.amazon.com/-/dp/0805316701/) are two, but I'd love to hear about more.

Today's link is to the **[literate version of lguest](http://swtch.com/lguest/)**, an up-and-coming x86 virtual machine monitor for Linux. To first approximation, [lguest](http://lguest.ozlabs.org/) is like [Xen](http://xen.org/) or [VMware](http://www.vmware.com/) but simpler.

(Comments originally posted via Blogger.)

- [Russ Cox](http://swtch.com/~rsc/)(January 19, 2008 9:02 AM) Ajay Kapal reports that [Physically Based Rendering: From Theory to Implementation](http://www.amazon.com/-/dp/012553180X) gives the implementation of a ray tracer using literate programming.

- [Ralph Corderoy](http://www.blogger.com/profile/13140975971019765573)(July 4, 2010 6:03 AM) Fraser and Hanson's follow-up to lcc, *C Interfaces and Implementations*, is also literate, as mentioned in their [preface](http://sites.google.com/site/cinterfacesimplementations/preface).
