---
title: unexpected concurrency — wingolog
url: https://wingolog.org/archives/2012/02/16/unexpected-concurrency
published: "2012-02-16T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2012/02/16/unexpected-concurrency
---

# unexpected concurrency — wingolog

## [unexpected concurrency](/archives/2012/02/16/unexpected-concurrency)

16 February 2012 10:12 PM

- [java](/tags/java)
- [jvm](/tags/jvm)
- [boehm](/tags/boehm)
- [concurrency](/tags/concurrency)
- [finalizers](/tags/finalizers)
- [gc](/tags/gc)
- [python](/tags/python)
- [guile](/tags/guile)
- [threads](/tags/threads)

OK kids, quiz time. Spot the bugs in this Python class:

```
import os

class FD:
    _all_fds = set()

    def __init__(self, fd):
        self.fd = fd
        self._all_fds.add(fd)

    def close(self):
        if (self.fd):
            os.close(self.fd)
            self._all_fds.remove(self.fd)
            self.fd = None

    @classmethod
    def for_each_fd(self, proc):
        for fd in self._all_fds:
            proc(fd)

    def __del__(self):
        self.close()

```

The intention is pretty clear: you have a limited resource (file descriptors, in this case). You would like to make sure they get closed, no matter what happens in your program, so you wrap them in objects known to the garbage collector, and attach finalizers that close the descriptors. You have a for\_each\_fd procedure that should at least allow you to close all file descriptors, for example when your program is about to `exec` another program.

So, bugs?

\\* \\* \\*

Let's start with one: `FD._all_fds` can't sensibly be accessed from multiple threads at the same time. The file descriptors in the set are logically owned by particular pieces of code, and those pieces of code could be closing them while you're trying to `for_each_fd` on them.

Well, OK. Let's restrict the problem, then. Let's say there's only one thread. Next bug?

\\* \\* \\*

Another bug is that signals cause arbitrary code to run, at arbitrary points in your program. For example, if in the `close` method, you get a SIGINT after the `os.close` but before removing the file descriptor from the set, causing an exception to be thrown, you will be left with a closed descriptor in the set. If you swap the operations, you leak an fd. Neither situation is good.

The root cause of the problem here is that *asynchronous signals introduce concurrency*. Signal handlers are run in another logical thread of execution in your program -- even if they happen to share the same stack (as they do in CPython).

OK, let's mask out signals then. (This is starting to get ugly). What next?

\\* \\* \\*

What happens if, during `for_each_fd`, one of the `FD` objects becomes unreachable?

