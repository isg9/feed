---
title: ports, weaks, gc, and dark matter — wingolog
url: https://wingolog.org/archives/2011/02/25/ports-weaks-gc-and-dark-matter
published: "2011-02-25T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2011/02/25/ports-weaks-gc-and-dark-matter
---

# ports, weaks, gc, and dark matter — wingolog

## [ports, weaks, gc, and dark matter](/archives/2011/02/25/ports-weaks-gc-and-dark-matter)

25 February 2011 1:59 PM

- [gc](/tags/gc)
- [scheme](/tags/scheme)
- [guile](/tags/guile)
- [azul](/tags/azul)

Just wanted to drop a note about some recent bug-hunting. I was experiencing memory leaks in a [long-running web server](//wingolog.org/software/tekuti), and, after fixing some port finalization leaks, ran into a weird situation: if I did a `(gc)` after every server loop, the memory usage (measured by [`mem_usage.py`](//wingolog.org/pub/mem_usage.py)) stayed constant. But if I just let things run, memory usage grew and grew.

I believe that the problem centered on the weak ports table. Guile maintains a global hash table of all open ports, so that when the process exits, all of the ports can be flushed. Of course we don't want to allow that table to prevent ports from being garbage collected, so it is a "weak" table.

In Guile, hash tables are implemented as small objects that point to a vector of lists of pairs (a vector of alists). For weak hash tables, the pairs have *disappearing links*: their car or cdr, depending on the kind of table, will be cleared by the garbage collector when the pointed-to object becomes unreachable.

And by "cleared", I mean set to NULL. But the rib stays around, and the vertebra stays in the bucket alist. So later when you traverse a weak hash table, looking for a value, you lazily vacuum out the dead entries.

Dead entries also get vacuumed out when the hash table grows or shrinks such that it should be resized. Guile has an internal table of roughly doubling prime sizes, and when hash table occupancy rises to 90% of the size, or falls below 25%, the hash table is resized up or down. So ideally there are not many collisions.

So, with all of that background information, can you spot the bug?

Well. I certainly couldn't. But here's what I think it was. When you allocate a port, it adds an entry to this weak hash table, allocating at least 4 words and probably more, when you amortize over rehashings. When GC runs and the port is no longer reachable, the port gets collected, and the weak entry nulled out---but the weak entry is still there. Allocation proceeds and your hash table gains in occupancy, vacuuming some slots but, over time, increasing occupancy. Some entries in the hash table might actually be to unreachable ports that haven't been collected yet, for whatever reason.

At some point occupancy increases to the rehashing level. A larger vector is allocated, repopulated with the existing values, and in the process vacuuming ribs and vertebrae for dead references. Overall occupancy is lower but not so much lower as to trigger a rehashing on the low-water-mark side. The process repeats, leading to overall larger memory usge.

❧

You wouldn't think it would be that bad though, would you? Just 32 bytes per port? Well. There are a couple of other things I didn't tell you.

Firstly, port objects are more than meets the eye: the eye of the garbage collector, that is. Besides the memory that they have that GC knows about, they also in general have `iconv_t` pointers for doing encoding conversions. Those `iconv_t` values hide kilobytes of allocation, on GNU anyway. This allocation is indeed properly reclaimed on GC---recall that the web server was not leaking when `(gc)` was run after every request---but it puts pressure on the heap without the GC knowing about it.

See, the GC only runs when it thinks it's necessary. Its idea of "necessary" depends, basically, on how much has been allocated since it last ran. The `iconv_t` doesn't inform this decision, though; to the GC, it is dark matter. So it is possible for the program to outrun the GC for a while, increasing RSS in the part of the heap the GC doesn't scan or account for. And when it is later freed, you have no guarantees about the fragmentation of those newly-allocated blocks.

I think it was ultimately this, that the GC wouldn't run for a while, we would reach the rehashing condition before GC realized the ports weren't accessible, and the process repeated.

This problem was exacerbated by what might be a bug in Guile. In Scheme, one doesn't typically open and close ports by hand. You use `call-with-input-file`, or `with-output-to-string`, or other such higher-order procedures. The reason is that you really want to make sure to delimit the lifetime of, for example, open files from the operating system. So these helper procedures open a port, call a function, close the port, then return the output string or the result from the procedure call or whatever. For the laity, it's like Python's `with` statement.

In the past, string ports did not have much state associated with them, so it wasn't necessary to actually close the port when leaving, for example, `with-output-to-string`. But now that Guile does unicode appropriately, all ports might have `iconv_t` descriptors, so closing the ports is a good idea. Unfortunately it's not a change we can make in the 2.0 series, but it will probably land in 2.2.

❧

Well, what to do? As you can tell by the length of this entry, this problem bothered me for some time. In the end, I do think that open ports are still a problem, in that they can lead to an inappropriately low rate of GC. But it's the interaction with the weaks table--- [remember Alice?](http://www.arlo.net/resources/lyrics/alices.shtml)---that's the killer. GC runs, you collect the ports, but memory that was allocated when the ports were allocated (the rib and vertebra) stays around.

The solution there is to fix up the weak hash tables directly, when the GC runs, instead of waiting for lazy fixup that might never come until a rehash. But the [Boehm-Demers-Weiser collector](http://www.hpl.hp.com/personal/Hans_Boehm/gc/) that we switched to doesn't give you hooks that are run after GC. So, game over, right?

Heh heh. Hie thee hither, hackety hack; sing through me, muse in a duct-tape dress. What we do is to allocate a word of memory, attach a finalizer, and then revive the object in its finalizer. In that way every time the object is collected, we get a callback. This code is so evil I'm going to paste it here:

```
static void
weak_gc_callback (void *ptr, void *data)
{
  void **weak = ptr;
  void *val = *weak;

  if (val)
    {
      void (*callback) (SCM) = data;

      GC_REGISTER_FINALIZER_NO_ORDER
         (ptr, weak_gc_callback, data, NULL, NULL);

      callback (PTR2SCM (val));
    }
}

static void
scm_c_register_weak_gc_callback (SCM obj, void (*callback) (SCM))
{
  void **weak = GC_MALLOC_ATOMIC (sizeof (void**));

  *weak = SCM2PTR (obj);
  GC_GENERAL_REGISTER_DISAPPEARING_LINK (weak, SCM2PTR (obj));

  GC_REGISTER_FINALIZER_NO_ORDER
     (weak, weak_gc_callback, (void*)callback, NULL, NULL);
}

```

And there you have it. I just do a `scm_c_register_weak_gc_callback (table, vacuum_weak_hash_table)`, and that's that.

❧

This discussion does have practical import for readers of this weblog, in that now it shouldn't die every couple days. It used to be that it would leak and leak, and then stop being able to fork out to git to get the data, leading to a number of interesting error cases, but unfortunately none that actually killed the server. It would accept the connection, go and try to serve it, fail, and loop back. It's the worst of all possible error states, if you were actually trying to read this thing; doubtless some planet readers were relieved, though :)

Now with this hackery, I think things are peachy, but time will tell. [Tekuti](//wingolog.org/software/) is still inordinately slow in the non-cached case, so there's some ways to go yet.

❧

Hey I'm talking about garbage collectors, yo! Did I mention the most awesome talk I saw at [FOSDEM](http://fosdem.org/)? No I didn't, did I. Well actually it was Eben Moglen's talk, [well-covered by LWN](http://lwn.net/Articles/426763/). No slides, just the power of thought and voice. I think if he weren't a lawyer he'd make a great preacher. He speaks [with the prophetic voice](//wingolog.org/archives/2010/03/12/with-the-prophetic-voice).

But hey, garbage collectors! The most awesome *technical* talk I saw at FOSDEM was Cliff Click's. It was supposedly about ["Azul's foray into Open Source"](http://www.fosdem.org/2011/schedule/event/azulfoss) \[sic\], but in reality he gave a brief overview of Azul's custom hardware -- their so-called "Vega" systems -- and then about how they've been working on moving downmarket to X86 machines running the Linux kernel.

Let me back up a bit and re-preach what is so fascinating about their work. Click was one of the authors of Java's HotSpot garbage collector. (I don't know how much of this work was all his, but I'm going to personify for a bit.) He decided that for really big heaps--- hundreds of gigabytes---that what was needed was constant, low-latency, multithreaded GC. Instead of avoiding stop-the-world GC for as long as possible, the algorithm should simply avoid it entirely.

What they have is a constantly-running GC thread that goes over all the memory, page by page, collecting everything all the time. My recollection is a little fuzzy right now---I'm writing in a café without internet---but it goes a little like this. The GC reaches a page (bigpages of course). Let's assume that it knows all active objects, somehow. It maps a new page, copies the live objects there, and *unmaps the old page*. (The OS is then free to re-use that page.) It continues doing so for all pages in the system, rewriting pointers in the copied objects to point to the new locations.

But the program is still running! What happens if the program accesses an object from one of the later pages that points to an object from an earlier one that was already moved? Here's where things get awesome: the page fault accessing the old page causes the GC to fix up the pointer, right there and then. La-la-la-la-life goes on. Now that I'm back on the tubes, [here's a more proper link](http://www.infoq.com/articles/azul_gc_in_detail).

OK. So Azul does this with their custom hardware and custom OS, great. But they want to do this on Linux with standard x86 hardware. What they need from the OS is the ability to quickly remap pages, and to have lower latency from the scheduler. Also there are some points about avoiding TLB cache flushes, and lowering TLB pressure due to tagged pointers. (Did you know that the TLB is keyed on byte locations and not word locations? I did not. They have some kernel code to mask off the bottom few bits.)

They want to get these hooks into the standard kernel so that customers running RHEL can just download their JVM and have it working, and to that end have started what they call the ["Managed Runtime Initiative"](http://www.managedruntime.org/).

Basically what this initiative is is a code dump. The initial patches were [totally panned](http://lwn.net/Articles/392307/), and when I mentioned this to Click, he said that maybe they should wait for the next "code dump". He actually used that term!

It's really a shame that they are not clueful about interacting with Free Software folk, because their code could help all language runtimes. In particular I wonder: the Boehm collector is a miracle. It's a miracle that it works at all, and furthermore that it works as well as it does. But it can't work optimally because it can't relocate its objects, so it can lead to heap fragmentation. But it seem that with these page-mapping tricks, the Boehm GC *could* relocate objects. It already knows about all pointers on the system. With some kernel support it could collect in parallel, remapping and compacting the heap as it goes. It's a daunting task, but it sounds possible.

Well. Enough words for one morning. Back to the hack!

## related articles

- [on hoot, on boot](/archives/2024/05/16/on-hoot-on-boot)
- [tree-shaking, the horticulturally misguided algorithm](/archives/2023/11/24/tree-shaking-the-horticulturally-misguided-algorithm)
- [a whiff of whiffle](/archives/2023/11/16/a-whiff-of-whiffle)
- [whiffle, a purpose-built scheme](/archives/2023/11/14/whiffle-a-purpose-built-scheme)
- [i accidentally a scheme](/archives/2023/11/13/i-accidentally-a-scheme)
- [requiem for a stringref](/archives/2023/10/19/requiem-for-a-stringref)

### 6 responses

1. Federico Mena Quintero says:[25 February 2011 9:12 PM](#a932d1355629eed8fc0e142a6284ea15229b9c00)

   Heh. I remember a bug I filed in Sawfish; it used to keep an alist of window properties for each window. When a property was deleted, it would just put a nil in the cadr, but it would never free the cell.

   Seems like this is a common anti-pattern.

2. William ML Leslie says:[27 February 2011 1:21 AM](#ab824057ef7aa04362b90a1e74ca106d03ac5a97)

   Wow, that "managed runtime initiative" looks interesting, just the sort of thing I've been wanting to hack on. They could try to reintegrate the work on the CRAMM VM into the kernel module, too.

   Regarding BDW and copying, I wonder how you stop it from changing integers that happen to look like pointers into space that has been moved. I don't mean in the case of a page miss (because if you are dereferencing it, it is obviously a pointer), but when you copy the page containing the integer, you are supposed to write the new value.

3. [Jeff Walden](http://whereswalden.com/) says:[27 February 2011 6:26 AM](#71a265f3b6fa84a806d51cb844f3bcc2a4185f0a)

   The memory-outside-the-garbage-collector's-purview problem you mention is a fairly common one for web browsers as well, or at least for Mozilla, due to things like images and canvases that may take up a lot of memory but won't make GC happen more often (assuming that's what "should" happen). GC heuristics, therefore, sometimes involve asking the OS for overall process memory usage and possibly GCing more eagerly if the process looks starved for memory even if GC isn't. I know Mozilla considered doing this at one point; I'm not sure if we actually did. Still, it's an interesting problem you get with multiple memory allocators and a GC that doesn't manage all of them.

4. mvo says:[28 February 2011 8:28 AM](#ef0ab2b27625f52de7bf5a830ea7155aaa2240b2)

   There used to be scm\_gc\_register\_collectable\_memory, which could be used to tell the GC about the iconv\_t hanging off of ports, but it has been made a no-op in 2.0. Maybe you can bring it back? But I guess you don't know how big a iconv\_t actually is...

5. NorthernL00n says:[6 April 2011 9:19 PM](#f01f4f080ed103203d0d791f0ede32ca844b9324)

   Just a small NB, Cliff Clicks speciality is/was compilers ; he designed most of the original Hotspot Compiler2, GC in HotSpot came from other folks. (I think Cliff was a PHD student of Michael Wolfe's).

6. John Cowan says:[7 July 2012 4:13 PM](#69176fd0825efaf2ddaef7a931edc12f20cbf6d8)

   [There's more than one way to hack an Alice.](http://www.arlo.net/resources/lyrics/alice_flame.shtml)

Comments are closed.
