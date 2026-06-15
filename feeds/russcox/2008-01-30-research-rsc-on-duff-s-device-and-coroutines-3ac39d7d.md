---
title: 'research!rsc: On Duff''s Device and Coroutines'
url: https://research.swtch.com/duff
published: "2008-01-30T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/duff
---

# research!rsc: On Duff's Device and Coroutines

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# On Duff's Device and Coroutines Russ Cox January 30, 2008 *research.swtch.com/duff* Posted on Wednesday, January 30, 2008.

At first glance, Duff's Device is one of the most mysterious pieces of C code you'll ever see:

```
    void
    send(short *to, short *from, int count)
    {
        int n=(count+7)/8;
        switch(count%8){
        case 0: do{ *to = *from++;
        case 7:     *to = *from++;
        case 6:     *to = *from++;
        case 5:     *to = *from++;
        case 4:     *to = *from++;
        case 3:     *to = *from++;
        case 2:     *to = *from++;
        case 1:     *to = *from++;
                }while(--n>0);
        }
    }

```

It's an 8x-unrolled while loop interlaced with a switch statement. The switch jumps into the middle of the while loop to handle the part of the count that isn't a multiple of 8. Once inside the while loop, the switch logic never executes again.

Tom Duff first described Duff's Device in a [November 1983 email to Ron Gomes, Dennis Ritchie, and Rob Pike](http://swtch.com/duffs-device/td-1983.txt). It spread to a larger group when he revised the note and **[posted it to net.lang.c](http://swtch.com/duffs-device/usenet-1984.txt)** in May 1984. The 1984 message is the less frequently reproduced but is the one in which Duff gave it a name: “I think I'll name it after myself — ‘Duff's Device’ has a nice ring to it.” Bjarne Stroustroup used a variant as an example in *The C++ Programming Language*. Eventually Duff posted [the 1983 message along with commentary](http://swtch.com/duffs-device/usenet-1988.txt) to Usenet in 1988, in response to a [heated debate](http://groups.google.com/group/comp.lang.c/browse_thread/thread/b1626d1e7eac569b/bb78298175c42411) about the merits of Duff's Device as an idiom.

Duff's Device is rarely worth reusing, but in the specific case that it was written for, it was an apt tool for the job. The more amazing thing about it is that it's valid C. Since a switch statement is just a computed goto and the case labels are just labels, switching into the middle of a while is just as legal as goto'ing there. In Duff's description, he wrote:

> It amazes me that after 10 years of writing C there are still little corners that I haven't explored fully. (Actually, I have another revolting way to use switches to implement interrupt driven state machines but it's too horrid to go into.)

In 2000, Simon Tatham wrote about a [C coroutine implementation](http://www.chiark.greenend.org.uk/~sgtatham/coroutines.html) based on this idea, to make callback-based event-driven programming easier. (He used it in his Windows SSH client, [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/).) A switch-based implementation has the advantage of being portable and relatively simple to emit using preprocessor macros, though it doesn't allow the use of other switch statements in the code.

A year ago, I asked Duff if that kind of coroutine implementation is what he had in mind, and he confirmed that it was, pointing at a [blog comment](http://brainwagon.org/?p=1060) and adding:

> I only ever did this twice: once for a coroutinish application in an SDLC driver (for UNIX to IBM OS/360 communication) in the 1970s and once for loop unwinding in 1983. Both were desperate situations. I have never felt the need to stoop so low since then.

(Today, of course, there are few excuses not to use threads, or at the very least, [coroutines with real stacks](http://swtch.com/libtask/). If you are stuck with callbacks and events, something like Max Krohn's [tame preprocessor](http://pdos.lcs.mit.edu/~max/docs/tame.pdf) is a cleaner solution than macros.)

(Comments originally posted via Blogger.)

- [Laurie Cheers](http://www.blogger.com/profile/09919006779080998004)(January 30, 2008 5:13 AM) You missed out all the ++s after the "to"s.

- [patraulea](http://www.blogger.com/profile/00286273078745020256)(January 30, 2008 5:24 AM) Regarding the missing ++ after 'to': if you are outputting to a memory-mapped IO port, 'to' doesn't need to increment.

I think one mentioned usage for Duff's device was exactly this.

- [Russ Cox](http://swtch.com/~rsc/)(January 30, 2008 5:42 AM) Right. The original instance of Duff's Device was writing to a memory-mapped device, so there was no to++. See the linked messages. (Stroustroup's variant in *The C++ Programming Languages* did use to++, presumably to avoid discussing arcana like memory-mapped I/O.)

I did update the code to use modern C prototypes, so that readers wouldn't be distracted by the pre-ANSI-isms.

- [viric](http://viric.livejournal.com/)(February 18, 2008 12:53 PM) Thank you for your series of posts, Russ.

I think the people from [Contiki OS](http://www.sics.se/contiki/) also deserve some mention with their [protothreads](http://www.sics.se/~adam/pt/), an implementation similar to the coroutines wrapped in C macros. They have quite interesting achievements.

- [Ralph Corderoy](http://www.blogger.com/profile/13140975971019765573)(August 4, 2010 6:29 AM) Just pointing out for other interested readers that Tame is C++, not C.