The Python language does not guarantee anything about when finalizers ( `__del__` methods) get called. (Indeed, it doesn't guarantee that they get called at all.) The CPython implementation will immediately finalize objects whose refcount equals zero. Running a finalizer on the method will mutate `FD._all_fds`, while it is being traversed, in this case.

The implications of this particular bug are either that CPython will throw an exception when it sees that the set was modified while iterating over it, or that the finalizer happens to close the fd being processed. Neither one of these cases are very nice, either.

This is the bug I wanted to get to with this article. Like asynchronous signals, *finalizers introduce concurrency*: even in languages with primitive threading models like Python.

Incidentally, this behavior of running finalizers from the main thread was an early [bug in common Java implementations](http://www.cs.arizona.edu/projects/sumatra/hallofshame/monitor-finalizer.html), 15 years ago. All JVM implementors have since fixed this, in the same way: running finalizers within a dedicated thread. This avoids the opportunity for deadlock, or for seeing inconsistent state. Guile will probably do this in 2.2.

For a more thorough discussion of this problem, Hans Boehm has published a number of papers on this topic. The 2002 [Destructors, Finalizers, and Synchronization](http://www.hpl.hp.com/techreports/2002/HPL-2002-335.html) paper is a good one.

## related articles

- [on the impossibility of composing finalizers and ffi](/archives/2024/02/26/on-the-impossibility-of-composing-finalizers-and-ffi)
- [requiem for a stringref](/archives/2023/10/19/requiem-for-a-stringref)
- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [ephemerons vs generations in whippet](/archives/2025/01/09/ephemerons-vs-generations-in-whippet)
- [on taking advantage of ragged stops](/archives/2024/09/06/on-taking-advantage-of-ragged-stops)
- [whippet progress update: funding, features, future](/archives/2024/07/24/whippet-progress-update-funding-features-future)

### 12 responses

1. Johan Ouwerkerk says:[17 February 2012 0:54 AM](#74444ba519ecfc931ecfbe3f992ad59f7376a87d)

   The real bug is of course, that since you don't know \_when\_ the finalizer runs, any long lived app effectively does “leak”. If an fd is occupied for months because you rely on a finalizer to be run, in, say, a server app you've as good as leaked the fd anyway.

   Still, with Python 3 you are supposed to use an entirely different approach: context managers. These do have some VM level guarantees attached to them, at least from what I could make out of the PEP/docs. Available since Python 2.7, by far superior than this manual resource management lark (though judicious use of finally works, it gets tedious).

2. verte (wleslie) says:[17 February 2012 6:17 AM](#503af08b3f6208ebd9ea98430350530082902260)

   That's an excellent summary of the main issue in Boehm's paper. Until you bought it up on the list, I hadn't considered this before; I do remember someone once saying that no single python feature had burned as many brain cells as \_\_del\_\_, and the fairly recent discussion on cap-talk made it even more obvious that people really don't grok all the nuances of finalisers. I just hadn't considered that I was one of those people until I'd read the article. Very interesting stuff.

3. Rodrigo says:[18 February 2012 6:11 PM](#9aa076427e9bb665304f64266504a45621cf9939)

   I think you can bypass the issue by using weak references to trigger "finalizers". If you store a weakref, you can attach a callback that will be executed right before the referenced object is collected. That will give you the chance to cleanup. Do you think this is a good idea?

4. [Thomas Vander Stichele](http://thomas.apestaart.org/) says:[20 February 2012 8:43 AM](#005c2cb884b42e14ec14c411ec49dc9c5acd11f0)

   You hacking on Flumotion again without telling us Andy?

5. ben says:[20 February 2012 9:59 AM](#9ce625358e6df1e3f20d698015a25f15bfedc35b)

   there is also a problem if in the proc you pass to the foreach method creates or closes a FD. you will get: RuntimeError: Set changed size during iteration

6. [wingo](http://wingolog.org/) says:[20 February 2012 10:22 AM](#2885f37ca52b383a74d09a49206c2ec783cda152)

   AFAIU, context managers are only useful within the dynamic extent of a particular block. If you can manage your resources with that pattern, then great. But if you have a global resource whose lifetime is not manageable within a dynamic extent, context managers don't help.

   Anyway, this article is not really about Python at all. I ran into it in the context of a Scheme implementation. Boehm writes about it in the context of C++, Java, and C#. The real message here is that *finalizers introduce concurrency*. If you haven't read the Boehm paper I linked to at the end, I really recommend it :)

7. [Michael Haggerty](http://softwareswirl.blogspot.com/) says:[20 February 2012 3:41 PM](#3263b12c80a5d8793e28dba870cfee0b3faee7a0)

   There is another problem that bit me once. The order of cleanup of global identifiers is not defined. So if an FD instance is created at global level and not deleted before program exit, then it could be that the os module is removed from the global namespace before the FD.\_\_del\_\_() method is called. In this case os.close(self.fd) will fail with a NameError.

   One solution to this particular problem is to store a reference to os within the FD class; \*that\* reference will not be cleaned up until the last FD object has been destroyed.

8. cedric says:[21 February 2012 0:17 AM](#e31d73b456961934e361130920c2fb901ceb30ce)

   Maybe one other bug : I'm not used to python but I believe os.close will report any error on the close by throwing an exception, thus leaving a \_closed\_ FD into the list.

9. oliver says:[29 February 2012 4:06 PM](#5bd7841ce1f0a7814d5fa92611016dc2c0b1af33)

   So if the finalizer is run from a separate dedicated thread, isn't that just exchanging one problem for another? Because now you \_have\_ to make your class thread-safe, else \_\_del\_\_ will be called from another thread and might modify \_all\_fds while the main thread uses it.

10. [wingo](http://wingolog.org/) says:[5 March 2012 10:07 AM](#6b70a0286d2be3108d57deae7e6a2ba6b049e30c)

    Oliver, that's correct, as far as it goes. The source of the concurrency is the finalizer, not whether it runs in a separate thread or not.

    To deal with the concurrency, you have to synchronize, which in practice means making finalizable objects threadsafe. Boehm's [presentation](http://www.hpl.hp.com/personal/Hans_Boehm/misc_slides/java_finalizers.pdf) mentions synchronization a number of times, with exclamation marks :-)

11. [cwillu](http://cwillu.com) says:[23 May 2012 3:13 AM](#be4e60fc55c46fc24737313e2f6909ca0e656619)

    How exactly can proc reduce the refcount of the FD instance to zero while there's still a "self" reference hanging around in the stack?

12. [cwillu](http://cwillu.com) says:[23 May 2012 3:16 AM](#ac1945946f48a260dd0003133befe80e6965db12)

    Er, disregard that (but please fix the naming convention to conform to pep8: "Always use cls for the first argument to class methods")

Comments are closed.
